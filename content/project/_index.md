---
cascade:
  - _target:
      kind: page
    params:
      show_breadcrumb: true
sections:
  - block: collection
    content:
      filters:
        folders:
          - project
    title: GitHub Repositories
    design:
      columns: 1
      view: article-grid
#  id: teaching
# summary: My courses
title: Github Repositories
linkTitle: Teaching
type: landing
# No "GitHub Repositories" page: the two pages are reached from the Teaching page
_build:
  render: never
---
