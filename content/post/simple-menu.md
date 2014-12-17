+++
title = "A Simple Menu"
Categories = ["menus"]
Description = "simple dynamic menus"
Tags = ["menus"]
date = "2014-12-17T15:39:00-06:00"
menu = "menus"
type = "post"
weight = 5
+++

[Hugo](http://gohugo.io/) comes with the ability to control how [menus](http://gohugo.io/extras/menus/)
get generated. This article will focus on just a few features - the mininum needed to make a set of
static menus look like they are dynamic.

I think that it is easiest to create a static version of what I am after and then
make changes to turn it into a template. That works really well with the [live reload](http://gohugo.io/extras/livereload/)
feature.

As always, you can find all of this at my [repo](https://github.com/quoha/blank).

## A Hard-Coded List

The site's menu is simply going to list most of the categories and provide a link to the list page
for the category. My theme has support for showing the "current" menu item by highlighting it, so
I would like to use that.

Here's what the generated HTML should look like.

```
<nav>
	<ul class="menu">
		<li><a class="current" href="{{ .Site.BaseUrl }}/" title="">Home</a></li>
		<li><a href="{{ .Site.BaseUrl }}/">Templates</a>
			<ul class="subpages">
				<li><a href="{{ .Site.BaseUrl }}/archetype">Archetypes</a></li>
				<li><a href="{{ .Site.BaseUrl }}/group">Groups</a></li>
				<li><a href="{{ .Site.BaseUrl }}/homepage">Home Page</a></li>
				<li><a href="{{ .Site.BaseUrl }}/list">Lists</a></li>
				<li><a href="{{ .Site.BaseUrl }}/menu">Menus</a></li>
				<li><a href="{{ .Site.BaseUrl }}/partial">Partials</a></li>
				<li><a href="{{ .Site.BaseUrl }}/single">Singles</a></li>
				<li><a href="{{ .Site.BaseUrl }}/static">Static</a></li>
				<li><a href="{{ .Site.BaseUrl }}/syntax">Syntax</a></li>
			</ul>
		</li>
		<li><a href="{{ .Site.BaseUrl }}/best">Best Practices</a>
		<li><a href="{{ .Site.BaseUrl }}/misc">Miscellaneous</a>
		<li><a href="{{ .Site.BaseUrl }}/privacy">Privacy</a></li>
		<li><a href="{{ .Site.BaseUrl }}/about">About</a></li>
	</ul>
</nav>
```

Note that I added the "current" class. To give it that dynamic feel, I want the menu item
highlighting to change to reflect the currently displayed content. That is not as hard as
it might seem.

I am using `.Site.BaseUrl` on the off-chance that I move this site to a sub-directory on
my web site. I doubt that will ever happen, but it is a good idea, especially if you are
working on a theme since you can't predict what users will do with it.

## Setting up the Front Matter

Hugo expects to find menu definitions in the front matter, so that is where I will start.

I want my menu to sort of mirror the tag and category structure, so I set my "menu" to the
name of the menu that I want to associate the content with. For example, this article is
about menus, so I add the following to my front matter:

```
menu = "menus"
```

For my "about" page, I set the value to "about." For an article about the home page, I would
set it to "homepage." That aligns very nicely with the paths in my menu structure.

## Create a Partial

I like to create partial templates for most of my work, and this is no different.

```
{{ "<!-- hugo: enter themes/blank/partials/nav.html -->" | safeHtml }}
<nav>
	<ul>
		<li><a class="current" href="{{ .Site.BaseUrl }}/" title="">Home</a></li>
		<li><a href="{{ .Site.BaseUrl }}/">Templates</a>
			<ul class="subpages">
				<li><a href="{{ .Site.BaseUrl }}/archetype">Archetypes</a></li>
				<li><a href="{{ .Site.BaseUrl }}/group">Groups</a></li>
				<li><a href="{{ .Site.BaseUrl }}/homepage">Home Page</a></li>
				<li><a href="{{ .Site.BaseUrl }}/list">Lists</a></li>
				<li><a href="{{ .Site.BaseUrl }}/menu">Menus</a></li>
				<li><a href="{{ .Site.BaseUrl }}/partial">Partials</a></li>
				<li><a href="{{ .Site.BaseUrl }}/single">Singles</a></li>
				<li><a href="{{ .Site.BaseUrl }}/static">Static</a></li>
				<li><a href="{{ .Site.BaseUrl }}/syntax">Syntax</a></li>
			</ul>
		</li>
		<li><a href="{{ .Site.BaseUrl }}/best">Best Practices</a>
		<li><a href="{{ .Site.BaseUrl }}/misc">Miscellaneous</a>
		<li><a href="{{ .Site.BaseUrl }}/privacy">Privacy</a></li>
		<li><a href="{{ .Site.BaseUrl }}/about">About</a></li>
	</ul>
</nav>
{{ "<!-- hugo: exit themes/blank/partials/nav.html -->" | safeHtml }}
```

In my template, I call the partial:

```
{{ partial "nav.html" . }}
```

I keep a browser window open next to my text editor so that I can see the changes every time
I save the file.

## Front Matter Variables

When you define a variable in the front matter, Hugo stores it in the `.Params` map. You just
have to add the name of the variable to reference it:

```
.Params.menu
```

We can use a simple test to determine which menu item to highlight: if the articles "menu" variable is
the same as the menu item's path, then it should be highlighted.

```
{{ if eq "archetypes" .Params.menu }}class="current"{{end}}
```

One of the nice features about the `.Params` map is that it doesn't throw an error if the variable
isn't defined. That means we can use `.Params.menu` even if the user forgets to define it in their
front matter.

(Note - if you are doing something like this, you should provide a default like `menu = "none"` in your archetypes.)

Let's update all the lines with that test:
```
<li><a {{ if eq "archetypes" .Params.menu }}class="current"{{end}} href="{{ .Site.BaseUrl }}/archetype">Archetypes</a></li>

```

That works pretty well. If an article defines a menu, then the corresponding menu item is highlighted.
There are a couple of problems, though. If the article doesn't define a menu, then nothing is highlighted.
Or if it defines one but there's a typo (say "archetype" or "architypes"), then nothing is highlighted.

While that second issue is generally an unsolvable problem, I want the easy out for the first - highlight the Home menu.

## Defaulting a Variable

This should be an easy thing to do: if the menu isn't defined, provide a default value of "home."

My favorite way to do that is with the "with" template command:

```
{{ with (or .Params.menu "home") }}
```

That reads "In the following block, use .Params.menu if it is defined, otherwise use the value 'home'." It's nifty.

```
{{ with (or .Params.menu "home") }}
	<ul class="menu">
		<li><a {{ if eq "home" . }}class="current"{{ end }} href="{{ .Site.BaseUrl }}/" title="">Home</a></li>
```

Saving that, though, causes problems with the generated page because the "with" command resets "." and
that means `.Site` is no longer valid. The fix is to do the variable-local-shuffle:

```
{{ $baseUrl := .Site.BaseUrl }}
{{ with (or .Params.menu "home") }}
	<ul class="menu">
		<li><a {{ if eq "home" . }}class="current"{{ end }} href="{{ $baseUrl }}/" title="">Home</a></li>
```

## The Final Partial

My theme's `layous/partials/nav.html` contains:

```

{{ "<!-- hugo: enter themes/blank/partials/nav.html -->" | safeHtml }}
<nav>
{{ $baseUrl := .Site.BaseUrl }}
{{ with (or .Params.menu "home") }}
	<ul class="menu">
		<li><a {{ if eq "home" . }}class="current"{{ end }} href="{{ $baseUrl }}/" title="">Home</a></li>
		<li><a href="{{ $baseUrl }}/" title="">Templates</a>
			<ul class="subpages">
				<li><a {{ if eq "archetypes" . }}class="current"{{end}} href="{{ $baseUrl }}/archetype">Archetypes</a></li>
				<li><a {{ if eq "groups"     . }}class="current"{{end}} href="{{ $baseUrl }}/group">Groups</a></li>
				<li><a {{ if eq "homepage"   . }}class="current"{{end}} href="{{ $baseUrl }}/homepage">Home Page</a></li>
				<li><a {{ if eq "lists"      . }}class="current"{{end}} href="{{ $baseUrl }}/list">Lists</a></li>
				<li><a {{ if eq "menus"      . }}class="current"{{end}} href="{{ $baseUrl }}/menu">Menus</a></li>
				<li><a {{ if eq "partial"    . }}class="current"{{end}} href="{{ $baseUrl }}/partial">Partials</a></li>
				<li><a {{ if eq "single"     . }}class="current"{{end}} href="{{ $baseUrl }}/single">Singles</a></li>
				<li><a {{ if eq "static"     . }}class="current"{{end}} href="{{ $baseUrl }}/static">Static</a></li>
				<li><a {{ if eq "syntax"     . }}class="current"{{end}} href="{{ $baseUrl }}/syntax">Syntax</a></li>
			</ul>
		</li>
		<li><a {{ if eq "best"    . }}class="current"{{end}} href="{{ $baseUrl }}/best">Best Practices</a>
		<li><a {{ if eq "misc"    . }}class="current"{{end}} href="{{ $baseUrl }}/misc">Miscellaneous</a>
		<li><a {{ if eq "privacy" . }}class="current"{{end}} href="{{ $baseUrl }}/privacy">Privacy</a></li>
		<li><a {{ if eq "about"   . }}class="current"{{end}} href="{{ $baseUrl }}/about">About</a></li>
	</ul>
{{ end }}
</nav>
{{ "<!-- hugo: exit themes/blank/partials/nav.html -->" | safeHtml }}
```
