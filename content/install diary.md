---
dg-publish: true
---
Follow Nicole's tutorial at

https://notes.nicolevanderhoeven.com/How+to+publish+Obsidian+notes+with+Quartz+on+GitHub+Pages

- [x] install quartz

-used https://nodejs.org/en and then https://nodejs.org/en/download

## local web

doing 
```
npx quartz build --serve
```
We can browse the webpage locally at localhost:8080

## move up to Github

- create repository gitsy un github *without* a readme file

https://github.com/aljrf/gitsy.git

Then, at terminal, type 

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

We can see the repository  at  https://github.com/aljrf/gitsy.git
## prettify obsidian

Every page needs a draft: false or true. It is assumed false.

Every page with the property draft:false is not shown on the web. We will install a templater to make the note taking easier and inserting always the same frontmatter.

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


## prepare for deployment 

According 

https://quartz.jzhao.xyz/hosting#github-pages

- create a deploy file
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
## Create a GitHub Action 

In GitHub, go to Settings > Pages.

Under _Source_, select _GitHub Actions_.

Then, go back to your terminal and sync. Remember, that's:

```
npx quartz sync
```

It's live at 
https://aljrf.github.io/gitsy/

![[Pasted image 20250615203551.png]]
## Themes:

Add the following linte to your `deploy.yml` before the `permissions` section:

```yaml
env:
  THEME_NAME: <THEME-NAME>
```

And add the following lines to your `deploy.yml` before the `build` step:

```yaml
- name: Fetch Quartz Theme
  run: curl -s -S https://raw.githubusercontent.com/saberzero1/quartz-themes/master/action.sh | bash -s -- $THEME_NAME
```

Important

Replace `<THEME-NAME>` with your desired theme name. See [Compatibility List](https://github.com/saberzero1/quartz-themes?tab=readme-ov-file#supported-themes)

Tip

Example for Tokyo Night:

```yaml
env:
  THEME_NAME: tokyo-night
```
https://github.com/saberzero1/quartz-themes?tab=readme-ov-file#supported-themes

This is how it is deploy.yml

Deply.yml is at gitsy.github/workflows/deploy.yml

```
name: Deploy Quartz site to GitHub Pages
 
on:
  push:
    branches:
      - v4
env:
  THEME_NAME: hackthebox
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
      - name: Fetch Quartz Theme
        run: curl -s -S https://raw.githubusercontent.com/saberzero1/quartz-themes/master/action.sh | bash -s -- $THEME_NAME
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



just change THEME_NAME for something you can pick from  [the list of themes](https://github.com/saberzero1/quartz-themes?tab=readme-ov-file#supported-themes) instead of *hackthebox*. 





