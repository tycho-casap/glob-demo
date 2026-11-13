# glob-demo

Mock files (tag `bal`, content: baked alaska instructions, sources linked in each
file) that probe how CODEOWNERS and a ruleset's required-reviewer `file_patterns`
read ambiguous glob forms. Each `probe` PR edits one file; the sole CODEOWNERS
owner is `@tychoish`, so "review requested" means the file matched a rule.

## Results (measured)


| PR | Rule | File touched | CODEOWNERS review requested | Ruleset blocks merge |
|----|------|--------------|------------------|
| #1 | `/desserts/baked-alaska/**` | `desserts/baked-alaska/steps.md` | yes | same |
| #2 | same | `desserts/baked-alaska/deep/nested/serve.md` | yes | same |
| #3 | `/**/*_bal.go` | `meringue_bal.go` (repo root) | yes: `/**/` matches zero dirs | same |
| #4 | same | `kitchen/prep/ice_cream_bal.go` | yes | same |
| #5 | `**/*_bal_test.go` (no slash) | `kitchen/prep/ice_cream_bal_test.go` | yes | same |
| #6 | `/**/*_bal.gen.go` | `kitchen/gen/torch_bal.gen.go` | yes | same |
| #7 | `/scripts/bal*/**` | `scripts/bal-oven/broil.md` | yes: `*` in a middle segment works | same |
| #8 | same | `scripts/bal-freezer/chill.md` | yes | same |
| #9 | `/menu/bal**` | `menu/bal-special.md` | yes: `bal**` behaves like `bal*` | same |
| #10 | same | `menu/bal-special/notes.md` | yes: matched via the directory | same |
| #11 | `/recipes/*.md` | `recipes/classic.md` | yes | same |
| #12 | same | `recipes/regional/alaska.md` | **no**: `*` does not cross `/` | same |
| #13 | `docs/frozen/**` (no leading slash) | `docs/frozen/storage.md` | yes | same |
| #14 | same | `site/docs/frozen/storage.md` | **no**: a mid-path slash anchors the pattern | same |
| #15 | `/src/*/torch.md` | `src/ovens/torch.md` | yes | same |
| #17 | same | `src/ovens/deep/torch.md` | **no** | same |
| #18 | `/notes/torch.md` | `notes/torch.md` (added) | yes | same |
| #16 | none (control) | `unowned/control.md` | no | same |

PR numbers are not in probe order for the last three (#16 control, #17 and #18 are
probes 16 and 17).

Caveat: review requests were empty right after the PRs were opened in a burst and
appeared only after close and reopen. Read results after that re-evaluation.


## Ruleset

`ruleset.json` is the `main` ruleset: required reviewer is the one-member team
`baked-alaska-reviewers`, with the same 10 patterns written without leading slashes.
A PR is "blocked" exactly when the reviewer rule applies (the control PR is clean).
A repository must be owned by an organization for `required_reviewers` to exist, and
the reviewer must be a team (`type: "User"` is rejected); the repo was transferred from
a personal account for that reason.

Finding: for these 18 files the ruleset and CODEOWNERS agree everywhere, including
`**/*_bal.go` at the repo root, `bal*/` in a middle segment, `menu/bal**`, and the
non-matches (`recipes/*.md` two levels deep, `docs/frozen/**` nested, `src/*/torch.md`
deep).

Not covered: forms the ruleset documentation does not describe (leading `/`, `?`, `[]`)
and whether the repository's own validator accepts what GitHub accepts.

## Extra forms (PRs #21 to #38, base `tycho/extra-codeowners`)

Their base branch carries the extra CODEOWNERS lines (PR #20, unmerged) and is added
to the extra ruleset's ref conditions (`ruleset-extra.json`). All ten ruleset patterns
were accepted at creation, so only the PRs show what they match.

| PR | Pattern (ruleset / CODEOWNERS) | File added | CODEOWNERS | Ruleset |
|----|--------------------------------|------------|------------|---------|
| #21 | `lead/slash.md` with a leading `/` | `lead/slash.md` | yes | blocks: a leading `/` is accepted and matches |
| #22 | `qm/file?.md` | `qm/file1.md` | yes | blocks |
| #23 | same | `qm/file12.md` | no | no: `?` is exactly one character |
| #24 | `br/q[0-9].md` | `br/q5.md` | **no**: GitHub flags it "Invalid pattern" | **blocks** |
| #25 | same | `br/qa.md` | no | no |
| #26 | `cb/{a,b}.md` | `cb/a.md` | no | no: no brace expansion |
| #27 | same | `cb/{a,b}.md` (literal name) | yes | blocks: braces are literal |
| #28 | `!neg/x.md` | `neg/x.md` | no | no: no negation, `!` is literal |
| #29 | `mid/**/leaf.md` | `mid/leaf.md` | yes | blocks: `/**/` matches zero dirs |
| #30 | same | `mid/x/leaf.md` | yes | blocks |
| #31 | same | `mid/x/y/leaf.md` | yes | blocks |
| #32 | `dirslash/` | `dirslash/f.md` | yes | blocks |
| #33 | same | `dirslash/sub/g.md` | yes | blocks |
| #34 | `*.balx` | `root.balx` | yes | blocks |
| #35 | same | `a/b/nested.balx` | yes | blocks: a slash-free pattern matches at any depth |
| #36 | `case/upper.md` | `Case/upper.md` | no | no: case sensitive |
| #37 | same | `case/upper.md` | yes | blocks |
| #38 | `menu/bal**/deep/**` | `menu/bal-special/deep/more.md` | yes | blocks |

The only disagreement between CODEOWNERS and the ruleset is `[0-9]`: the ruleset
honors it, CODEOWNERS rejects it. Everything else matches.
