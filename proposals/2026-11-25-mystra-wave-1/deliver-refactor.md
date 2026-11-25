# Piece 1 — `deliver()`: one delivery path, extracted, zero behavior change

Written by: Mystra (2026-11-25). Reviewed by: nobody yet.

## Why

Today a note is delivered by two separate code paths that must never drift:
`send()`'s tail (comms-deck:659–674) and `import_remote()`'s hand-off loop
(comms-deck:2390–2399). Anything that changes delivery (load-balancer's
interception, remote-seat receipts) must currently edit both. This piece
extracts one `deliver()` and makes both callers use it — nothing else.

The self-test already covers both paths end-to-end (send/fetch-remote
checks), so it is the regression net. No new checks are added by design:
that would test the refactor, not the behavior.

## The new function — insert right before `def send(...)`, SECTION 4

```python
def deliver(record, targets, enqueue=True):
    """One delivery path, shared by send() and import_remote(): each target
    gets the note (an inbox file; the desktop bell for the owner; nothing
    for a remote seat, whose mailbox is the git copy), then the git queue,
    then the log, then the @deck actions.

    `enqueue=False` is import_remote's case: the record came FROM the git
    copy of the flight-log, so queueing it again would commit it back a
    second time.

    Written by: Mystra (2026-11-25); the body is send()'s tail
    (comms-deck 659-674) moved rather than changed, with the `if enqueue`
    added around `queue_out`.
    """
    for seat in targets:
        if seat == "owner":
            notify(f"Note from {record['from']}", record["text"])
        elif APP_PROCESS.get(seat) == "remote":
            pass  # a remote seat (docs/REMOTE.md) takes its notes from the git
                  # copy; nothing here would ever take it out of an inbox, so
                  # the squares on the screen must not count one that won't be read
        else:
            write_json(DATA / "inbox" / seat / f"{record['id']}.json", record)
    if enqueue:
        queue_out(record)
    append_log(record)
    queue_actions(record)
```

## The two callers

**`send()`** — its tail becomes exactly this (the comment block about
recipients stays where it is, above):

```python
    deliver(record, targets)
    return record, warning
```

**`import_remote()`** — the inner block becomes exactly this:

```python
            deliver(record, targets, enqueue=False)
            brought += 1
```

## Order of operations (unchanged, restated so a reviewer can check it)

`send()`: inboxes/bell/remote-skip → `queue_out` → `append_log` →
`queue_actions`. `import_remote()`: inboxes/bell (no `queue_out`) →
`append_log` → `queue_actions`. Both now read as one line each, and a
future interception point (`deliver()`) exists for the load-balancer.

## Reviewer focus

1. Diff the extracted body against comms-deck:659–674 word by word — the
   only difference is `if enqueue:` around `queue_out(record)`.
2. Confirm no caller of `send()` checks anything the old tail produced
   (it returned nothing but `record, warning` — unchanged).
3. Run `comms-deck self-test`: every existing check should pass untouched.
