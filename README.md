# lapobini.github.io

Source of my personal website, [lapobini.github.io](https://lapobini.github.io). The layout and style
changes are listed in [Customizations](#customizations).

## How the site is published

Every push to `main` triggers the GitHub Action in `.github/workflows/publish.yaml`. It builds the site
and deploys it to GitHub Pages, usually within 1–2 minutes. Check progress in the **Actions** tab of
the repository. Nothing else needs to be built or uploaded by hand: the `public/` folder and the build
caches are generated on the fly and are not tracked by git (see `.gitignore`).

After a deploy, browsers may show the old version for up to 10 minutes. Reload the page to see the
update.

## Folder structure

```
config/_default/     site settings: params.yaml (theme, colors), menus.yaml (top menu), hugo.yaml
content/
  _index.md          homepage sections (bio, latest research)
  authors/admin/     bio, role, education, social links (_index.md) and profile photo (avatar.jpg)
  publication/       working papers, one folder per paper (index.md + PDFs)
  ongoing/           ongoing projects, one folder per project
  teaching/          one folder per course page (index.md + downloadable .zip)
  discussion/        PDFs of the discussion sections, one folder per course
  project/           "Code" pages (Bayesian VAR, Monetary Economics)
  resume.pdf         the CV opened by the CV menu link
data/coauthors.yaml  personal websites of coauthors (their names become links)
layouts/             customized page templates
static/              files copied as is (favicons)
assets/media/        site icon
```

## Common tasks

### Replace a paper, appendix or slides

Overwrite the file in the paper's folder **keeping the same file name**, e.g.
`content/publication/100years/100years.pdf`. All buttons and links pick it up automatically.

| Paper | PDF | Appendix | Slides |
|---|---|---|---|
| Supply chain (`publication/SupplyChain/`) | `Bini2025.pdf` | `appendixBini2025.pdf` | `slidesBini2025.pdf` |
| 100 years (`publication/100years/`) | `100years.pdf` | `OnlineAppendix-100years.pdf` | `slides-100years.pdf` |
| Nowcasting (`ongoing/nowcasting_inflation/`) | `Main_NowcastingInflation.pdf` | | |

The supply chain paper also has a downloadable shock series, `IV_shock.xlsx`.

### Edit a paper's details

Everything is in the front matter of the paper's `index.md`:

| Field | What it does |
|---|---|
| `title`, `authors`, `date` | Title, author list (`admin` is me) and the year shown |
| `abstract` | Abstract (homepage toggle and Research page). Math works with `$...$` |
| `status` | Grey italic line under the title, e.g. `"Revise & Resubmit, The Economic Journal"` |
| `update` | Small blue badge after the title, e.g. `"New draft"`. Delete the line to remove it |
| `awards` | List of awards, joined with `;` |
| `presentations` | Talks by year. An event is a plain name or `name:` + `url:` |
| `title_url` | Where the title links: a file in the folder (`100years.pdf`) or a web address |
| `url_pdf`, `url_slides`, `links` | Files behind the PDF, Slides and Appendix buttons |
| `shock_data` | File behind the Shock Data button |

Example of `presentations`:

```yaml
presentations:
  - year: "2026"
    events:
      - name: SED Annual Meeting, Athens
        url: https://economicdynamics.org
      - Macroeconomic Seminar Series, UCSD
```

### Add a new paper

Copy an existing folder in `content/publication/` (or `content/ongoing/` for an ongoing project), rename
it, replace the PDFs and edit `index.md`. Keep the `_build` lines, which stop the build from generating a
separate page for the paper.

### Update the CV

Replace `content/resume.pdf`. The master copy is compiled from
`~/Projects/CV_Letters_template/CV/LAPO_cv.tex` (LuaLaTeX).

### Edit the bio

Edit `content/authors/admin/_index.md`. The profile photo is `avatar.jpg` in the same folder.

### Link a coauthor's name to their website

Add a line to `data/coauthors.yaml`, with the name written exactly as in the papers' `authors` list:

```yaml
"Lucrezia Reichlin": https://www.lucreziareichlin.eu
```

### Add teaching material

1. Copy the PDF into `content/discussion/<course>/` (e.g. `ECON120A_fall24/120A_26_5.pdf`).
2. Add a link in the course page `content/teaching/<course>/index.md`, under the right class:
   ```markdown
   5. [Midterm 2 Review](https://lapobini.github.io/discussion/ECON120A_fall24/120A_26_5.pdf)
   ```
3. Add the same PDF to the course's `.zip` in `content/teaching/<course>/`:
   ```bash
   cd content/discussion/ECON120A_fall24
   zip ../../teaching/ECON120A_fall24/ECON120A_slides.zip 120A_26_5.pdf
   ```

Office hours and general info are at the top of each course page. The order of the courses is set by
`weight` in each page's front matter.

### Edit the Code pages

`content/project/Bayesian VAR/index.md` and `content/project/ECON210C/index.md` are Markdown with LaTeX
math (`$...$` inline, `$$...$$` for displayed equations). Keep long equations split over several lines
(`\begin{aligned} ... \\ ... \end{aligned}`), otherwise they do not fit on a phone screen. Citations link
to the reference list at the bottom of the page:

```markdown
[Litterman](#ref-litterman1986) ([1986](#ref-litterman1986))          <!-- in the text -->
<span id="ref-litterman1986"></span>**Litterman, Robert B.** 1986. ...  <!-- in References -->
```

## Customizations

Files in `layouts/` replace the default page templates:

| File | Change |
|---|---|
| `partials/views/citation.html` | Paper entries: title, status, update badge, awards, buttons, abstract and talks |
| `partials/page_metadata_authors.html` | Coauthor names link to `data/coauthors.yaml` |
| `partials/views/teachcard*.html` | Course and code cards on the Teaching page |
| `partials/components/sidebar.html` | Left menu on teaching/code pages; same top menu as the rest of the site on phones |
| `partials/components/last-edited.html` | No "Last updated" date |
| `partials/components/next-in-series.html` | Previous/next links without dates |
| `partials/docs_layout.html` | Teaching/code pages without the breadcrumb bar |
| `partials/blox/resume-biography-3.html` | Homepage bio: Download CV button under the social icons, no Interests/Education grid |
| `partials/site_footer.html` | Footer without the template credit line |
| `_default/single.html` | Individual paper pages: no date, type row or "Last updated" |
| `partials/hooks/head-end/custom.html` | **All custom CSS**: colors, spacing, phone layout |
