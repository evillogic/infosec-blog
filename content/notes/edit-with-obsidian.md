---
title: Edit the Blog with Obsidian
date: 2023-12-30
draft: false
---
Maybe I will use this more if I can edit it with Obsidian. Let's see!

In order to edit with Obsidian, I took some inspiration from this:

https://github.com/vinibaggio/obsblog-template

Where the basic idea is just to use Obsidian Git to push changes to the repository. No idea how well this works on mobile, and it seems to have the drawback that Obsidian still can't edit .md files by default.

Obsidian Git's documentation is useless. Set the custom base path to infosec-blog and make sure your ssh is configured correctly, then reload Obsidian. You can just close it an reopen it.

I then use a github action to build the hugo content. This is the most challenging part because of the submodules. I ended up using a personal access token to allow the action runner to use `actions/checkout@v4` with a specific token, which worked much easier than trying to set the PAT in the remote or the credentials within a shell.

This site was very helpful: https://joht.github.io/johtizen/build/2022/01/20/github-actions-push-into-repository.html

### Mobile & other devices

The other drawback of this is that LiveSync doesn't seem to support syncing hidden folders, even when syncing hidden files is enabled and the `.git` folder is not skipped. This seems like it should be supported.

Without the .git folder, Obsidian Git won't be able to recognize the repository. Functionally this means you have to clone the git repo from every machine you want to publish from. I'll probably try and fix this at some point.

### Github action

The github action is fairly simple, I forked the code from the default hugo builder and added a repository secret that is a classic personal access token (PAT). Most of the script is dedicated to installing dependencies, the important parts is the call to `actions/checkout@v4` which uses the PAT to set up the pages repository as the PAT user.

Once the repositories are cloned properly, this follows the same workflow as [regular edit procedure](/notes/edit-the-blog/). That is to say, that we build the hugo data, then push the changes to the submodule and update the reference on the main repository.

```yaml
# Sample workflow for building and deploying a Hugo site to GitHub Pages
name: Deploy Hugo site to Pages

on:
  # Runs on pushes targeting the default branch
  push:
    branches: ["master"]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# Sets permissions of the GITHUB_TOKEN to allow deployment to GitHub Pages
permissions:
  contents: write

# Allow only one concurrent deployment, skipping runs queued between the run in-progress and latest queued.
# However, do NOT cancel in-progress runs as we want to allow these production deployments to complete.
concurrency:
  group: "pages"
  cancel-in-progress: false

# Default to bash
defaults:
  run:
    shell: bash

jobs:
  # Build job
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.120.4
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb
      - name: Install Dart Sass
        run: sudo snap install dart-sass
      - name: Checkout
        uses: actions/checkout@v4
      - name: Checkout theme
        uses: actions/checkout@v4
        with:
          repository: evillogic/wooper-risotto
          path: themes/risotto
      - name: Checkout pages
        uses: actions/checkout@v4
        with:
          repository: evillogic/evillogic.github.io
          path: public
          token: ${{ secrets.BLOG_PAT }}
      - name: Install Node.js dependencies
        run: "[[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci || true"
      - name: Build with Hugo
        env:
          # For maximum backward compatibility with Hugo modules
          HUGO_ENVIRONMENT: production
          HUGO_ENV: production
        run: |
          hugo \
            --minify
      - name: Blog submodule commit and push
        working-directory: ./public
        env: 
          CI_COMMIT_MESSAGE: Continuous Integration Build Artifacts
          CI_COMMIT_AUTHOR: Continuous Integration
        run: |
          git config --global user.name "${{ env.CI_COMMIT_AUTHOR }}"
          git config --global user.email "evillogic1@gmail.com"
          git add -A
          git commit -a -m "${{ env.CI_COMMIT_MESSAGE }}"
          git push origin HEAD:master
      - name: Update references
        env: 
          CI_COMMIT_MESSAGE: Continuous Integration Build Artifacts
          CI_COMMIT_AUTHOR: Continuous Integration
        run: |
          git config --global user.name "${{ env.CI_COMMIT_AUTHOR }}"
          git config --global user.email "evillogic1@gmail.com"
          git submodule update --remote --merge
          git add -A
          git commit -a -m "${{ env.CI_COMMIT_MESSAGE }}"
          git push origin HEAD:master

```