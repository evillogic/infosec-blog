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

Building the pages is simply `hugo`. You can then simply navigate to the public directory and push the change.

```shell
pushd public
git add .
git commit -m "added some content"
git push
popd
```

Or here is a handy alias

```shell
alias update-blog='f(){ pushd public; git add .; git commit -m "$@"; git push; popd; unset -f f; }; f'
```
