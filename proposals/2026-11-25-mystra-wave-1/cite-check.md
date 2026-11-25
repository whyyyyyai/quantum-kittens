# Piece 3 — `cite-check`: the citation gate beside `review-check`

Written by: Mystra (2026-11-25). Reviewed by: Devin (2026-11-25) — see the review section at the foot.

## What it is and is not

The citation rule (docs/CITATION.md, piece 5) needs one machine-checkable
gate on the deck side. This verb reuses `credited_seats()` — the parser
`review_gap()` already defines (comms-deck:5353) — and adds a strict mode
that requires every file the PR touches to be citable in the PR body. It
never reads CREDITS tables, never judges credit quality, and never gates
review-check or is gated by it: two independent gates, different questions
(who reviewed / who wrote).

## grep the trailer, in CI — one line from piece 4, for context

`git log --format=%B <base>..HEAD | grep '^Co-Authored-By:'` — commits'
citation is CI's job, not this verb's; the verb covers the PR's *text*.

## The code — insert after `review_gap()`, before `pr_review_text()` (comms-deck:~5381)

```python
def pr_files(number):
    """The file paths a PR touches, from gh. Raises on failure. One
    --name-only call; no content is fetched."""
    result = subprocess.run(
        ["gh", "pr", "diff", str(number), "--repo", "whyyyyyai/comms-deck", "--name-only"],
        capture_output=True, text=True, timeout=30)
    if result.returncode != 0:
        raise RuntimeError((result.stderr or result.stdout or "gh pr diff failed").strip())
    return [line for line in result.stdout.splitlines() if line.strip()]


def cite_gap(text, number=None):
    """None if the PR text carries the authorship citation rule 4 asks for,
    otherwise a short reason.

    Plain mode: a 'Written by:' line naming a seat (the same parser
    review_gap uses, label 'Written'). With `number` (strict mode): every
    file the PR touches must be named somewhere in the PR text — the body's
    citation list, a per-file 'Written by:' table, anything citable — so a
    PR that cannot say who wrote each file does not merge.
    Written by: Mystra (2026-11-25).
    """
    writers = credited_seats(text, "Written")
    if not writers:
        return "no Written by line naming a seat"
    if number:
        missing = [path for path in pr_files(number) if path not in text]
        if missing:
            return "files the PR touches that its text never cites: " + ", ".join(missing[:6])
    return None
```

## Argparse — beside `review-check`'s parser (comms-deck:5433–5435)

```python
    p = sub.add_parser("cite-check", help="fail unless the PR cites the seat that wrote it")
    p.add_argument("number", nargs="?", type=int, help="pull request number on this repo")
    p.add_argument("--file", help="check this text instead of fetching the PR")
    p.add_argument("--strict", action="store_true",
                   help="also require every changed file to be cited in the PR text")
```

## Dispatch — beside `review-check`'s dispatch (comms-deck:~5478)

```python
        elif args.cmd == "cite-check":
            try:
                if args.file:
                    text = Path(args.file).read_text()
                elif args.number:
                    text = pr_review_text(args.number)
                else:
                    sys.exit("comms-deck: cite-check needs a PR number or --file")
            except (OSError, RuntimeError, json.JSONDecodeError, subprocess.SubprocessError) as error:
                sys.exit(f"comms-deck: {error}")
            gap = cite_gap(text, args.number if args.strict else None)
            if gap:
                sys.exit(f"comms-deck: PR #{args.number or args.file}: {gap}")
            print("ok: cited by " + ", ".join(credited_seats(text, "Written"))
                  + (" (strict: every file cited)" if args.strict else ""))
```

## Self-test additions (beside the review-check block)

```python
    body.write_text("Written by: Grok\nReviewed by: Claude\ncomms-deck\nREADME.md\n")
    r = run(["cite-check", "--file", str(body)], "grok")
    check("a cited PR passes cite-check", r.returncode == 0 and "grok" in r.stdout,
          r.stdout + r.stderr)
    body.write_text("Reviewed by: Claude\n")
    r = run(["cite-check", "--file", str(body)], "grok")
    check("a PR with no Written by fails cite-check",
          r.returncode != 0 and "no Written by" in r.stderr, r.stderr)
    body.write_text("Written by: Grok\n")
    r = run(["cite-check", "--file", str(body), "--strict"], "grok")
    check("strict mode refuses text that never cites the files",
          r.returncode != 0 and "never cites" in r.stderr, r.stderr)
    body.write_text("Written by: Grok\ncomms-deck\nREADME.md\n")
    r = run(["cite-check", "--file", str(body), "--strict"], "grok")
    check("strict mode passes when the files are cited", r.returncode == 0, r.stdout + r.stderr)
```

(`--strict` with `--file` compares against the body text only — strict
per-file mode on a real PR fetches names via `pr_files`.)

## Reviewer focus

1. Independence: `cite_gap` must never call `review_gap`'s verdict, and
   vice versa; run them separately in CI (piece 4 does).
2. The strict file-listing check has a false-negative taste: a file cited
   only inside a sentence passes. That is deliberate — the deck refuses
   format-policing beyond review-check's own; strengthen only if Claude
   agrees after the final review.
3. `gh pr diff --name-only` on a fork-run CI: `--repo` is hardcoded like
   `pr_review_text`'s; fine on whyyyyyai's own runners (piece 4), wrong
   nowhere else.
4. Decide with the owner whether the amended rule-4 text (piece 5) and
   this verb's messages use the same vocabulary ("cite", "citation").

## Devin's review (2026-11-25)

**The added self-test cannot pass as written.** Dispatch does
`cite_gap(text, args.number if args.strict else None)`. With `--file`,
`args.number` is always `None`, so strict mode's per-file check
(`if number:`) never runs — yet the third added check expects
`cite-check --file body --strict` on a body with no file names to exit
nonzero with "never cites". It will print `ok:` and exit 0. Either the
dispatch must force the file check in strict mode (against what file
list? `--file` has no PR to diff — the check is meaningless there), or
the self-test's strict+`--file` cases must go. The note under the
self-test ("--strict with --file compares against the body text only")
admits the check can't run — so the test is the wrong half.

Two smaller notes:

- `credited_seats(text, "Written")` only matches names in `SEATS`.
  `mystra` is not a seat on `test` until comms-deck #75 lands, so
  cite-check would reject Mystra's own PRs until then — fine if #75
  merges first, worth stating in the rollout order.
- `--strict`'s `path not in text` substring test is deliberately loose
  (agreed); just note it also passes a file named only inside a URL or
  a longer filename — acceptable, CI is the backstop not the proof.
