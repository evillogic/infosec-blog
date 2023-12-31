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

I then use a github action to build the hugo content. This is the most challenging part because of the submodules. I ended up using a personal access token to allow the action runner to use actions/checkout@v4 with a specific token, which worked much easier than trying to set the PAT in the remote or the credentials within a shell.

This site was very helpful: https://joht.github.io/johtizen/build/2022/01/20/github-actions-push-into-repository.html
