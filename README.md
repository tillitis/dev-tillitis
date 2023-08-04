[![deploy hugo site](https://github.com/tillitis/dev-tillitis/actions/workflows/deploy.yaml/badge.svg)](https://github.com/tillitis/dev-tillitis/actions/workflows/deploy.yaml)

This is
[automatically](https://github.com/tillitis/dev-tillitis/actions/workflows/deploy.yaml)
deployed to https://dev.tillitis.se

# Developer Handbook

Source of [Tillitis TKey Developer Handbook](https://dev.tillitis.se/).

## Hugo

See under `hugo` for our static site source. You need to install
[Hugo](https://gohugo.io/) before using it to generate a Hugo website.

To see the Developer Handbook:

```
% cd hugo
% hugo server
...
Web Server is available at http://localhost:1313/ 
...
```

Point your browser at http://localhost:1313/ to see our documentation.

## Editing pages

All real content is under `hugo/content` and are Markdown files
compatible with [CommonMark](https://spec.commonmark.org/current/).
[Mermaid diagrams](https://mermaid.js.org/intro/) are supported, too.

## First page

The first page is in `hugo/content/_index.md`.

## To create a new chapter

Create a directory, for instance `tools`.

Create an `_index.md` containing something like:

```yaml
---
title: Tools & libraries
weight: 2
---

# Tools & libraries

Text about tools and libraries.
```

## Write content in a chapter

If you're not going to edit an existing file, create a new one with
your favourite editor.

In the front matter (the stuff inside `---` in the beginning of the
file) you can specify the internal order with `weight: 2` to get it in
second place in the table of contents.
