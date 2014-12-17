+++
title = "Overview of Taxonomies"
Categories = ["taxonomy"]
Description = ""
Tags = ["categories", "lists", "tags"]
date = "2014-12-16T13:57:47-06:00"
menu = "group"
type = "post"
weight = 5
+++

[Hugo](http://gohugo.io/) has an exceptionally flexible system for managing with taxonomies.
That's just a fancy way to say that Hugo helps you to classify and organize your site's content.

<!--more-->
I visualize taxonomies as a sort of encyclopedia. It's a really odd encyclopedia because every volume of
my metaphorical set is about a different topic. There's a volume on chimpanzees and a volume on baseball
players and one on Shakespeare.

Another odd thing about this set is that none of the volumes contain any articles. Open one to any
page and all you'll find is a link to the article. The articles are stored in a separate file cabinet.
That allows articles to be referenced in multiple volumes. For example, my post on a tribe of baseball
playing chimpanzees named The Shakespeares could be included in three separate volumes.

If you can accept the metaphor, then it's not much of a leap to say:

* Taxonomy - a volume of the encyclopedia. It can contain any number of articles.
* Term - the name of an article. It's the key used to find the actual article. Terms are unique within a taxonomy. The same term can appear in multiple taxonomies.
* Content - the article. The content of the article can be referenced from any number of taxonomies but can only be referenced by one term.
