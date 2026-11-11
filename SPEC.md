# robo_air: project specification

A website that showcases a simulated airline baggage-loading robot. The simulation runs live in the browser. There is no backend.

Repo: [https://github.com/loki52501/robo_air](https://github.com/loki52501/robo_air)

## 1. Stack

- Vite + React + TypeScript
- Three.js through `@react-three/fiber` and `@react-three/drei` for 3D
- Rapier (`@dimforge/rapier3d-compat`) for physics
- Tailwind for styling
- Vitest for unit tests
- Must build to static files so it deploys on Vercel or GitHub Pages

## 2. The simulation

### 2.1 Scene

- A cutaway aircraft cargo hold: a box about 4.0 m long, 1.4 m wide and 1.1 m high.
- One side is transparent so the viewer can see inside.

### 2.2 Bags

- Bags arrive one at a time on a conveyor at the door.
- Each bag is a box with:
  - random size, from a small carry-on up to a large suitcase
  - random weight, 5 to 32 kg
  - a type: hard case, soft duffel, or fragile
- Bags are colour-coded by type.
- A seeded random generator produces the bags, so the same seed always gives the same bag sequence.

### 2.3 Robot behaviour

- Online packing: the robot sees only the current bag, never future ones.
- Placement only: a policy picks a position and a 90-degree orientation.
- The bag animates along a smooth path into place, carried by a simple gantry or arm end-effector.
- Physics then settles the bag.

### 2.4 End of a run

A run ends when either:

- the current bag has no valid placement, or
- 60 bags have been loaded.

## 3. Policies

The user can switch between policies. All policies sit behind one interface, so a reinforcement-learning policy can be added later.

### 3.1 Interface

Input: the current bag and the hold state. Output: a placement, or nothing if no valid placement exists.

Suggested shape (the engine worker may refine the names, but must keep one input and one output):

```ts
interface Policy {
  name: string;
  place(bag: Bag, hold: HoldState): Placement | null;
}

interface Placement {
  position: { x: number; y: number; z: number }; // metres, hold coordinates
  rotation: 0 | 90;                               // degrees about the vertical axis
}

```

### 3.2 First-fit baseline

Takes the first spot that fits, scanning back to front and floor up.

### 3.3 Smart heuristic

Scores candidate spots using a heightmap.

Rewards:

- low placement
- contact with walls and neighbouring bags
- flat support under the bag

Penalties:

- a heavy bag on top of a fragile bag
- centre-of-gravity drift

## 4. Metrics

A live panel that updates after every bag:


| Metric                   | Definition                                                      |
| ------------------------ | --------------------------------------------------------------- |
| Fill percentage          | total bag volume / hold volume                                  |
| Bags loaded              | count                                                           |
| Toppled or shifted bags  | bags that moved more than 10 cm after settling                  |
| Centre-of-gravity offset | distance from the hold centre, in cm, with a small 2D indicator |
| Rule violations          | heavy bag resting on a fragile bag                              |


## 5. Page layout

1. **Hero:** project title, a one-line description, and the live simulation auto-playing straight away.
2. **Controls:** play/pause, reset, speed (1x, 4x, 16x), seed input, policy picker, orbit camera.
3. **Compare mode:** two holds side by side on the same seed, baseline against smart heuristic, each with its own metric panel.
4. **Results:** a "Run benchmark" button that runs 20 seeds headless (no rendering) for each policy, then shows a bar chart of average fill percentage and toppled bags.
5. **How it works:** a short section covering the problem, the constraints, and the two policies.
6. **Footer:** a GitHub link placeholder.

## 6. Design

- Dark, clean, technical look. The simulation is the centrepiece.
- Responsive: works on a laptop. On mobile, the panels stack under the canvas.

## 7. Quality bar

- Simulation logic lives in plain TypeScript modules, separate from React and rendering, so the benchmark can run it headless.
- Unit tests (Vitest) for the bag generator, the heightmap, and both policies.
- Holds 60 fps with 60 bags on a normal laptop.
- A README with run, build and deploy steps.

## 8. Folder layout and ownership

This layout is an addition to the original brief. It exists so each agent owns a separate set of files.


| Path       | Contents                                                                                                    | Owner        |
| ---------- | ----------------------------------------------------------------------------------------------------------- | ------------ |
| `src/sim/` | bag generator, seeded RNG, heightmap, hold state, policies, metrics, headless runner                        | engine       |
| `src/ui/`  | React components, Three.js scene, controls, panels, charts, page sections                                   | frontend     |
| `tests/`   | Vitest tests                                                                                                | qa           |
| repo root  | [`SPEC.md`](http://SPEC.md), [`CLAUDE.md`](http://CLAUDE.md), [`README.md`](http://README.md), config files | orchestrator |


Rules:

- Nothing under `src/sim/` imports React, Three.js or anything from `src/ui/`.
- `src/ui/` uses the engine only through what `src/sim/index.ts` exports.

## 9. Stages

Work in stages. Verify each one before moving on: run the dev server and check it in the browser.

The checks listed under each stage are an addition to the original brief, to give QA something concrete to test.

### Stage 1: hold, bag generator, physics

- The cutaway hold renders with one transparent side.
- The same seed produces the same bag sequence twice in a row (unit test).
- Bag sizes, weights (5 to 32 kg) and types stay within range over 1,000 generated bags (unit test).
- A dropped bag settles on the floor and stays inside the hold.

### Stage 2: baseline policy and metrics

- First-fit places bags back to front, floor up, and never overlaps an existing bag (unit test).
- A run ends at 60 bags or when no placement is valid.
- All five metrics update after every bag.
- The heightmap matches a hand-built example (unit test).

### Stage 3: smart heuristic

- The heuristic implements the same `Policy` interface as the baseline.
- It never returns a placement that overlaps or leaves the hold (unit test).
- It never places a heavy bag on a fragile one when another valid spot exists (unit test).
- The policy picker switches between the two policies.

### Stage 4: compare mode and benchmark

- Compare mode shows both holds on the same seed with both metric panels.
- "Run benchmark" completes 20 seeds per policy with no rendering and shows the bar chart.
- The benchmark gives the same numbers when run twice.

### Stage 5: page polish

- All six page sections are present.
- The layout works on a laptop and stacks correctly on a phone-width screen.
- 60 bags loaded holds 60 fps on a normal laptop.
- `pnpm build` produces static files, and the README covers run, build and deploy.

## 10. Out of scope for now

- Any backend or server-side code
- A reinforcement-learning policy (the interface must allow one later)
- Full robot arm kinematics: the gantry or arm is a visual carrier only