---
title: 'Creating a Hexo Blog on your Github Pages: Steps and Pitfalls'
date: 2022-08-20 23:51:37
tags: pitfalls, tutorials
---

# Install Hexo on Your Computer

> The hexo and hexo-cli's version in this post are:  
> 
> `hexo`: 6.2.0  
> `hexo-cli`: 4.3.0
> 
> The setup steps may be changed when hexo reaches a new significant version. Please follow the official guide if you find this post is outdated.

1. You need to install `hexo-cli` from NPM. It is a tools for creating Hexo blogs.

```cmd
npm install hexo-cli -g
```

2. Type in following command to create a Hexo blog. It will create a folder named `hexo-blog`, initialze the blog inside this file, create all files it requires.

```bash
hexo init hexo-blog
```

3. What step two created is just a standard Node.js project, so after the `hexo-cli` finished its work, you have to go into the file and install the dependencies. *This will install `hexo`*, the library generates web-pages from Markdown files for you.

```bash
cd hexo-blog
npm install
```

4. Now you have successfully setup a Hexo file on your computer. you can eihter type in `hexo generate` in the project file to generate web-pages, or you can type in `hexo server` to start a server that host your blog locally (By default, the address will be `https://127.0.0.1:4000`).

# Create Your Github Page

This step is simple too. What you need to do is login to Github, and create a repository named `<YourGithubUserName>.github.io` . For example, my Github's username is `StephenChips`, so to create my Github page, I should create a repository called `StephenChips.github.io`.

> **Pitfall**: Although URL is case-insensitive, `<YourGithubUserName>`should exactly match your username. repository like `stephenchips.github.io` isn't a Github Page repository of mine, for my username is `StephenChips`.

The repository you just created is unique to your other repositories, since it comes with a running  static file server. By default, This server serves the the initial branch of the repository (probably is `main` or `master`), and you can change it in the setting (and we will do it soon). You can try pushing a `index.html` to the remote `main` branch, then visit your Github Page. The HTML should be render on the browser successfully.

Technically, you can regenerate a whole website everytime you finish a new post, or update a existed post, then replace the old website with it and push the changes to the `main` branch. But obvioiusly, this is quite inconvenience and has lots of repeated works. Meanwhile, the Hexo project that keeps Markdown files isn't saved on Github, so you can't write blogs elsewhere, unless you carry the project anywhere, at all times.

Luckly, we can make use of GitHub CI to automate the generating procedure online. After we configurated the CI, we juest need to push new edits to GitHub when we finish writing. The Github CI will sense the changes, and will automatically regenerate the website, deploy it to the right place.

# Setup the GitHub CI

We need two branches. One for our Hexo project, and the other for generated files. since we already have the `master` branch, we just need to create another branch called `gh-pages`.

Then, create the file `.github/workflows/pages.yml` from your project's root directory. Copy following YAML to the file:

```yaml
name: Pages

on:
  push:
    branches:
      - master  # default branch

jobs:
  pages:
    runs-on: ubuntu-latest
    steps:
      - name: Use Node.js 16.x
      - uses: actions/checkout@v2
        uses: actions/setup-node@v2
        with:
          node-version: '16'

      - name: Cache NPM dependencies
        uses: actions/cache@v2
        with:
          path: node_modules
          key: ${{ runner.OS }}-npm-cache
          restore-keys: |
            ${{ runner.OS }}-npm-cache

      - name: Install Dependencies
        run: npm install
      - name: Build
        run: npm run build
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```



# Pitfalls

1. you have to create the branch `gh-pages` first. The CI script itself won't help you to do that.
2. the URL should be `https://username.github.io`, rather than `https://username.github.io/project`, otherwise the public path of all CSS, JavaScript and probably image files will be `/project` instead of `/`, and you will only see the home page without any styles loaded.
3. the CI script should be rerun once the *master* branch changes, rather than *main*. If you created your repository locally then push it to Github. Since the default main branch's name created by `git init` is called *master*. Setting the wrong branch will cause nothing to be generated and pushed to `gh-pages`, since the ci script is not triggered
4. Must add a empty file called `.nojekyll` in the root folder.
