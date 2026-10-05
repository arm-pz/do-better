# do better

"That's a bad answer. Do better."

Now the model actually has to. `do-better` spins up N sub-agents (default 100, `--quick` for 16),
hands every one of them the exact same task word for word, and gives each a different strategy
card: a reasoning mode, a workflow, and a strategy, so all N attack the problem differently. Then
the bracket starts. Each pair of solutions attacks the other, each side defends and revises, and a
judge scores the match on a written rubric. Losers are out. 100 becomes 50, 25, 13, 7, 4, 2, 1.

What comes back: the champion solution, the attacks it survived, its card, and the round count.
If you started from an answer you rejected, a final blind judge compares the champion with it, and
reports the truth even when your old answer scores higher.

**It never touches your project.** Every sub-agent writes inside `.dobetter/`. Applying the
winning answer is your call, and it asks first.

## Install (Qoder CLI)

```bash
git clone https://github.com/arm-pz/do-better.git
cp -r do-better/skills/do-better ~/.qoder/skills/
```

Needs Python 3.8+ on PATH and a CLI that can spawn sub-agents. Nothing to pip install.

## Use

```
/do-better
/do-better --quick write the headline for our pricing page
/do-better --agents 32 fix the flaky test in tests/test_api.py
/do-better --seed 7 plan my launch week, I have 6 hours a day
```

Bare `/do-better` takes your last request as the task and your unsatisfying answer as the
baseline to beat. That is also the point of the name: the skill triggers on what you actually
say when you are unhappy ("try again", "do better", "that's wrong", "make them compete"), and
when it starts itself like that, it asks before spending anything.

| flag | what it does |
| --- | --- |
| `--agents N` | N competitors. Default 100. |
| `--quick` | 16 competitors. The everyday setting. |
| `--seed S` | Same seed, same cards and bracket. Default random, and recorded. |
| `--wave W` | Sub-agents per wave. Default 10, matching the CLI's concurrency. |

## The cost, honestly

The tokens are yours, and 100 agents is a lot of them.

| agents | rounds | sub-agent calls | waves of 10 |
| --- | --- | --- | --- |
| 100 | 7 | 595 | 70 |
| 64 | 6 | 379 | 49 |
| 32 | 5 | 187 | 28 |
| 16 (`--quick`) | 4 | 91 | 16 |
| 8 | 3 | 43 | 10 |

`python skills/do-better/bracket.py plan --agents N` prints the numbers before you commit. Use
`--quick` for everyday tasks and save the full run for the answer that matters.

## How it works

1. **Spawn.** Each competitor gets the identical task text (byte for byte, a test checks it) plus
   one of 2,160 strategy cards dealt without repeats from 15 reasoning modes, 12 workflows and 12
   strategies.
2. **Attack.** Paired solutions attack each other: up to 7 specific, checkable flaws each,
   labelled FATAL / MAJOR / MINOR. Pairings avoid matching two agents with the same reasoning mode.
3. **Defend.** Each side concedes or rebuts every attack with evidence, then resubmits a revised
   solution.
4. **Judge.** A separate judge checks every claim itself and scores both on
   [rubric.md](skills/do-better/rubric.md): correctness 30, completeness 25, robustness 20,
   specificity 15, clarity 10. Verified fatal cannot beat non-fatal. The orchestrator never picks
   a winner; the arithmetic is done by `bracket.py`.
5. **Repeat** until one survives (odd rounds give a bye, never twice to the same agent while
   anyone else waits). Then report.

The whole tournament lives in one JSON file managed by `bracket.py` (standard-library Python).
Sub-agents reply with one receipt line and keep their work on disk, so the orchestrator's context
stays small, and a mid-run context compaction loses nothing: `next` resumes from the file.

## Caveats

- N copies of one model, differentiated by cards, not N different models.
- The champion is the best-scoring answer this tournament found, not a proof of correctness.
- Sub-agents can't see your chat: everything they know comes from the generated task file, so
  unstated requirements are missed N ways.
- Expect permission prompts unless edits are auto-accepted: hundreds of files get written to
  `.dobetter/`.
- A vague task in, 100 flavours of vague out.

## Files

```
skills/do-better/SKILL.md          orchestrator steps + the exact brief every sub-agent gets
skills/do-better/bracket.py        tournament state machine, standard library only
skills/do-better/strategies.json   15 reasoning modes x 12 workflows x 12 strategies. Edit freely
skills/do-better/rubric.md         the five criteria judges score on
tests/test_bracket.py              27 tests, incl. a full simulated 100-agent tournament
```

Run the tests: `python -m unittest discover -s tests`

## Credit

Renamed and adapted for the Qoder CLI from **Arena** by Jake Schincariol
([Jakeschincariol/arena-skill](https://github.com/Jakeschincariol/arena-skill), MIT). Same
tournament, new name, Qoder-native plumbing.

## License

MIT, same as upstream. See [LICENSE](LICENSE).
