# SOURCE

Two deployables, one folder each, per
[repo §2.3](https://github.com/aracreate-group/aracreate-conventions/blob/main/repo/readme.md#23-multi-service-repos).

| Folder | What it is |
| --- | --- |
| [web/](web/) | Next.js app — the private dashboard plus the public widget API |
| [widget/](widget/) | The embeddable script that runs on the client's live site |

Depth for either lives in `docs/web/` and `docs/widget/`, not here. Each folder
keeps a short `readme.md` saying what it is and how to run it.
