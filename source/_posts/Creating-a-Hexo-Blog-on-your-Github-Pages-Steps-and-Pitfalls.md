---
title: 'Creating a Hexo Blog on your Github Pages: Steps and Pitfalls'
tags: 
  - Pitfalls
  - Tutorials
date: 2022-08-20 23:51:37
---


# Setup Hexo on Your Computer

Just type in following command to install Hexo:

```bash
# Install the command line tool
npm install hexo-cli -g

# Use the command line tool to create a new blog inside a folder (created automatically).
hexo init hexo-blog

# The blog is just a Node.js project, so you have to install packages before using.
cd hexo-blog
npm install
```

When installatiom is completed, you can type `hexo generate` to start a local server, then you can visit `https://127.0.0.1:4000` to preview your blog locally.

## Configuration

*Hexo*'s configuration file is `_config.yml`, which is placed at the project's root directory. It specified information and settings of the blog, e.g. `title` for the name of your blog, which is displayed at the home page. You can look up there definition on this [webpage](https://hexo.io/docs/configuration.html). Through many of them are straightforward, one option is worth to mention: `url`.

This defines the public root of your blog.

1. Fill it with Github Page's address, e.g. `https://stephenchips.github.io`, if the blog is the only application hosted on your Github Page.
2. If you want to host your blog under a public path, fill it with Github Page's address, along with the public path. e.g. `https://stephenchips.github.io/blog`.

The reason why Hexo needs it is because Hexo doesn't know that which public path will the website be deployed at when generating the website, and it needs this information to correctly transform links of resources referenced by Markdown posts from its local address to the public address (e.g. Images, JavaScript, CSS). If the `URL` isn't correct, resources won't be loaded successfully, `404 Not Found` errors will be thrown.

# Create Your Github Page Repository

The steps is same as creating a normal repository, except the name is fixed:

1. `git init` to create a new repository on your computer.
2. Create a EMPTY repository named `<YourGithubUsername>.github.io`, DO NOT do any initialization. After created, you should see following webpage:
   ![](empty-repo.png)
3. Add remote source and push the local `master` branch to the remote origin.
4. *DONE!*

For example, my Github's username is `StephenChips`. To create my Github page, I just need to create a repository called `stephenchips.github.io` (all lowercase).

Try pushing an `index.html` to the remote `master` branch, then visit your Github Page. The HTML should be render on the browser successfully.

**AGAIN**, please **DO** create an **empty repository**. It isn't that you can't setup a Hexo blog on Github Page if you don't do that, just the main branch's name will be `main` instead of `master` in that case. To make the texts brief, The main branch will always be `master` in this article. If your main branch's name isn't `master`, you will have to change some configuration options (you will see), which are quite unremarkable and easy to omit, and may lead to wired error messages.

## Explanations

1. This repository is unique to your other repositories, since it comes with a running static file server.
2. By default, This server serves files from repository's `master` branch

# Setup the GitHub CI

The website Hexo generates has a fixed content, that's why it's called "static site generator", so after you've modified or created a post, you have to generate and deploy the site again. It's quite painful to do it manually, so we will use Github CI to help us automate this process. It will generate and deploy the website for you when we push a new change to the `master` branch.

## Steps

1. Create a branch called `gh-pages`, you can create remote branch first or create the local branch first, it's up to you. Personally I recommand you to create it locally, then push it to the remote origin. Because you can create complete empty branch with following command.
   
   ```bash
   git switch --orphan <new branch>
   git commit --allow-empty -m "complete empty branch"
   git push -u origin <new branch>
   ```

2. Go to `https://github.com/<username>/<username>.github.io/settings`, look at the sidebar, select **Page**, scroll to **Build and deployment**, let the source to be **Deploy from a branch**, let the branch to be **gh-pages**, and let the root directory to be **/ (root)**.
    ![Just like this](set-branch.PNG)

3. Create `.github/workflows/pages.yml` in your project's root directory, copy following YAML into the file and push it to the remote repository. 
   
   ```yaml
   name: Pages
   
   # Tell Github CI to generate and deplopy the website
   # when we push a new commit to the master branch
   on:
     push:
       branches:
         - master # the main branch's name, change it if isn't called "master"
   
   # The jobs Github CI needs to do to generate and deploy the website
   jobs:
     pages:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v2
   
         # Job No.1
         - name: Use Node.js 16.x
           uses: actions/setup-node@v2
           with:
             node-version: '16'
   
         # Job No.2
         - name: Cache NPM dependencies
           uses: actions/cache@v2
           with:
             path: node_modules
             key: ${{ runner.OS }}-npm-cache
             restore-keys: |
               ${{ runner.OS }}-npm-cache
   
         # Above two jobs set up a proper Node.js environment which
         # Hexo requires.
         #
         # You can imagine Github CI creates a virtual machine or a
         # sandbox with nothing but Git before running the jobs. To
         # run Node.js we should set up the environment first.
   
         # Job No.3
         # Install the dependenies
         - name: Install Dependencies
           run: npm install
   
         # Job No.4
         # Generate website files and output them to the `./public` directory.
         # The `./public` directory is `.gitignored` in the `master` but isn't in the `gh-pages`
         - name: Build
           run: npm run build
   
         # Job No.5
         # Checkout the `gh-pages`,
         # move all files and folders inside publish_dir (./public) to destination_dir (./),
         # then commit the changes.
         - name: Deploy
           uses: peaceiris/actions-gh-pages@v3
           with:
             github_token: ${{ secrets.GITHUB_TOKEN }}
             publish_dir: ./public # The directory where hexo output the website files
             destination_dir: ./
   ```
   
   1. Add an empty file `.nojekyll` to root folder, the one contains `package.json`.

## Explanation

1. The `.github/workflows/pages.yml` is a CI configuration file that tells Github CI how and when to generate and deploy the website.
2. By default Github Pages uses another static site generator called *jekyll*. `.nojekyll` tells Github Page that we do not use *jekyll*, and let it bypass relative process. Otherwise the it will fail to generate website.

## Pitfalls

1. You should create the branch `gh-pages` yourself. Github CI will not create it for you if it found the branch is absent.
2. if you **INSIST ON** creating a **NON-EMPTY** repository on the GitHub (e.g. creates with a `.gitignore`), The main branch's name will be `main` other than `master`. In this case, the GitHub CI should listen `main` branch's changes, instead of `master` branch's. Therefore the YAML's `on` section should become:
   
   ```yaml
   on:
     push:
       branches:
         - main
   ```

### Role of Branches

1. `master`: branch stores the Hexo configuration and Markdown files
2. `gh-pages`: branch stores the website Hexo generates

### What will Github CI do?

It will switch to `master` branch, ask Hexo to generate a new website, then switch to `gh-pages` branch, commit the new website to this branch, when we have pushed a new commit to `master` branch. How these jobs are done is described in the `.github/workflows/pages.yml` in detail.

