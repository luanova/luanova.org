+++
title = "An introduction to Orbit"
date = "2009-12-22"
author = "stevejdonovan"
+++

## Installing Orbit

[Orbit](http://orbit.luaforge.net/) is a lightweight framework for Web applications written in Lua. Unlike [Sputnik](http://luanova.org/sputnik/), Orbit gives you the basic tools for constructing applications built on [WSAPI](http://wsapi.luaforge.net/).  If you want authentication, standard look-and-feel, wiki-like functionality, you are better off starting with Sputnik; but Orbit is a satisfying way to build web applications from scratch.

The easiest way to install Orbit on a Unix machine is using [LuaRocks](http://luarocks.org/), but here for a change I will assume that you already have [Lua for Windows](http://luaforwindows.luaforge.net/) which already has most of the dependencies for Orbit.  Then download [ orbit-2.0.2-all.zip](http://mysite.mweb.co.za/residents/sdonovan/lua/orbit-2.0.2-all.zip), unzip it somewhere preserving folder names, and run  **lua install-orbit.lua** from this directory in a command prompt.

The examples in this tutorial are then in the **orbit-2.0.2\samples\basic** folder.

## The Basics

The simplest Orbit application is one that serves up a single HTML page:

{{< highlight lua >}}
-- basic1.lua
require"orbit"

-- declaration
module("basic1", package.seeall, orbit.new)

-- handler
function index(web)
  return render_index()
end

-- dispatch
basic1:dispatch_get(index, "/", "/index")

-- render
function render_index()
   return [[
    <head></head>
    <html>
    <h2>Pretty Easy!</h2>
    </html>
    ]]
end
{{< /highlight >}}

An Orbit application is a Lua module, declared in the usual way except for the extra 'orbit.new'. Lua's **module** function can take a number of extra arguments, which are all functions which modify the module table in some way: **package.seeall** makes the global environment accessible to the module, and **orbit.new** adds some extra methods to the new module (you may think of this as 'inheritance'.)

Any HTML request is mapped onto handler functions - in this case, the result of doing a GET request on '/' or '/index'. **index** just asks **render_index** to actually generate the HTML for this particular page. You can then run this application using the default Xavante server:

    c:\basic> orbit basic1.lua
    Starting Orbit server at port 8080

And you can view the result by going to **http://localhost:8080** in your favourite browser.

Not perhaps very impressive, seeing as the result could be achieved by a simple static page, but the render function can return _any_ dynamically created string. And this example will work with Xavante, FastCGI or CGI, because Orbit is a WSAPI application.

All the usual web request variables are parsed and available to you. The **web** variable returned from the handler contains a table **GET** which has all the parameters passed to the server from a **GET** request. Call this script with **http://localhost:8080/?cat=felix&dog=fido** and you will see that **cat** and **dog** are fields of **web.GET**, appropriately translated from URL encoding.

{{< highlight lua >}}
-- basic3.lua
require"orbit"

module("basic3", package.seeall, orbit.new)

basic3:dispatch_get(function(web)
    local ls = {}
    for k,v in pairs(web.GET) do
        table.insert(ls,('<li>%s = %s</li>'):format(k,v))
    end
    ls = '<ul>'..table.concat(ls,'\n')..'</ul>'
    return ([[
    <html><head></head><body>
    Web Variables <br/>
    %s
    </body></html>
    ]]):format(ls)
end, "/", "/index")
{{< /highlight >}}

This is not a pretty script; it's only virtue is that it is short. It is ugly for two basic reasons; first, generating HTML with regular string operations is awkward, and second, it does not separate the logical parts out neatly. The HTML problem will be discussed in the next section, but the next example separates the various operations into their own sections.

This also shows how Orbit can serve up static content as well, in this case anything inside the **/images** directory:

{{< highlight lua >}}
-- basic2.lua
require"orbit"

module("basic2", package.seeall, orbit.new)

function index(web)
  return render_index(collectgarbage("count"))
end

basic2:dispatch_get(index, "/", "/index")
-- any file in this directory will be served statically
basic2:dispatch_static ("/images/.+")

local template = [[
<head></head>
<html>
%s
</html>
]]

function render_page(contents)
    return template:format(contents)
end

function render_index(mem)
   return ([[
    <img src="/images/lua.png"/>
    <h2>Memory used by Lua is %6.0f kB<h2>
   ]]):format(mem)
end
{{< /highlight >}}

Note that **dispatch_static** is passed a Lua _string pattern_, not a file mask; in this case, anything in the directory.

We are now presenting true dynamic content (the amount of memory managed by Lua) and made a first start to prettifying the result. It is convenient to only generate the 'inner' HTML and use a template to generate the full page, so **render_page** has been factored out.

Orbit follows the Model-View-Controller (MVC) pattern that separates an application into three distinct parts. The Controller acts as the executive, the Model manages the data, and the View presents the data to the user; here the dispatch rules calls **index** which uses the renderer **render_index** to present the HTML form of the data. This pattern comes from a basic principle, [Separation of Concerns](http://en.wikipedia.org/wiki/Separation_of_concerns). You do not want to mix database access, application logic and visual presentation together because the result is not clear and may become unmaintainable quickly.

## HTML the Orbit Way

The HTML problem is usually solved using templates, such as [Cosmo](http://cosmo.luaforge.net/) as discussed in the last [article](http://luanova.org/sputnik) on Sputnik. Orbit provides another option, which is that Lua code can generate HTML:


{{< highlight lua >}}
-- html1.lua
require"orbit"

function generate()
    return html {
        head{title "HTML Example"},
        body{
            h2{"Here we go again!"}
        }
    }
end

orbit.htmlify(generate)

print(generate())
{{< /highlight >}}

    ==>
    C:\basic>lua html1.lua
    <html ><head ><title >HTML Example</title>></head><body ><h2 >
    Here we go again!</h2></body></html>
    </pre>

This certainly reads better if you are used to Lua, but the magic involved needs some explanation. You may imagine that Orbit has introduced a whole bunch of little functions into the module, but this is not how it is done.  Instead, **orbit.htmlify** modifies the environment of the function **generate** so that any undefined variable is assumed to be a HTML tag and becomes a suitable function. So you can generate anything, even with unknown tags. So if **generate** was:

{{< highlight lua >}}
function generate()
    return sensors {
        sensor {name="one"},
        sensor {name="two"},
    }
end
{{< /highlight >}}

Then the result would be this syntactically valid XML:

    <sensors ><sensor name="one"></sensor>
    <sensor name="two"></sensor></sensors>

In presenting the resulting markup I've inserted a few line breaks for making reading a bit easier.  Naturally you might like to [beautify](http://n2.nabble.com/Orbit-htmlify-beautification-tc3889818.html#a3889818) the output for debugging purposes; web browsers do not care. [HTML Tidy](http://infohound.net/tidy/) is another option. which will also give you a host of other diagnostic checks.

One problem to watch out for is that a mistyped tag name will give you odd HTML, not a Lua error. And there are some HTML tags which are valid Lua globals, such as **table** - the solution is to say **H('table')**, etc.  But HTML can now be generated without making your eyes bleed, as with the ugly web variables script. This little snippet uses the fact that tag functions take table arguments:

{{< highlight lua >}}
function generate (web)
    local list = {}
    for name,value in pairs(web.GET) do
        table.insert(list,li(name.." = "..value))
    end
    return ul(list)
end
{{< /highlight >}}

Any technique can produce over-elaborate code, as this program which generates an HTML form formatted with a table:

{{< highlight lua >}}
require"orbit"

function wrap (inner)
    return html{ head(), body(inner) }
end

function test ()
    return wrap(form (H'table' {
        tr{td"First name",td( input{type='text', name='first'})},
        tr{td"Second name",td(input{type='text', name='second'})},
        tr{ td(input{type='submit', value='Submit!'}),
            td(input{type='submit',value='Cancel'})
        },
    }))
end


orbit.htmlify(wrap,test)

print(test())
{{< /highlight >}}

Not quite eye-bleeding, but getting there!  The solution is to capture common patterns in a library. The **util** module in the **basic** folder makes for much more readable code:

{{< highlight lua >}}
function show_form (web)
    return wrap(form {
        htable {
            {text("First name",'first')},
            {text("Second name",'second')},
            {submit 'Submit', submit 'Cancel'},
        }
    })
end
{{< /highlight >}}

## Databases the Orbit Way

Orbit can create an Object-Relational Mapping (ORM) between the tables of a relational database and Lua tables. The restriction is that the SQL table is required to have a numeric key called **id**. In the following discussion, we must switch between two very different meanings of the word **table**, so I'll use 'array' to mean 'an array-like Lua table' and 'map' to mean 'a hash-like Lua table'.

Consider the following SQL table definition found in the **blog** example database:

{{< highlight sql >}}
CREATE TABLE blog_post
       ("id" INTEGER PRIMARY KEY NOT NULL,
       "title" VARCHAR(255) DEFAULT NULL,
       "body" TEXT DEFAULT NULL,
       "n_comments" INTEGER DEFAULT NULL,
       "published_at" DATETIME DEFAULT NULL);
{{< /highlight >}}

This code uses LuaSQL's sqlite3 driver to dump out a particular row of this table by defining a mapper:

{{< highlight lua >}}
-- ORM1.lua
require "orbit"
require "luasql.sqlite3"
local env = luasql.sqlite3()

function dump (t)
    if #t > 0 then
        for _,row in ipairs(t) do
            dump(row)
        end
    else
        for field,value in pairs(t) do
            if type(value) == 'string' then
                value = value:sub(1,60)
            end
            print(field,type(value),value)
        end
        print '-----'
    end
end

mapper = orbit.model.new()
mapper.conn = env:connect("../blog/blog.db")
mapper.driver = "sqlite3"
mapper.table_prefix = 'blog_'

posts = mapper:new 'post'  -- maps to blog_post table

-- print out the second post
second = posts:find(2)

dump(second)
{{< /highlight >}}

    ==>
    published_at    number  1096923480
    title   string  The Care And Feeding Of Your Sleeping Bicycle
    id  number  2
    n_comments  number  0
    body    string
    Thunder, thunder, thundercats, Ho! Thundercats are on the m
    -----

**posts:find(2)** gets the database row with **id** equal to 2 and returns a map with corresponding fields and values.

SQL is a more strongly-typed language than Lua, so that both **published_at** and **n_comments** are mapped to the **number** type. However, the mapper does preserve this information in its field **meta**; for instance, **mapper.meta.published_at.type** is 'datetime'.

Orbit's ORM mappers simplify database access by bridging the gap between the underlying database and the language; the most involved code here is just to dump out Lua maps and arrays.

As always, the interactive prompt is the best way to explore the available mapper methods:

    C:\basic>lua -lmapper
    Lua 5.1.4  Copyright (C) 1994-2008 Lua.org, PUC-Rio
    > posts = mapper:new 'post'
    > dump(posts:find_first('n_comments > 0'))
    published_at    number  1097769720
    title   string  'Star Wars' As Written By A Princess
    id      number  4
    n_comments      number  1
    body    string
    Ten years ago a crack commando unit was sent to prison by a
    -----

**find_first** also returns a row, but by an explicit condition; **find_all** is similar, but returns an array of all matching rows. Note that the condition is a SQL expression, not Lua; this is effectively a more friendly way to create an SQL query:

{{< highlight lua >}}
dump(posts:find_all("id = ? or id = ?",
    {2,4,order = "published_at asc"}))
{{< /highlight >}}


ORM mapping works particularly nicely with dynamic languages, but there are some downsides.  consider the case that the table **blog_post** had many thousands of rows; the default mapper will grab all the columns, including the potentially rather large **body**. So opportunities for optimizing queries are lost, if we were only interested in searching the title for a match. It's possible to reorganize things so that **blog_post** only contained 'meta' data about each post (plus perhaps a short summary) and a new table **blog_text** would contain the actual text of each post, indexable using the same id.

## Putting it Together

The next example will bring together these three concerns: request pattern matching (Controller), ORM mapping to a database (Model) and Orbit htmlification (View). It uses the database from the blog example but presents it in summary form.  For each row, it puts out a nicely formatted summary.  The actual rows are selected by a user set of keywords.

All our HTML-generating functions begin with 'render_'. **orbit.htmlify** can be called in another way, where you give the module first, and then specify a string pattern to match the functions in the module that you want to htmlify.

{{< highlight lua >}}
orbit.htmlify(blog1,'render_.+')
{{< /highlight >}}

Initially, the actual model-controller part of the application is simply this:

{{< highlight lua >}}
function index(web)
  return render_index(posts:find_all())
end

blog1:dispatch_get(index, "/", "/index")
{{< /highlight >}}

And the view part:

{{< highlight lua >}}
local function limit (s,maxlen)
    if #s > maxlen then
        return s:sub(1,maxlen)..'...'
    else
        return s
    end
end

function render_page(contents)
    return html {
        head { title 'Blog Example' },
        body (contents)
    }
end

function render_post(post)
    return div {
        h3 (post.id .. ' ' .. post.title),
        p (limit(post.body,128)),
        em(os.date('%a %b %d %H:%M %Y',post.published_at)),
        ' ',post.n_comments,' comments'
    }
end

function render_index(posts)
    local res = {}
    for i,post in ipairs(posts) do
        res[i] = render_post(post)
    end
    return render_page(res)
end
{{< /highlight >}}

The heart of the view is **render_post**, which formats each row from the model. You will only achieve date formatting happiness once you have learned the formating characters for **os.date**; these are the same as those used in the C <a href="http://msdn.microsoft.com/en-us/library/fe06s4ak%28VS.80%29.aspx">strftime</a> function.

All in all, not bad for a 55-line program. The next step is to add simple keyword searching, which takes an extra line:

{{< highlight lua >}}
function index(web)
  local keyword = web.GET.keyword or ''
  if keyword == '' then return {}
  else
    return render_index(posts:find_all('title like ?',{'%'..keyword..'%'}))
  end
end
{{< /highlight >}}

The SQL **like** operator works with patterns, so that finding a title containing the word 'Chicken' would be "title like '%Chicken%'", where '%' means 'any sequence of characters'.  The keyword is passed as a GET request variable: enter the following URL to see the results of the search: **http://localhost:8080/?keyword=Chicken**.

It would be useful to have _some_ user interface, especially when we start passing multiple keywords.  **render_index** becomes:

{{< highlight lua >}}
function render_index(posts)
    local res = {}
    append(res,p (form {
        input {type='text',name='keyword'},
        input {type='submit',value='Submit'},
    }))
    append(res,hr())
    append(res,h2(('Found %d posts'):format(#posts)))
    for _,post in ipairs(posts) do
        append(res,render_post(post))
    end
    return render_page(res)
end
{{< /highlight >}}

By default, HTML forms submit their values to the same URL as the page, so this works fine.

This introduction was intended to help you begin reading the Orbit samples, documentation and tutorials. In the next in this series, we will discuss building more complicated applications, caching generated pages, and Orbit Pages, which is a powerful Lua-powered template engine for generating HTML.

