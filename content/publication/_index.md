---
title: "Research"
type: landing
cms_exclude: true

sections:
  - block: collection
    id: working-papers
    content:
      title: "Working Papers"
      #subtitle: "Recent working paper drafts and ongoing studies under peer review"
      filters:
        folders: ["publication"]
        tags: ["working-paper"]
    design:
      view: citation
      columns: '1'

  - block: collection
    id: ongoing-projects
    content:
      title: "Ongoing Projects"
      #subtitle: "Research in progress or at the idea stage"
      filters:
        folders: ["publication"]
        tags: ["ongoing-project"]
    design:
      view: citation
      columns: '1'
---
