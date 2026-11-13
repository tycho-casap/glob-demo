# glob-demo

Mock files (tag `bal`, content: baked alaska instructions with web sources) that
probe how CODEOWNERS and a ruleset's required-reviewer `file_patterns` read
ambiguous glob forms. Each `probe/*` PR edits exactly one file; compare whether
CODEOWNERS auto-requests a review and whether the ruleset blocks the merge.
Results are filled in below once the PRs are open.
