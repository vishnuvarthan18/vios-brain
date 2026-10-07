# DATABASE

| File | What it does |
| --- | --- |
| `schema.sql` | Every table, trigger and view. Builds from nothing |
| `load-eee.sql` | The real EEE data: 14 teams, 55 students |
| `load-ece.sql` | The real ECE data: 38 teams, 151 students. Runs after `load-eee.sql` |
| `migration-team-codes.sql` | Puts every team on one code format, `DEPT-TNN-TEAMNAME` |
| `migration-profile.sql` | Adds profiles to a database built before that feature |
| `seed.sql` | Placeholder data for a scratch database. Not for a real run |

**The loaders are setup scripts, not start commands.** `load-eee.sql` and
`load-ece.sql` each begin by deleting their department's students, which
cascades to every daily post, attendance mark and quiz answer those students
have. Run them on an empty database and never again.
