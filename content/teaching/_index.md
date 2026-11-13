---
title: Teaching
linkTitle: Teaching
summary: Teaching material and code
type: landing

design:
  # Tighten the gap between sections, matching the Research page
  spacing: "2rem"

cascade:
  - _target:
      kind: page
    params:
      show_breadcrumb: true

sections:
  - block: collection
    id: teaching-material
    content:
      title: Teaching
      sort_by: Weight
      order: asc
      filters:
        folders:
          - teaching
    design:
      view: teachcard
      columns: 1
  - block: collection
    id: code
    content:
      title: Extra Material
      sort_by: Weight
      order: asc
      filters:
        folders:
          - project
    design:
      view: teachcard
      columns: 1
---
