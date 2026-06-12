---
title: "Research"
type: landing
cms_exclude: true

design:
  # Tighten the vertical gap between sections (default is ~6rem)
  spacing: "2rem"

sections:
  - block: collection
    id: working-papers
    content:
      title: "Working Papers"
      #subtitle: "Recent working paper drafts and ongoing studies under peer review"
      filters:
        folders: ["publication"]
        publication_types: ["working-paper"]
    design:
      view: citation
      columns: '1'

  - block: collection
    id: ongoing-projects
    content:
      title: "Ongoing Projects"
      #subtitle: "Research in progress or at the idea stage"
      filters:
        folders: ["ongoing"]
        publication_types: ["ongoing-project"]
    design:
      view: citation
      columns: '1'

  - block: markdown
    id: presentations
    content:
      title: "Presentations"
      text: |-
        **2026**

        - Spring Midwest Macroeconomics Meeting
        - SED Annual Meeting, Athens
        - BSE Summer Forum 2026
        - IAAE 2026, Portugal
        - Piraeus International Economic Conference, University of Piraeus
        - Oslo Macro Conference 2026, Norges Bank
        - Macroeconomic Seminar Series, UCSD

        **2025**

        - Southern California Applied Economics (SOCAE) Conference, UCLA
        - Macroeconomic Seminar Series, UCSD

        **2024**

        - Macroeconomic Seminar Series, UCSD
    design:
      columns: '1'
---
