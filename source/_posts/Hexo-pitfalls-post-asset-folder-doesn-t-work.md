---
title: 'Hexo pitfalls: `post_asset_folder` doesn''t work'
tags: Pitfalls
date: 2022-08-25 20:00:13
---

# the `post_asset_folder` Option

There are two way to store iamges in a Hexo blog. The simplest way is to create a `image` folder inside the `source` folder, and store all images inside this folder. Later you can reference this image using something like `![](images/images.png)`. Following is the directory structure: 

```yaml
- source
    - _post
        - a-guide-to-markdown
    - images
        - markdown.png
```

Hexo also provides a more organized way to manage images and other static resources: by using a "Post Asset Folder". It's a folder has the same name of the post, and located in the same folder. After you set `post_asset_folder: true` in the `_config.yml`, You can put resources in this folder and reference them with *just it's name*

For example, there was a post named `a-guide-to-markdown.md`, and if I want to reference a picture called `markdown-icon.png` in my post, I can create a folder that is also called `a-guide-to-markdown` which sits beside the post, and add the picture in it. The directory structure will be:

```yaml
- source
    - _post
        - a-guide-to-markdown
            - markdown.png
        - a-guide-to-markdown.md
```

Then inside the post, I can simply write `![](markdown.png)`. Hexo will find the associated folder and generate a correct URL while generating website.

# the Problem

Well, it doesn't work. If a post's path is `/2022/08/20/a-guide-to-markdown/`, and I inserted an image in my post: `![](markdown-icon.png)`, the image's link in the output HTML should be `/2022/08/20/a-guide-to-markdown/markdown-icon.png`, but it doesn't. Instead, it mistakenly links the images to the root path `/./markdown-icon.png`, as if the option weren't turned on.

# the Solution

Not only should you set the `post_asset_folder` to true, but also `postAsset` of the `marked` object.

``` yaml
post_asset_folder: true

marked:
  postAsset: true
```

The second one is a options of the Hexo plugin `hexo-renderer-marked`. The document is here: [https://github.com/hexojs/hexo-renderer-marked](https://github.com/hexojs/hexo-renderer-marked). IMO, this is utterly a terrible design. I see no reason why we have to set two option for one functionality. More worst, Hexo doesn't document it at all!

# How I Found the Solution

I didn't get any useful answer from the official website nor Google, so I look into the code and try debug the process of `hexo g`. After some trials I found following key code in the file `node_modules\hexo-renderer-marked\lib\renderer.js`

```js
module.exports = function(data, options) {
  const { post_asset_folder, marked: markedCfg, source_dir } = this.config;
  const { prependRoot, postAsset, dompurify } = markedCfg;
  const { path, text } = data;

  // ... other codes

  let postPath = '';
  if (path && post_asset_folder && prependRoot && postAsset) {
    const Post = this.model('Post');
    // Windows compatibility, Post.findOne() requires forward slash
    const source = path.substring(this.source_dir.length).replace(/\\/g, '/');
    const post = Post.findOne({ source });
    if (post) {
      const { source: postSource } = post;
      postPath = join(source_dir, dirname(postSource), basename(postSource, extname(postSource)));
    }
  }

  // ... other codes
}
```

This file is about transforming a Markdown post to HTML, and this part of code it is about setting the variable `postPost`, which I presume it is the base path of resources referenced in this Markdown post. By default it is empty, and probably means the root path `/`. This matches the bahaviour when `post-asset-folder` is disabled. So I guessed, if `post-asset-folder` is enabled. The condition `path && post_asset_folder && prependRoot && postAsset` should be true, and the statements in the *if-block* should be executed and get the `postPath` rewritten. So what went wrong? I noticed that all variable is *truthy*, **excepts for** `postAsset`. Therefore, if I can find out where the `postAsset` is set, and why it isn't set to be `true`, the problem will be solved.

The `postAsset` initially is defined in the file `/node_modules/hexo-renderer-marked/index.js`, and it is injected when the plugin is being installed.

```js
/* global hexo */

'use strict';

const renderer = require('./lib/renderer');

hexo.config.marked = Object.assign({
  gfm: true,
  pedantic: false,
  breaks: true,
  smartLists: true,
  smartypants: true,
  modifyAnchors: 0,
  autolink: true,
  mangle: true,
  sanitizeUrl: false,
  dompurify: false,
  headerIds: true,
  anchorAlias: false,
  lazyload: false,
  prependRoot: true,
  postAsset: false, // HERE!
  external_link: {
    enable: false,
    exclude: [],
    nofollow: false
  },
  descriptionLists: true
}, hexo.config.marked);

renderer.disableNunjucks = Boolean(hexo.config.marked.disableNunjucks);

hexo.extend.renderer.register('md', 'html', renderer, true);
hexo.extend.renderer.register('markdown', 'html', renderer, true);
hexo.extend.renderer.register('mkd', 'html', renderer, true);
hexo.extend.renderer.register('mkdn', 'html', renderer, true);
hexo.extend.renderer.register('mdwn', 'html', renderer, true);
hexo.extend.renderer.register('mdtxt', 'html', renderer, true);
hexo.extend.renderer.register('mdtext', 'html', renderer, true);
```

So, how can I change it? The object assign to `hexco.config.marked` looks like a configuration file for the plugin, so probably I can change it via changing Hexo's configuration file. I visited the official website of the plugin `hexo-renderer-makred`, and in the document, I found the option's definition:

![the postAsset option](./postAsset-option.PNG)

and it also said:

![](./change-config.PNG)

Volia, problem solved! I just enable this option in my `_config.yml`, then regenerate the webpage, and everything is done!

# Unsolved Problems

If the article is a draft instead of a post, the images still aren't displayed.
