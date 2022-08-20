---
title: 'Creating a Hexo Blog on your Github Pages: Steps and Pitfalls'
date: 2022-08-20 23:51:37
tags: pitfalls, tutorials
---

# Pitfalls

1. The Hexo's official guide is incorrect (maybe it is obsolete), following that guide won't let you create your hexo blog successfully.
2. you have to create the branch `gh-pages` first. The CI script itself won't help you to do that.
3. the URL should be `https://username.github.io`, rather than `https://username.github.io/project`, otherwise the public path of all CSS, JavaScript and probably image files will be `/project` instead of `/`, and you will only see the home page without any styles loaded.
4.  the CI script should be rerun once the *master* branch changes, rather than *main*. If you created your repository locally then push it to Github. Since the default main branch's name created by `git init` is called *master*. Setting the wrong branch will cause nothing to be generated and pushed to `gh-pages`, since the ci script is not triggered.
