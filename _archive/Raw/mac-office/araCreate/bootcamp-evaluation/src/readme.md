# SOURCE

The dashboard itself.

| Path | What it is |
| --- | --- |
| `server.js` | The whole API, plus a small `.env` loader |
| `db.js` | The PostgreSQL pool and the startup connection check |
| `db/` | Schema, migrations, and the roster loaders |
| `public/` | The front end, and the araCreate design system it is built on |

There is no build step. `public/index.html` links the design system directly.

Third-party code you do not maintain is vendored here as git submodules, not
edited in place.
