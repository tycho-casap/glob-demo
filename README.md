# glob-demo

Mock files (tag `bal`, content: baked alaska instructions, sources linked in each
file) that probe how CODEOWNERS and a ruleset's required-reviewer `file_patterns`
read ambiguous glob forms. Each `probe` PR edits one file; the sole CODEOWNERS
owner is `@tychoish`, so "review requested" means the file matched a rule.

## CODEOWNERS result (measured)

| PR | Rule | File touched | Review requested |
|----|------|--------------|------------------|
| #1 | `/desserts/baked-alaska/**` | `desserts/baked-alaska/steps.md` | yes |
| #2 | same | `desserts/baked-alaska/deep/nested/serve.md` | yes |
| #3 | `/**/*_bal.go` | `meringue_bal.go` (repo root) | yes: `/**/` matches zero dirs |
| #4 | same | `kitchen/prep/ice_cream_bal.go` | yes |
| #5 | `**/*_bal_test.go` (no slash) | `kitchen/prep/ice_cream_bal_test.go` | yes |
| #6 | `/**/*_bal.gen.go` | `kitchen/gen/torch_bal.gen.go` | yes |
| #7 | `/scripts/bal*/**` | `scripts/bal-oven/broil.md` | yes: `*` in a middle segment works |
| #8 | same | `scripts/bal-freezer/chill.md` | yes |
| #9 | `/menu/bal**` | `menu/bal-special.md` | yes: `bal**` behaves like `bal*` |
| #10 | same | `menu/bal-special/notes.md` | yes: matched via the directory |
| #11 | `/recipes/*.md` | `recipes/classic.md` | yes |
| #12 | same | `recipes/regional/alaska.md` | **no**: `*` does not cross `/` |
| #13 | `docs/frozen/**` (no leading slash) | `docs/frozen/storage.md` | yes |
| #14 | same | `site/docs/frozen/storage.md` | **no**: a mid-path slash anchors the pattern |
| #15 | `/src/*/torch.md` | `src/ovens/torch.md` | yes |
| #17 | same | `src/ovens/deep/torch.md` | **no** |
| #18 | `/notes/torch.md` | `notes/torch.md` (added) | yes |
| #16 | none (control) | `unowned/control.md` | no |

PR numbers are not in probe order for the last three (#16 control, #17 and #18 are
probes 16 and 17).

Caveat: review requests were empty right after the PRs were opened in a burst and
appeared only after close and reopen. Read results after that re-evaluation.

## Ruleset half (not run)

`required_reviewers` rulesets need GitHub Pro or a public repository; the create call
returned 403 "Upgrade to GitHub Pro or make this repository public". The patterns that
would be tested are in the PR description of the results PR.
