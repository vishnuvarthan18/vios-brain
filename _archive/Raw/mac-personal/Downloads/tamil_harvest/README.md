# Semmozhi (செம்மொழி)

A reading room for Tamil language, literature and history: real texts and articles, free tools and fonts for the old Tamil scripts, every source linked back with its licence. Honest about what is and is not proven.

**Live site:** deployed from main. https://www.semmozhi.online loads.
**Preview of `dev`:** https://dev.semmozhi.online
**Admin (private):** https://engine.semmozhi.online

## The project has three parts

| Part | Public? | Lives in | In one line |
|---|---|---|---|
| **Website** | **Yes** | [`website/`](website/) | Static site: Home, Cholas, Literature, About, and the Lab (Brahmi, Grantha, Vatteluttu, Tamil, fonts). No build step. |
| **Design system** | No | [`design/`](design/) and `design/showcase/` | Tokens, components, style guide, identity gallery, realism experiments, reference-photo tools. |
| **Data engine** | No | `tamil_harvest/`, `engines/`, `scripts/`, `vps/`, `viewer_app/`, `dashboard/` | Scrapy crawlers and builders that collect open Tamil texts and turn them into the JSON the website reads. |

Only `website/` is ever published. `bash deploy.sh` and the GitHub Actions upload that folder and nothing else.

## Quick start

```bash
git clone https://github.com/vishnuvarthan18/tamil-data-collector.git
cd tamil-data-collector
git checkout dev
make local        # then open http://localhost:8000/local/  (a hub that links to every page)
make check        # run before every pull request
```
Needs Python 3.12 and Node 22 or newer (for the syntax check). Full setup: [docs/GETTING_STARTED.md](docs/GETTING_STARTED.md).

## How we work

- `main` = production. It is deployed automatically. Nobody commits to it directly.
- `dev` = integration. Branch from `dev`, open a pull request into `dev`. Merging `dev` into `main` is a release.
- Every pull request runs the checks. Pushes to `dev` publish the preview site.
- Details: [CONTRIBUTING.md](CONTRIBUTING.md).

## Documentation

| Read this | When |
|---|---|
| [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) | You want the big picture: how data becomes pages, what talks to what |
| [docs/GETTING_STARTED.md](docs/GETTING_STARTED.md) | First day: set up, run, check |
| [docs/DEPLOYMENT.md](docs/DEPLOYMENT.md) | Publishing, domains, secrets, rollback |
| [docs/ENGINE.md](docs/ENGINE.md) | Crawling, adding a source, rebuilding site data |
| [docs/CONTENT_RULES.md](docs/CONTENT_RULES.md) | Writing, sourcing, licences, the "unverified" rule |
| [docs/KNOWN_ISSUES.md](docs/KNOWN_ISSUES.md) | What is unfinished or risky, and decisions the team still has to make |
| [website/README.md](website/README.md), [design/README.md](design/README.md) | Per-part detail |
| [PROJECT_MAP.md](PROJECT_MAP.md) | Folder-by-folder map |
