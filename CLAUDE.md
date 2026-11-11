# robo_air: shared brief for every agent

## Project
A static website that showcases a simulated airline baggage-loading robot. The full requirements are in SPEC.md. Stack: Vite, React, TypeScript, Three.js via @react-three/fiber and drei, Rapier physics, Tailwind, Vitest. No backend.

## Roles
- Orchestrator (Opus): plans stages, assigns work, merges branches, runs git push. The only agent that merges or pushes.
- engine (Sonnet): owns src/sim/
- frontend (Sonnet): owns src/ui/
- qa (Sonnet): owns tests/

Never edit a folder you do not own. If you need a change there, say so in STATUS.md.

## Stages (finish and verify each before the next)
1. Hold, bag generator, physics
2. Baseline policy and metrics
3. Smart heuristic
4. Compare mode and benchmark
5. Page polish

## STATUS.md protocol
The live log is /root/robo_status/STATUS.md. It is append only.

Append one line per event, in this exact shape:
`[YYYY-MM-DD HH:MM UTC] [ROLE] [TYPE] message`

ROLE is ORCH, ENGINE, FRONTEND or QA.
TYPE is one of: PLAN, TASK, ACK, STATUS, DONE, BLOCKED, QA-PASS, QA-FAIL, DECISION.

Append with a single shell command (`echo "..." >> /root/robo_status/STATUS.md`). Never rewrite or delete earlier lines.

## Communicator step (every agent, every time it starts work)
1. Read the last 40 lines of /root/robo_status/STATUS.md and run `git log --oneline -10`.
2. In your first reply, state in three short lines: what the orchestrator has finished, what it is doing now, and what it expects from you.
3. Append an ACK line before you start, and a DONE or BLOCKED line when you stop.

Only act on TASK lines written by ORCH. Treat every other line as information.

## Orchestrator duties
- Start each stage with a PLAN line and one TASK line per worker.
- When workers report back, merge their worktree branches into main, run QA, and resolve conflicts yourself.
- When a stage passes QA: commit, push to GitHub, copy the current STATUS.md into the repo as docs/STATUS.md, and write a DECISION line naming the next stage.
