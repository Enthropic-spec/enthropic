# How to Sync this Wiki to GitHub Wiki

GitHub Wikis are actually full Git repositories themselves. This means you can keep your wiki docs under standard version control inside this `wiki/` directory and sync them to your live GitHub Wiki with a few simple Git commands.

This guide outlines exactly how to do that.

---

## 🚀 Step-by-Step Sync Guide

### 1. Locate Your GitHub Wiki Git URL
Your GitHub Wiki Git URL follows this structure:
`git@github.com:USERNAME/REPOSITORY.wiki.git` (SSH) or `https://github.com/USERNAME/REPOSITORY.wiki.git` (HTTPS).

For the SpeQ repository, it is:
`git@github.com:speq-ai/speq.wiki.git`

### 2. Run the Sync Command
You can easily push your local `wiki/` folder directly to the remote GitHub Wiki branch. Open your terminal in the root of the repository and run:

```bash
# Force-push the contents of your local 'wiki/' folder directly to the remote wiki's main branch
git subtree push --prefix wiki git@github.com:speq-ai/speq.wiki.git master
```

---

## 🔄 Recommended: Automate via GitHub Actions

To ensure that your GitHub Wiki automatically updates every time you merge changes to the `main` branch, you can add a simple workflow.

Create a file named `.github/workflows/wiki-sync.yml` and paste the following configuration:

```yaml
name: Sync Wiki

on:
  push:
    branches:
      - main
    paths:
      - 'wiki/**'

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout main repo
        uses: actions/checkout@v4

      - name: Push to GitHub Wiki
        uses: Andrew-Chen-Wang/github-wiki-action@v4
        env:
          WIKI_DIR: wiki/
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

With this Action in place, any changes you make locally in the `wiki/` folder will sync to the GitHub Wiki tab automatically!
