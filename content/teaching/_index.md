---
title: Teaching
summary: Teaching material and code
type: landing

cascade:
  - _target:
      kind: page
    params:
      show_breadcrumb: true

sections:
  - block: collection
    id: teaching-material
    content:
      title: Teaching Material
      filters:
        folders:
          - teaching
    design:
      view: article-grid
      columns: 1
  - block: collection
    id: code
    content:
      title: Code
      filters:
        folders:
          - project
    design:
      view: article-grid
      fill_image: false
      columns: 1
---
