---
layout: post
title:  "Implementing iMessage Link Previews with Open Graph"
date:   2019-03-05
---

*Goal: Improve the quality of shared links from [Push of Love][push-of-love].*

* [Default iMessage Links](#default-imessage-links)
* [Enabling iMessage Link Previews](#enabling-imessage-link-previews)
* [Implementing Link Previews on Push of Love](#implementing-link-previews-on-push-of-love)
* [Resources](#resources)

-----

## Default iMessage Links
Out of the box when you share a link on iMessage it looks like this:

![iMessage link preview without image][link-preview-without-image]

That's pretty boring; it doesn't provide me with any useful information about the content of the page and doesn't make me want to click on it.
 
We can do better.

## Enabling iMessage Link Previews
It turns out we can make the link more compelling by adding some [Open Graph metadata][open-graph] to the page.

If we add the `og:title` and `og:image` Open Graph meta tags to the page like this:

```html
<meta property="og:title" content="iPhone" />
<meta property="og:image" content="https://example.com/image.png" />
```

The shared link will now have a custom title and image, and look like this:

![iMessage link preview with an image][link-preview-with-image]

Awesome!

## Implementing Link Previews on Push of Love

Now let's apply this to Push of Love! 

First lets see what we are starting with:

![PushOfLove.com iMessage link preview with heading and logo][push-of-love-link-preview]

Interesting! We already have a title and a logo. 

It turns out, if iMessage is unable to find an `og:title` it looks for the `<title>` tag on the page. In addition, if iMessage is unable to find a `og:image` it looks for a logo and in this case it found our [apple-touch-icon.png][apple-touch-icon] on the page.

Let's specify an `og:image` in the `<head>` of the page:

```html
<meta property="og:image" content="https://pushoflove.com/icon.png" />
```

![PushOfLove.com iMessage link preview with heading and image][push-of-love-link-preview-image]

Nice, now we can see a large image of our logo when the link is shared!

This is great for the homepage of Push of Love, but what about the pages for individual pushes?

💡 Idea! What if each "push" has it's own image that demonstrates how it would look on your phone?!

That's a great idea... but we create new content for Push of Love every day and we really don't want to spend all our time creating new images in Photoshop... 

*So, can we automate it?*

**Hell yes we can.**

*Stay tuned for the next post on how.*


## Resources

* Official [Open Graph Documentation][open-graph]
* Facebook's [Open Graph Debugger][open-graph-debugger]
* Apple's (no longer updated) [Best Practices for Link Previews][apple-link-preview-best-practices]


[push-of-love]: https://pushoflove.com
[open-graph]: http://ogp.me
[open-graph-debugger]: https://developers.facebook.com/tools/debug/
[apple-link-preview-best-practices]: https://developer.apple.com/library/archive/technotes/tn2444/_index.html
[apple-touch-icon]: https://developer.apple.com/library/archive/documentation/AppleApplications/Reference/SafariWebContent/ConfiguringWebApplications/ConfiguringWebApplications.html#//apple_ref/doc/uid/TP40002051-CH3-SW4


[link-preview-without-image]: {{ site.url }}/assets/imessage-link-preview-default.png
[link-preview-with-image]: {{ site.url }}/assets/imessage-link-preview-with-image.png
[push-of-love-link-preview]: {{ site.url }}/assets/imessage-link-preview-push-of-love-initial.png
[push-of-love-link-preview-image]: {{ site.url }}/assets/imessage-link-preview-push-of-love-image.png
