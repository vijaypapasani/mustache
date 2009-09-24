rtemplate
=========

Inspired by [ctemplate](http://code.google.com/p/google-ctemplate/)
and
[et](http://www.ivan.fomichev.name/2008/05/erlang-template-engine-prototype.html),
rtemplate is a framework agnostic way to render your views.

It's not a markup language because there is no language. There is no
logic.

Overview
--------

Think of rtemplate as a replacement for your views. Instead of views
consisting of ERB or HAML with random helpers and arbitrary logic,
your views are broken into two parts: a Ruby class and an HTML
template.

We call the Ruby class the "view" and the HTML template the
"template."

All your logic, decisions, and code is contained in your view. All
your markup is contained in your template. The template does nothing
but reference methods in your view. 

This strict separation makes it easier to write clean templates,
easier to test your views, and more fun to work on your app's front end.

Usage
-----

We've got an `examples` folder but here's the canonical one:

    class Simple < RTemplate
      def name
        "Chris"
      end

      def value
        10_000
      end

      def taxed_value
        value - (value * 0.4)
      end

      def in_ca
        true
      end
    end

We simply create a normal Ruby class and define methods. Some methods
reference others, some return values, some return only booleans.

Now let's write the template:

    Hello {{name}}
    You have just won ${{value}}!
    {{#in_ca}}
    Well, ${{taxed_value}}, after taxes.
    {{/in_ca}}

This template references our view methods. To bring it all together,
here's the code to render actual HTML;

    Simple.new.to_html

Which returns the following:

    Hello Chris
    You have just won $10000!
    Well, $6000.0, after taxes.

Simple.

Tag Types
---------

Tags are indicated by the double mustaches. `{{name}}` is a tag. Let's
talk about the different types of tags.

### Variables

The most basic tag is the variable. A `{{name}}` tag in a basic
template will try to call the `name` method on your view. If there is
no `name` method, an exception will be raised.

All variables are HTML escaped by default. If you want, for some
reason, to return unescaped HTML you can use the triple mustache:
`{{{name}}}`. 

### Boolean Sections

A section begins with a pound and ends with a slash. That is,
`{{#person}}` begins a "person" section while `{{/person}}` ends it.

If the `person` method exists and calling it returns false, the HTML
between the pound and slash will not be displayed.

If the `person` method exists and calling it returns true, the HTML
between the pound and slash will be rendered and displayed.

### Enumerable Sections

Enumerable sections are syntactically identical to boolean sections in
that they begin with a pound and end with a slash. The difference,
however, is in the view: if the method called returns an enumerable,
the section is repeated as the enumerable is iterated over. 

Each item in the enumerable is expected to be a hash which will then
become the context of the corresponding iteration. In this way we can
construct loops.

For example, imagine this template:

    {{#repo}}
      <b>{{name}}</b>
    {{/repo}}

And this view code:

    def repo
      Repository.all.map { |r| { :name => r.to_s } }
    end

When rendered, our view will contain a list of all repository names in
the database.

### Comments

Comments begin with a bang are are ignored. The following template:

    <h1>Today.</h1>{{! ignore me }}
    
Will render as follows:

    <h1>Today.</h1>

TODO
----

    [ ] Partials: {{<partial}}
