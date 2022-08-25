---
title: 'Hexo pitfalls: `post_asset_folder` doesn''t work'
tags: Pitfalls
---

# the `post_asset_folder` Option

There are two way to store iamges in a Hexo blog. The simplest way is to create a `image` folder inside the `source` folder, and store all images inside this folder. Later you can reference this photo in a post using something like `![](images/images.png)`.

Hexo also provides a more organized way to manage images and other static resources: using a "Post Asset Folder". It's a folder with the same name of the post, and located in the same folder. After you set `post_asset_folder: true` in the `_config.yml`. Everything referenced by the post can be put inside this folder, and will be associated with the post, then you can referenced them with a simple relative path in the post.

For example, there was a post named `a-guide-to-markdown.md`, and if I want to reference a picture called `markdown-icon.png` in my post, I can put it in a folder that is also called `a-guide-to-markdown`, sits beside the post, and add the picture in it. The directory structure will be:

```yaml
- source
    - _post
        - a-guide-to-markdown
            -markdown.png
        - a-guide-to-markdown.md
```

Then inside the post, I can simply write `![](markdown.png)` in the post. Hexo will find the associated folder for the image while generating website.

# the Problem

Well, it doesn't work. If a article is publish under an path of `/2022/08/20/a-guide-to-markdown/`, and I inserted an image in my post: `![](markdown-icon.png)`, the image's link in the output HTML should be `/2022/08/20/a-guide-to-markdown/markdown-icon.png`, but it doesn't. Instead, it mistakenly links the images to the root path `/./markdown-icon.png`, as if the option weren't turned on.

# the Solution

Not only should you set the `post_asset_folder` to true, but also `postAsset` of the `marked` object.

``` yaml
post_asset_folder: true

marked:
  postAsset: true
```

The second one is a options of the Hexo plugin `hexo-renderer-marked`. The document is here: [https://github.com/hexojs/hexo-renderer-marked](https://github.com/hexojs/hexo-renderer-marked). IMO, this is utterly a terrible design. I see no reason why we have to set two option to enable one functionality. More worst, Hexo's document doesn't mention it at all!