---
title: "Edit the Blog"
date: 2022-09-04T16:18:56-07:00
draft: false
---

clone the blog with

```shell
git clone --recurse-submodules git@github.com:evillogic/infosec-blog.git
```

create a new page with

```shell
hugo new <page-name>.md
```

Building the pages is simply `hugo`, however due to submodule behavior we need to first checkout the master submodule branch. You can then simply navigate to the public directory and push the change.

```shell
pushd public
git checkout master
popd
hugo
pushd public
git add .
git commit -m "added some content"
git push
popd
git submodule update --remote --merge
git add .
git commit -m "update the submodule pointer and add source content"
git push
```

Or here is a handy alias which can be invoked with an argument commit message

```shell
alias update-blog='f(){ pushd public; git checkout master; popd; hugo; pushd public; git add .; git commit -m "$@"; git push; popd; git submodule update --remore --merge; git add .; git commit -m "$@"; git push; unset -f f; }; f'
```
