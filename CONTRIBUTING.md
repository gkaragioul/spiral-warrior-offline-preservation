# Contributing

This project is **paused and looking for collaborators**. The original author
has taken it as far as they intend to; everything below is an honest map of
where it stands so someone else can pick it up.

This repository is archived, so it no longer accepts issues or pull requests.
To carry the work on, fork it. The maintained copy is in the
[Game Preservation Hub](https://github.com/gkaragioul/game-preservation-hub) under `projects/spiral-warrior`. There is no CLA
and no style bureaucracy — but there are a few rules that exist for good
reasons.

## Ground rules

**Never commit game content.** No APKs, no assets, no decrypted or decompiled
source, no keys, no screenshots of the client, no packet captures. The
`.gitignore` already blocks the obvious cases; do not work around it. See
[NOTICE](NOTICE) for the full boundary.

**Bring your own copy.** The tooling operates on a client you supply. Nothing
here downloads, distributes or unlocks the game.

**Evidence, not assertions.** This codebase was built by reading the client's
own behaviour, and it was wrong several times in ways that looked right. If
you claim the protocol does something, say how you know — a captured frame, a
line in the decompiled bundle, an observed refusal. `docs/BLOCKERS.md` records
the four contracts that were modelled wrongly and how each was caught; that
format is the house style.

**Test-first, and watch it fail.** Every behavioural change should have a test
that fails before the change and passes after. Never edit an assertion to make
a failure disappear — if a test is wrong, say why it encoded a wrong belief
about the client, and cite the evidence that corrects it.

**Deny by default.** An unknown route returns 404; an unknown net ID is logged
and left unanswered. The gateway must never fabricate a success it cannot
justify. A request it cannot honour closes the socket rather than guessing.

## Running the tests

```powershell
py -3.12 -m venv server\.venv_win
server\.venv_win\Scripts\python.exe -m pip install -r server\requirements-win-py312.lock
server\.venv_win\Scripts\python.exe -m pip install -e server
server\.venv_win\Scripts\python.exe -m pytest server\tests -q -m "not preservation_artifact"
```

A clean checkout runs 350 of 365 tests. The other 15 are marked
`preservation_artifact`: they need a locally supplied client and the
`SPIRAL_BUNDLE_KEY` environment variable, and they are the tests that verify
the committed data tables still match the shipped ones.

Run the full suite with the live stack stopped. Several launcher tests bind
the real service ports and share launcher state, so they fail intermittently
while a session is running.

## Where help is most useful

Roughly in order of value to the project:

1. **The tournament mode (锦标赛).** The other main game mode, and the
   lobby's own tutorial quest still points at it. Some scaffolding exists
   (net 166/167 `ChapterOpr`), but the mode is unmodelled.

2. **Chapters beyond the prologue.** Only `BelongChapter == -1` is modelled.
   The extraction tooling in `tools/extract_roguelike_*.py` already takes a
   `--chapter` argument, so the path is mostly mechanical — but every new
   chapter is a chance to find another contract that was assumed rather than
   read.

3. **Chapter progression the client can see.** A clear is now recorded and
   `RogueLikeRecord` reaches the client, but the exploration percentage and
   the first-clear reward chest on the Adventure entry screen are still
   driven by `ChapterRecords`, which is not modelled.

4. **The paid revive.** `RL_MiscOpr_Diamond` (7) is refused at the parser —
   `RogueLike_Diamond` sends only `OprType` with no `RogueLikeVersion`.
   Modelling it also needs `RogueLike_Report_TopStatus`, whose trigger
   carries no event on the node.

5. **The unanswered net IDs.** 144, 146, 159, 217 and 288 are journalled and
   ignored. None of them blocks anything observed so far, which is exactly
   why nobody has had to understand them yet.

6. **The international build.** `com.oversea.spinarena` installs and launches
   but its Cocos resource path still needs work. It is reference-only today.

## Reading the evidence trail

The gateway writes a frame journal to `logic-frames.jsonl` under the save
directory. Each line records direction, net ID, whether the ID is modelled,
and — for node requests — the exports, trigger path and per-top state the
request carried. Because an unacceptable request is answered by closing the
socket, that journal is usually the fastest way to see *why* something was
refused. It was added after a refusal cost twenty minutes of guessing.

`research/runtime/README.md` indexes what each retained run proves. The
captures themselves stay on the preservation host and are not in this
repository.
