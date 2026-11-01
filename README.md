# chesterton

[ภาษาไทย](README.th.md) · From the maker of [honest-tests](https://github.com/nuttakulsv/honest-tests) and [phantom-deps](https://github.com/nuttakulsv/phantom-deps)

> Don't take down a fence until you know why it was put up. (G. K. Chesterton, roughly)

Coding agents love to clean up. They'll happily delete a `sleep(0.5)`, a retry loop or a weird special case for one store ID, because it looks pointless. Sometimes that line is the only thing between you and a 3 a.m. page, and the reason isn't in the code. It's in the git history.

chesterton is a skill that makes your agent check why odd code exists before removing it, plus a small script, `why.py`, that collects the history behind any lines into one page. It's useful for humans too.

## What `why.py` shows

```
$ python3 why.py src/cache.py:6

# Why: src/cache.py:6-6 (in `warm_cache`)

    6      time.sleep(0.5)  # do not remove: replica lag, see #42

## Warning signs and notes nearby
- `time.sleep(0.5)  # do not remove: replica lag, see #42`

## Commits that shaped these lines (newest first)
- **81d18ed** 2024-03-02 Dana: Wait for replica before reading back (#57)
  > Reads right after load() hit the replica
  > before it caught up. Fixes #42.

## Pull requests
- #57 Wait for replica before reading back (https://github.com/acme/shop/pull/57)
  > We saw stale prices for ~300ms after each warm. This waits...

## Issues
- #42 Customers see old prices right after a price update (...)

## Tests that mention the same names
- tests/test_cache.py:3:def test_warm_cache():  (mentions `warm_cache`)
```

It uses `git log -L` to follow those exact lines through renames and edits, pulls in full commit messages, and fetches the linked PRs and issues with the GitHub CLI (`gh`) if you have it. It never changes anything.

## What the skill makes the agent do

Before it removes or rewrites sleeps, retries, special cases, magic numbers, swallowed exceptions, compatibility shims, or anything near a "do not remove" comment, the agent:

1. runs `why.py` on those lines,
2. reads the linked PR or issue,
3. looks for a test that covers it,
4. then keeps it (adding a one-line "why" comment if it's missing), removes it with the reason in the commit message, or leaves it and tells you it couldn't find a reason.

At the end, it lists what it kept and removed, and why:

```
Kept    time.sleep(0.5) in warm_cache: replica lag after load() (81d18ed, PR #57, fixes #42)
Removed legacy v1 header check: v1 clients were retired in PR #210
Unsure  special case for store_id 4471: no history found, left in place
```

You can also just ask: "why is this code here?"

## Install

Claude Code:

```
/plugin marketplace add nuttakulsv/chesterton
/plugin install chesterton@chesterton
```

Codex, Cursor, or anything else that reads SKILL.md:

```
npx skills add nuttakulsv/chesterton
```

Or use the script on its own. It only needs `python3` and `git`:

```
curl -O https://raw.githubusercontent.com/nuttakulsv/chesterton/main/skills/chesterton/why.py
python3 why.py path/to/file.py:120-128
```

## Limits

- It's only as good as your history. Squash merges with messages like "fix stuff" leave little to find.
- PR and issue details need `gh`, logged in, in a repo with a GitHub remote. Without it you still get the numbers.
- "Tests that mention the same names" is a text search, not coverage.

## Tests

```
python3 skills/chesterton/why.py --test
```

## License

MIT
