# imprenta-template

A starting point for a content repo arranged and rendered by
[imprenta](https://github.com/jmcmeen/imprenta): some lorem ipsum, a theme, a
catalog, and the Makefile and CI that drive them. Use it as a GitHub template
("Use this template"), then replace the content and the catalog with your own.

Each release of this template pins the imprenta version it was made for
(`pyproject.toml`); template `vX.Y.Z` means "set up for imprenta X.Y.Z". The
pin is the only version there is: the project version and the release tag
repeat it, and CI refuses a release where they differ. See
[Versions](#versions) for how to upgrade a repo made from it.

## Setup

```sh
brew install uv pandoc node          # or your platform's equivalents
brew install --cask google-chrome    # pagedjs-cli drives it to make PDFs
make setup                           # uv sync, npm ci, fetch the theme fonts, check the tools
make                                 # check, then build every product
```

## What is here

```text
catalog.main.yaml      the catalog: rendering variants, labels, groups, recipes, products
content/               the content: <name>.md files; the name is the path without .md
  lorem.md, ipsum.md       two chapters with term tables (front matter `label`, own H1, H2 sections)
  lorem-cover.md, …        covers: an eyebrow paragraph, the H1 title, an optional subtitle
  shared/welcome.md, …     reusable parts, written once and placed by name
templates/             the theme: leaflet.html and volume.html (pandoc templates), print.css,
                       epub.css, sizes/, palettes/, fonts/
scripts/fetch_fonts.py downloads the theme fonts (run by make setup)
output/, tmp/          generated, git-ignored; output is flat: <product>.main.<variant>.<ext>
```

Three products are defined: `lorem` and `ipsum` (each a leaflet from the
`leaflet` recipe: cover, welcome, the chapter, a glossary) and `omnibus` (a
volume: cover, table of contents, foreword, both chapters, the combined
glossary, a colophon). `make lorem`, `make omnibus`, `make main/lorem.ebook`.

## Make it yours

1. Replace the files in `content/`. Keep the shape: optional front matter,
   one `# Title`, sections at `##`.
2. Edit `catalog.main.yaml`: your groups, your recipes, your products. Add
   `catalog.<other>.yaml` files for other sets of products (another language,
   a different line) that share the same content.
3. Restyle `templates/`: `print.css` and `epub.css` hold the colors and type,
   `palettes/` and `sizes/` the layers variants pick from, the two `.html`
   files the page frame. Themes are picked per product.
4. `make check` must pass. `make manager` opens a dashboard over it all.
5. Leave the comment next to the imprenta pin in `pyproject.toml` ("started
   from imprenta-template vX.Y.Z"): it is how you later find the template
   release to diff against when you upgrade.

Publishing is yours to add: give products tags (`tags: [release]`) and let your
own CI select them with `imprenta build --tag release`, then upload wherever
you ship.

## Rules the checker enforces

- `content/` holds only `.md` files. Every file starts with its own H1 after
  optional front matter (a styled part such as a cover may open with a
  paragraph), has one H1, sections at H2, nothing deeper than H4.
- Every name in a parts list is `toc`, `terms`, a group or a content file.
- Every product has a status (`draft | ready | published`), a quoted version
  once it leaves draft, known variants and string tags.
- Every rendering variant points at css layers that exist.

## Versions

The template follows the engine's version numbers, so a content repo can
always tell which template a given imprenta release expects.

- Template `vX.Y.Z` is set up for imprenta `X.Y.Z` and pins exactly that.
- Patch releases of imprenta (`0.1.1`, `0.1.2`) change nothing a content repo
  sees, so they get no template release: bump the pin, run `uv lock`, done.
- Minor and major releases (`0.2.0`, `1.0.0`) get a template release of the
  same number whenever the catalog schema, the template variables, the
  Makefile or the CI changed. A template tag is the signal that an upgrade
  involves more than the pin.

### Upgrading a repo made from this template

1. Bump `imprenta[tui]==X.Y.Z` in `pyproject.toml` with the engine's release
   notes open, then run `uv lock`.
2. If a template release of that number exists, read `CHANGELOG.md` between
   the tag you started from (the comment next to the pin) and the new one,
   and apply what it names: `git diff vA.B.C vX.Y.Z -- Makefile .github
   templates` against a clone of this repo shows the exact lines.
3. Update the comment next to the pin to the new template release, then
   `make check` and `make`.

### Releasing the template (maintainers)

1. Set the pin and the project version in `pyproject.toml` to the new
   imprenta release, run `uv lock`, make the template changes, and record them
   in `CHANGELOG.md` under that number.
2. Merge to `main`, then tag the merge commit `vX.Y.Z` and push the tag. CI
   checks that the tag, the project version and the pin agree.
