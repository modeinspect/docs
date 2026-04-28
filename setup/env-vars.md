# Environment variables

**Environment variables** are key/value pairs we inject into every
[setup step](./steps.md) and into the [dev server](./dev-server.md). This
is where things like API base URLs, database connection strings, and
feature flags belong.

If your local project reads from `process.env.API_URL`, it'll find it
here.

## How it works

You add entries in the editor as `KEY` / `VALUE` pairs. On every
sandbox replay, we:

1. Take the sandbox's default environment
2. Layer your env vars on top (your values **override** sandbox defaults
   if the key collides)
3. Pass the merged env to every shell step
4. Pass the same merged env to the dev server when we start it

There's no scoping or per-step env — everything is global to the run.

![Screenshot of the "Environment variables" section in the setup editor. Header reads "Environment variables" in uppercase. Below it, three rows of two monospace inputs side by side — "API_KEY" / "sk_test_…", "DATABASE_URL" / "postgres://localhost:5432/dev", "NEXT_PUBLIC_API_URL" / "https://api.example.com" — each row with a small X button to remove it. Below the rows, a subtle "Add variable" button with a plus icon.](./images/env-vars-editor.png)

## What to put here

✅ **Yes:**

- Public-ish config: API base URLs, feature flags, region names
- Build-time configuration your tooling reads from `process.env`
- Connection strings for development databases
- Anything you'd put in a `.env.development` file

❌ **No:**

- Production credentials
- Anything you'd be unhappy seeing in a screenshot of the editor
- Long blobs (certificates, private keys) — use a [file step](./files.md)
  instead

## ⚠️ The secrets caveat

**Env vars are stored in plaintext in our database.** They are not
encrypted at rest with a per-project key, they aren't masked in our logs,
and anyone on your team with project access can read them in the editor.

The same is true of [file step](./files.md) contents.

The threat model to assume: anything you put here has roughly the same
exposure as committing it to a private repo. If that's not okay for a
given value, don't put it here.

In practice, that means:

- Use **scoped / restricted** keys — a Stripe test key, not your live
  account key
- Use **short-lived** keys where you can — rotate them on a schedule you
  set elsewhere
- Use **per-environment** values — never paste a production env into a
  setup config
- For values where the above isn't possible, talk to your security people
  before adding them

## Working with env vars in your steps

Inside a setup step, env vars are just regular shell variables, so:

```sh
curl -H "Authorization: Bearer $API_TOKEN" https://api.example.com/seed | psql $DATABASE_URL
```

works as expected. Same in the dev server command:

```sh
PORT=$PORT pnpm dev
```

(Though you usually don't need to forward — the env is already in scope.)

## Working with env vars in your file steps

[File step](./files.md) contents are static — we do **not** template them
with your env vars. If your `.env.local` needs to reference a value, write
it literally into the file before uploading.

If you want a single source of truth for a value that's used both in a
file and in the dev server, define it as an env var and have the file
reference it indirectly (e.g. `API_URL=$NEXT_PUBLIC_API_URL` in a wrapper
script), or just accept the duplication.

## Editing env vars

Add, change, or remove entries in the editor and click **Verify** to
replay setup with the new values. You don't need to re-run the agent —
verification is enough. Once verification passes, **Save** writes the new
env into the canonical config.

Empty keys are dropped on save (so you can leave a half-typed row in the
editor without it breaking anything). Empty *values* are kept — that's
how you set a flag to the empty string.
