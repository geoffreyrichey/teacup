![teacup](https://raw.github.com/goodeggs/teacup/master/docs/teacup.jpg)

Teacup is templates in CoffeeScript.

Compose DSL functions to build strings of HTML.  
Package templates and helpers in CommonJS, AMD modules, or vanilla coffeescript.  
Integrate with the tools you love: Express, Backbone, Rails, and more.

[![Build Status](https://travis-ci.org/goodeggs/teacup.png)](https://travis-ci.org/goodeggs/teacup)

The Basics
---------------

Use the `renderable` helper to create a function that returns an HTML string when called.

```coffee
{renderable, ul, li} = require 'teacup'

template = renderable (items)->
  ul ->
    li item for item in items

console.log template(['One', 'Two'])
# Outputs <ul><li>One</li><li>Two</li></ul>
```

Use the `render` helper to render a template to a string immediately.

```coffee
{render, ul, li} = require 'teacup'

output = render ->
  ul ->
    li 'First Item'
    li 'Second Item'

console.log output
# Outputs <ul><li>First Item</li><li>Second Item</li></ul>
```


### Express

To use Teacup as your Express template engine:

Install from npm

    $ npm install teacup

Register Teacup as a view engine.

``` coffee
express = require 'express'
teacup = require 'teacup/lib/express'

app = express()
app.configure ->
  app.engine "coffee", teacup.renderFile
```

Then write your views as regular old coffee files that export a renderable template.

```coffee
# views/example.coffee
{renderable, div, h1} = require 'teacup'

module.exports = renderable ({title}) ->
  div '#example', ->
    h1 "Hello, #{title}"
```

You can use Teacup templates even if your Express app is not using CoffeeScript.

### connect-assets

If you are using [connect-assets](https://github.com/TrevorBurnham/connect-assets) to compile your CoffeeScript in
an asset pipeline, you can use the Teacup middleware which registers connect-assets `js` and `css` helpers with Teacup.

Grab the module to get started

    $ npm install teacup

Then configure the middleware

```coffee
express = require 'express'
connectAssets = require 'teacup/lib/connect-assets'
app = express()
app.configure ->
  app.use connectAssets(src: 'assets', jsDir: 'javascripts', cssDir: 'stylesheets')
```

And in your templates:

```coffee
{renderable, js, css, html, head, body} = require 'teacup'

module.exports = renderable ->
  html ->
    head ->
      js 'app'
      css 'app'
    body ->
      # ...
```

The Teacup middleware passes the provided options to connect-assets and returns an instance of the connect-assets middleware.

### Browser

To use for client-side rendering, all you need is [teacup.js](https://raw.github.com/goodeggs/teacup/master/lib/teacup.js).  You can 
toss it in a script tag, `require()` and browserify it, load it with an AMD loader, send it down an asset pipeline 
like Rails or connect-assets, or use some sweet custom build process.

Teacup claims window.teacup if you arent using AMD or CommonJS.

```coffee
{renderable, ul, li} = teacup

template = renderable (items)->
  ul ->
    li item for item in items

console.log template(['One', 'Two'])
```

### Backbone

Feel free to write your template in the same file as a Backbone View and call it from `view.render()` like so:
```coffee
{renderable, div, h1, ul, li, p, form, input} = teacup

template = renderable (kids) ->
  div ->
    h1 "Welcome to our tea party"
    p "We have a few kids at the table..."
    ul ->
      kids.each (kid) ->
        li kid.get 'name'
    form ->
      input placeholder: 'Add another'

class PartyView extends Backbone.View

  constructor: (kids) ->
    @kids = new Backbone.Collection kids
    super()

  render: ->
    @$el.html template(@kids)
    @$('form input').focus()
    @
    
```
Check out [teacup-backbone-example](https://github.com/goodeggs/teacup-backbone-example) for a complete Backbone + Express app.


### Rails

The [Teacup::Rails](https://github.com/goodeggs/teacup-rails) gem makes Teacup available to the asset pipeline in Rails 3.1+.

Guide
---------

### Escaping

Teacup escapes input by default. To disable escaping, use the `raw` helper.

```coffee
{render, raw, h1, div} = require 'teacup'

inner = render ->
  h1 'Header'

console.log render ->
  div inner
# Outputs <div>&lt;h1&gt;Header&lt;/h1&gt;</div>

console.log render ->
  div ->
    raw inner
# Outputs <div><h1>Header</h1></div>
```

### Text

The text helper inserts a string in the template without wrapping it in a tag.  It creates a [text node](https://developer.mozilla.org/en-US/docs/DOM/Text).

```coffee
{render, text, b, em, p} = require 'teacup'

console.log render ->
  p ->
    text 'Sometimes you just want '
    b 'plain'
    text ' text.'
# Outputs <p>Sometimes you just want <b>plain</b> text.</p>
```

### Helpers

Write view helpers as renderable functions and require them as needed.

Here's a helpers file that defines a set of [microformats](http://microformats.org/wiki/hcalendar).

```coffee
# views/microformats.coffee
{renderable, span, text} = require 'teacup'
moment = require 'moment'

module.exports =
  hcalendar: renderable ({date, location, summary}) ->
    span ".vevent", ->
      span ".summary", summary
      text " on "
      span ".dtstart", moment(date).format("YYYY-MM-DD")
      text " was in "
      span ".location", location
```

And a view that uses one of the helpers.

```coffee
# views/events.coffee
{renderable, ul, li} = require 'teacup'
{hcalendar} = require './microformats'

module.exports = renderable ({events}) ->
  ul ->
    for event in events
      li ->
        hcalendar event
```

### Compiling Templates

Just use the CoffeeScript compiler.  Uglify will make em real small.

```
$ coffee -cl -o build src
```

FAQ
----

**How's this different from CoffeeCup?**

[CoffeeCup](http://github.com/gradus/coffeecup) is the currently maintained fork of
[CoffeeKup](http://github.com/mauricemach/coffeekup) and is what we were using at Good Eggs before switching to Teacup.
The problem with CoffeeCup is that it uses some `eval` magic to put the tag functions in scope. This magic breaks
closure scope so you can't actually write templates using the functional constructs that you'd expect.

Legacy
-------

[Markaby](http://github.com/markaby/markaby) begat [CoffeeKup](http://github.com/mauricemach/coffeekup) begat
[CoffeeCup](http://github.com/gradus/coffeecup) and [DryKup](http://github.com/mark-hahn/drykup) which begat **Teacup**.

Contributing
-------------

```
$ git clone https://github.com/goodeggs/teacup && cd teacup
$ npm install
$ npm test
```
