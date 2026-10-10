# leaf_spy_doc

Documentation for **leaf_spy**, a small header-only wrapper around [Boost.LEAF](https://www.boost.org/doc/libs/release/libs/leaf/doc/html/index.html) that sees any set of types in an exception.

Published site: <https://goloveshin.github.io/leaf_spy_doc/>

## Layout

| path | what it is |
| --- | --- |
| `source/` | the pages (Markdown) |
| `source/images/` | SVG pictures. **Generated** from `drawio/`, but committed, so the site builds without draw.io |
| `drawio/` | the **source** of the pictures |
| `export_drawio.ps1` | exports `drawio/*.drawio` to `source/images/*.svg` |
| `mkdocs.yml` | MkDocs configuration |
| `.github/workflows/pages.yml` | builds the site and publishes it to GitHub Pages |

## Build locally

```text
pip install mkdocs
mkdocs serve          # http://127.0.0.1:8000, reloads on changes
mkdocs build --strict # writes the site to site/, fails on broken links
```

## Edit a picture

1. Edit the file in `drawio/` with [draw.io](https://www.drawio.com/).
2. Export it again:

   ```text
   powershell -File export_drawio.ps1
   ```

   This needs the draw.io desktop application. Pass `-DrawioExe <path>` if it is not in `C:\Program Files\draw.io`.
3. Commit the `.drawio` file and the new `.svg` together.

## Publish

Every push to `main` publishes the site through the workflow in `.github/workflows/pages.yml`.

Set it up once in the repository on GitHub: **Settings, Pages, Build and deployment, Source: GitHub Actions**.

## License

[Boost Software License 1.0](LICENSE)
