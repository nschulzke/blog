+++
title = "The Tooling"
date = 2022-09-01T20:46:25-06:00

[taxonomies]
tags = ["Languages", "Tooling"]
+++

In this post I'm going to get set up with the playground tooling I'll use to showcase the languages I discuss here. The goal is to have something similar to a Jupyter notebook: a series of executable text-editor fields that share an environment.

<!-- more -->

For this example, I'll be using a simple custom lisp. The details of the language aren't important for this post, but [you can find the repo here](https://github.com/nschulzke/lisp).

{{ language_embed(name="lisp", path="tiny-lisp-0.1/lisp.js") }}

Here is a playground:

{% playground(name="lisp") %}
(progn
  (def a 1)
  (+ a 1))
{% end %}

All playgrounds for the same language in the same file share the same environment, so you can use defines from one in the others (after they've been run).

{% playground(name="lisp") %}
(+ a 2)
{% end %}

For normal results, the playground will show the result and give a green check to indicate the listing has been run. If there's an error, the playground will show the error in red and have an X:

{% playground(name="lisp") %}
(+ b 2)
{% end %}

Typing in any field will clear the status. Refreshing the page will reset the environment.
