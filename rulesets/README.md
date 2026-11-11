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
| 05 → 06 → 08 | C-13's flip, 2026-11-03: the `security` approval for `external/**` lifted at 19:10:50Z so edge-config#7 could merge without `--admin` (merged 19:10:58Z, `b1c3f99`), and restored at 19:11:00Z; 08 equals 05. One person cannot give the approval (C-09) | 55281693 → 56118778 → 56118811 |
| 09 → 10 | ADR-0023 §2's guard, 2026-11-11: `.github/**` added to the `security` team's required-review patterns, beside `external/**`, so a pull request that edits the workflow producing `route files` needs the approval. Read on edge-config#19 afterwards: CLEAN before, BLOCKED with `security` requested after | 56118811 → 57378541 |
