# TKey Developer Handbook

Source of the [Tillitis TKey Developer Handbook](https://dev.tillitis.se/).

We use [Hugo](https://gohugo.io/) to generate static web pages from
source written in Markdown files compatible with
[CommonMark](https://spec.commonmark.org/current/). [Mermaid
diagrams](https://mermaid.js.org/intro/) are supported, too.

You need to install Hugo before using it to generate a Hugo website.

Read up on how [Hugo's front
matter](https://gohugo.io/content-management/front-matter/) works to
add titles to your files.

To look at the result:

```
% cd hugo
% hugo serve
...
Web Server is available at http://localhost:1313/ 
...
```

Point your browser at http://localhost:1313/ to see our documentation.

## Editing Hugo pages

All real content is under `hugo/content` and 

## To create a new chapter

Create a directory, for instance `building`.

Create an `_index.md` containing something a like:

```
---
title: Building
weight: 1
---

# Building TKey Programs
  
```

## Write content in a chapter

If you're not going to edit an existing file, create a new one with
your favourite editor.

Please keep lines wrapped at 70 characters.

In the front matter (the stuff inside `---` in the beginning of the
file) you can specify the internal order with `weight: 1`.

Please give at least a title:

```
---
title: Building
weight: 1
---

# Building TKey Programs
  
```
