# docs-experiment

A test repo for documentation technologies and processes.

## Hugo

See under `hugo` for an example using [Hugo](https://gohugo.io/). 

You need to install Hugo before using it to generate a Hugo website.

Read up on how [Hugo's front
matter](https://gohugo.io/content-management/front-matter/) works to
add titles and dates to your files.

```
% cd hugo
% hugo serve
...
Web Server is available at http://localhost:1313/ 
...
```

Point your browser at http://localhost:1313/ to see our documentation.

Please observe that this is just an example with cut and pasted
documentation and a Hugo theme chosen by random.

### Editing Hugo pages

All real content is under `hugo/content` and are Markdown files
compatible with [CommonMark](https://spec.commonmark.org/current/).
[Mermaid diagrams](https://mermaid.js.org/intro/) are supported, too.

#### To create a new chapter

Create a directory, for instance `building`.

Create an `_index.md` containing something like:

```
+++
title = "Building"
date = 2023-01-25T16:19:52+01:00
weight = 1
chapter = true
pre = "<b>1. </b>"
+++

### Chapter 1

# Building apps
  
```

#### Write content in a chapter

If you're not going to edit an existing file, create a new one with
your favourite editor.

In the front matter (the stuff inside `+++` in the beginning of the
file) you can specify the internal order with `weight = 1`.
