# Hello World — auto-deploy to Cloudflare Pages on merge to main

A minimal static site. Cloudflare watches this GitHub repo and redeploys automatically
every time something lands on `main`.

```
public/index.html   <- the website
learn-later/        <- a GitHub Actions version of this pipeline, for later
```

---

## Step 1 — Push this repo to GitHub

Create an empty repo on github.com (no README, no .gitignore — you already have them), then:

```bash
git commit -m "Hello world site"
git remote add origin https://github.com/<your-username>/<your-repo>.git
git push -u origin main
```

## Step 2 — Connect Cloudflare to the repo

1. https://dash.cloudflare.com → **Workers & Pages** → **Create** → **Pages** tab → **Connect to Git**
2. Authorize Cloudflare's GitHub App, and give it access to this repo
3. Pick the repo, then configure the build:

| Setting | Value |
|---|---|
| Production branch | `main` |
| Framework preset | None |
| Build command | *(leave empty — plain HTML needs no build)* |
| Build output directory | `public` |

4. **Save and Deploy**

Cloudflare clones the repo, deploys `public/`, and gives you a URL like
`https://<project-name>.pages.dev`.

No API tokens. No secrets. No config files in your repo. That is the whole appeal of this option.

## Step 3 — Watch it auto-deploy

Change the heading in `public/index.html`, then:

```bash
git add .
git commit -m "Change heading"
git push
```

Within seconds, Cloudflare → your project → **Deployments** shows a new build starting,
tagged with your commit message and hash. When it finishes, refresh your `.pages.dev` URL.

**That is continuous deployment.**

## Step 4 — Do it the way real teams do it (via a Pull Request)

Pushing straight to `main` is fine for solo practice, but the workflow you actually want to learn:

```bash
git checkout -b change-the-colour       # new branch
# ...edit public/index.html...
git commit -am "Try a blue background"
git push -u origin change-the-colour
```

Open the PR on GitHub. Two things now happen:

- Cloudflare builds a **preview deployment** at its own URL and posts the link as a PR check —
  you can look at the change live before it goes to production
- `main` is untouched

Click **Merge pull request**. The merge is itself a commit on `main`, so Cloudflare fires
again and deploys to production.

This is the loop: **branch → PR → preview → review → merge → auto-deploy.**

---

## How this works under the hood

Connecting the repo installed a **GitHub App** on your account. GitHub sends Cloudflare a
webhook on every push. Cloudflare's build servers clone the repo, run your build command
(none here), and publish the output directory to its global CDN.

Note where the configuration lives: **in Cloudflare's dashboard**, not in your repository.
That's the main trade-off of this approach — it's fast to set up, but your deployment
settings aren't version-controlled and nobody can review a change to them.

## When you outgrow this

Option A can't run your tests before deploying. The moment you want
"run the test suite, and only deploy if it passes", you need GitHub Actions.

[learn-later/github-actions-deploy.yml](learn-later/github-actions-deploy.yml) is that pipeline,
already written and commented. To activate it later, move it to `.github/workflows/deploy.yml`.

⚠️ **It will not work on this Pages project.** A project created with "Connect to Git" rejects
direct uploads from `wrangler`. Create a *second* Pages project using **Upload assets** when
you're ready to try Actions.

## Troubleshooting

| Problem | Fix |
|---|---|
| Deploy succeeds but page is 404 | Build output directory must be `public`, not blank or `/` |
| Cloudflare doesn't see the repo | GitHub → Settings → Applications → Cloudflare Pages → grant repo access |
| Push doesn't trigger a build | Production branch in Cloudflare must match your branch name (`main`) |
