# Ruleset records

The JSON of a repository's ruleset, recorded by hand as `gh api` returns it,
before and after each change (ADR-0021 §10). It lives here, in a repo other
than the one whose rules it describes. With one human, the record and the
rules have the same author, so the ruleset history endpoint is the read that
does not depend on this record:

    gh api repos/platform-factory/<repo>/rulesets/<id>/history

Files are named `<UTC time>-<step>-<before|after|history>-<change>.json`.

## edge-config, ruleset 20204888 (`main-protection`)

| Step | Change | History version |
|---|---|---|
| 01 → 02 | C-09: required reviewers, team `security`, one approval, `external/**` | 45124681 → 55281350 |
| 03 → 04 | C-10: `route files` (GitHub Actions) becomes a required status check, after it failed on #10–#17 and passed on #6–#9 | 55281350 → 55281693 |
