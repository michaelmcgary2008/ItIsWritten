# Publishing changes to It Is Written

This page is for Tim. It explains how to change the site and get the change live.
You do not need to be a developer. If a step confuses you, stop and ask Michael.

## How the pieces fit

The site's files live in a **Gitea** repository (a shared folder that remembers every
change) at https://repo.watersmedia.org/Waters-Church/ItIsWritten. **Vercel** is the
service that runs the live site at https://it-is-written-waters-church.vercel.app.
**GitHub** is only a relay: Gitea copies every change to it automatically and Vercel
picks it up from there. You never need to open GitHub, and you must never change
anything on it, because Gitea overwrites it.

A "commit" is one saved set of changes with a short note. `main` is the main line of
the project, and whatever is on `main` goes live.

## Two ways to edit

Your Gitea account can commit straight to `main`. Because that goes live, read your
change over before you save it.

### A. In the browser (no software needed)

1. Sign in at https://repo.watersmedia.org and open the ItIsWritten repository.
2. Click the file you want to change (for example `index.html`).
3. Click the pencil icon (Edit File) at the top right of the file.
4. Make your change.
5. Scroll down to **Commit Changes** and type a short message that says what you
   changed, such as "Reword the home page intro".
6. Choose **Commit directly to the main branch**, then click **Commit Changes**.

### B. On a computer with git

Git is the free tool that copies the repository to your computer and sends your changes
back. Copy the repository once:

```sh
git clone https://repo.watersmedia.org/Waters-Church/ItIsWritten.git
```

If you have an SSH key on your Gitea account, you can use this address instead:
`ssh://git@repo.watersmedia.org:3022/Waters-Church/ItIsWritten.git`

Then, each time you make a change:

```sh
cd ItIsWritten
git pull                         # get the latest version first
# ...edit the files...
git status                       # check that only the files you meant to change are listed
git add -A
git commit -m "Short note about what you changed"
git push origin main
```

## What happens after a commit to main

1. Gitea copies the commit to GitHub, within about a minute.
2. Vercel notices it, builds the site and publishes it. Allow 1 to 3 minutes.

So a change usually shows up within a few minutes. To confirm, open
https://it-is-written-waters-church.vercel.app and reload. If you still see the old
version, do a hard refresh (Mac: Cmd+Shift+R, Windows: Ctrl+Shift+R).

Only `main` goes live. A commit to any other branch makes a private test copy that needs
a Vercel login to open; ask Michael if you want one shared with you.

## What is safe to edit

- `index.html` holds all the copy, layout and styling. This is where nearly all changes
  belong.
- `api/claude.js` holds the wording of the prompts (the instructions sent to the AI).
  Change only the text inside the `ACTIONS` section. The prompt text sits between
  backticks and contains pieces like `${topic}` that the code fills in, so keep those
  and the backticks exactly as they are.
- Leave `MODEL`, `LIMITS`, the `SYSTEM` rules, the API key handling and `vercel.json` alone unless Michael
  agrees. They control cost, abuse protection and how the site is deployed.
- Never put an API key or password in any file. The key lives only in Vercel's settings
  and in a private `.env.local` file on a computer, which git ignores.

## Previewing on your own computer (optional)

You need Node.js 20 or newer. In the project folder:

```sh
cp .env.example .env.local       # then ask Michael for a test key and put it in .env.local
npm run dev
```

Open http://localhost:3000. Search needs the key to work. More detail is in `README.md`
under "Run locally".

## If something goes wrong

- **A change broke something.** Undo it on Gitea: open the commit (click its message in
  the Commits list), open the **Operations** menu, choose **Revert**, and commit the
  result to `main`. That publishes the undo through the same steps as above.
- **The change does not appear.** If a build fails, Vercel emails Michael and the live
  site keeps showing the last version that worked. If your change is still missing after
  about 10 minutes, tell Michael.
- **Last resort.** Michael can publish by hand with the Vercel command line tool. The
  steps are in `README.md` under "Deploy to Vercel".
