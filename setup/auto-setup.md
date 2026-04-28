# Auto setup

Auto setup hands the job to an AI agent. You point Modeinspect at your repo;
the agent spins up a fresh sandbox, looks around, runs the obvious things,
and writes down what worked. When it finishes, you review the result and
either save it or push back.

Most JavaScript / TypeScript projects with a recognizable package manager
finish auto setup without you needing to type anything.

![Screenshot of the auto setup in progress in the Modeinspect dashboard. Shows the setup panel with a running indicator, a list of steps the agent has discovered so far (e.g. "Install dependencies", "Run database migrations"), each with its own status (running / done / pending), and a streaming log area at the bottom showing the agent's most recent shell output.](./images/auto-setup-in-progress.png)

## What happens, in order

1. **Sandbox provisioning.** We create a clean Linux sandbox and copy your
   codebase into it.
2. **Fast path.** If we can detect a familiar package manager (npm, pnpm,
   yarn, or bun), we run its install command first as a single
   deterministic step. If that's enough, we move straight to step 4.
3. **Agent loop.** If the fast path didn't get us to a working dev server
   — or if you explicitly asked for the agent — the agent takes over. It
   has access to the sandbox shell and can read/write files. It tries
   commands, writes config, starts your dev server, and records a step for
   each thing that worked.
4. **Verification.** Once the agent says "done", we throw away the sandbox
   and replay the recorded config in a fresh one. If everything passes, the
   config is saved and the project is ready.
5. **Review.** You see what the agent produced — the steps, the dev server
   command, the env vars it asked for. You can edit anything before final
   save (or after, in the dashboard editor).

## When the agent pauses

The agent doesn't run forever. It's bounded by:

- **75 iterations** of "try a thing, see what happened"
- **20 minutes** of wall clock time

If it hits either limit, it auto-resumes once. If it hits them again, the
session pauses and you'll see an "Action required" panel with whatever the
agent had figured out so far. From there you can:

![Screenshot of the "Action required" state of the setup panel after the agent paused. The top of the panel shows a red "What went wrong" callout with a multi-line failure summary (e.g. "Dev server exited with code 1: missing DATABASE_URL"). Below it, the editable steps list shows what the agent had built up. Two action buttons sit at the bottom: a primary "Retry setup" button and a subtle "Let the agent take over" button.](./images/auto-setup-paused.png)

- **Edit the config** and verify it manually
- **Hand it back** to the agent with more context
- **Switch to manual setup** entirely

The agent will also stop and ask you for input mid-flight when it has to —
for example, if your dev server needs a specific environment variable that
isn't in your repo, it'll prompt for it rather than guess.

## When verification fails

Verification runs the agent's recorded config in a fresh sandbox, with the
agent no longer in the loop. Sometimes this fails even when the agent
"succeeded" — usually because:

- A step relied on state that the agent built up but didn't record (a
  globally installed CLI, a temp file)
- The dev server needs a port we couldn't probe (firewall, wrong port
  recorded)
- An env var the agent set during exploration wasn't persisted into the
  config

When this happens, the session pauses with a summary of what failed. You
can edit the steps in the editor (most often: add a missing install step,
or fix the dev server port) and click **Retry verification**. You get up to
two retries before we ask you to either invoke the agent again or finish in
manual mode.

## What the agent can and can't do

It **can**:
- Run any shell command that would work in a normal Linux dev environment
- Read and write files in your project
- Start and stop processes (including your dev server)
- Ask you for clarification (env var values, port numbers, etc.)
- Add **file steps** for files it had to create that aren't in your repo

It **can't**:
- Access secrets you haven't given it
- Reach private package registries without credentials
- Install OS-level dependencies that need root (the sandbox is unprivileged)
- Decide for you between two valid choices — it'll either pick the more
  common option or pause and ask

## After auto setup finishes

The output is the same shape as a manual setup config — see [Manual
setup](./manual-setup.md) for the structure, and the per-section pages
([Steps](./steps.md), [Checks](./checks.md), [Files](./files.md),
[Dev server](./dev-server.md), [Env vars](./env-vars.md)) for what each
piece does and how to edit it.

You're not locked in: anything the agent generated, you can change later.
