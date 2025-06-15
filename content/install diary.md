Follow Nicole's tutorial at

https://notes.nicolevanderhoeven.com/How+to+publish+Obsidian+notes+with+Quartz+on+GitHub+Pages

- [x] install quartz

-used https://nodejs.org/en and then https://nodejs.org/en/download



npx quartz build --serve
browse at localhost:8080

create repository gitsy un github *without* a readme file

https://github.com/aljrf/gitsy.git

type 

```
git remote -v
```

output is

```
origin	https://github.com/jackyzha0/quartz.git (fetch)
origin	https://github.com/jackyzha0/quartz.git (push)
upstream	https://github.com/jackyzha0/quartz.git (fetch)
upstream	https://github.com/jackyzha0/quartz.git (push)
```
- and replace the *origin* part with the repository:

```
git remote rm origin

```
```
git remote add origin https://github.com/aljrf/gitsy.git 
```

then the structure looks like 

```
aljrf@newdelly:~/gitsy$ git remote -v
origin	https://github.com/aljrf/gitsy.git (fetch)
origin	https://github.com/aljrf/gitsy.git (push)
upstream	https://github.com/jackyzha0/quartz.git (fetch)
upstream	https://github.com/jackyzha0/quartz.git (push)

```

- then push everything in one go:
```
npx quartz sync --no-pull
```

Backing up your content
[v4 7bee655] Quartz sync: Jun 15, 2025, 5:31 PM
 11 files changed, 1543 insertions(+), 987 deletions(-)
 delete mode 100644 content/.gitkeep
 create mode 100644 content/First note.md
 create mode 100644 content/How to actually publish.md
 create mode 100644 content/Publish alternatives.md
 create mode 100644 content/Welcome.md
 create mode 100644 content/index.md
 create mode 100644 content/install diary.md
 create mode 100644 content/invisible page.md
 create mode 100644 content/netlify.md
Pushing your changes
Enumerating objects: 11509, done.
Counting objects: 100% (11509/11509), done.
Delta compression using up to 4 threads
Compressing objects: 100% (4120/4120), done.
Writing objects: 100% (11509/11509), 10.86 MiB | 11.05 MiB/s, done.
Total 11509 (delta 7320), reused 11451 (delta 7274), pack-reused 0 (from 0)
remote: Resolving deltas: 100% (7320/7320), done.
To https://github.com/aljrf/gitsy.git
 * [new branch]      v4 -> v4
branch 'v4' set up to track 'origin/v4'.


branch is v4, out of the blue?


- install templater plugin

with following template

```
---
title: "How to publish Obsidian notes with Quartz on GitHub Pages"
draft: false
tags:
  - 
---
```


Next thing: prepare for deplyment according 

https://quartz.jzhao.xyz/hosting#github-pages

touch .github/workflows/deploy.yml

and fill it up:
```
name: Deploy Quartz site to GitHub Pages
 
on:
  push:
    branches:
      - v4
 
permissions:
  contents: read
  pages: write
  id-token: write
 
concurrency:
  group: "pages"
  cancel-in-progress: false
 
jobs:
  build:
    runs-on: ubuntu-22.04
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0 # Fetch all history for git info
      - uses: actions/setup-node@v4
        with:
          node-version: 22
      - name: Install Dependencies
        run: npm ci
      - name: Build Quartz
        run: npx quartz build
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: public
 
  deploy:
    needs: build
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```
Then:
### Create a GitHub Action 

In GitHub, go to Settings > Pages.

Under _Source_, select _GitHub Actions_.

Then, go back to your terminal and sync. Remember, that's:

```
npx quartz sync
```

It's live at 
https://aljrf.github.io/gitsy/



Change 


    pageTitle: "Gitsy",

To change site title