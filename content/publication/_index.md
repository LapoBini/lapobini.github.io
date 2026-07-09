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

        - <span class="pres-marker">†</span> <a href="https://www.marquette.edu/business/economics/macro-meeting.php" target="_blank" rel="noopener">Spring Midwest Macroeconomics Meeting</a>
        - <span class="pres-marker">†</span> <a href="https://economicdynamics.org" target="_blank" rel="noopener">SED Annual Meeting, Athens</a>
        - <span class="pres-marker">†</span> <a href="https://bse.eu/summer-forum/workshops/structural-shocks" target="_blank" rel="noopener">BSE Summer Forum 2026 — Advances in Structural Shocks Identification</a>
        - <span class="pres-marker">†</span> <a href="https://iaae2026.pages.dev" target="_blank" rel="noopener">IAAE 2026, Portugal</a>
        - <span class="pres-marker">†</span> Piraeus International Economic Conference, University of Piraeus
        - <span class="pres-marker">†</span> <a href="https://sites.google.com/site/oslomacro/omc" target="_blank" rel="noopener">Oslo Macro Conference 2026, Norges Bank</a>
        - <span class="pres-marker">†</span> Macroeconomic Seminar Series, UCSD

        **2025**

        - <span class="pres-marker">†</span> <a href="https://sites.google.com/view/socae/past-editions" target="_blank" rel="noopener">Southern California Applied Economics (SOCAE) Conference, UCLA</a>
        - <span class="pres-marker">†</span> Macroeconomic Seminar Series, UCSD

        **2024**

        - <span class="pres-marker">†</span> Macroeconomic Seminar Series, UCSD
    design:
      columns: '1'
---
