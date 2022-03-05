---
title: Responsive Youtube
date: 2014-01-29
description: So when I actually loaded my website on my phone I saw that my embedded Youtube videos weren't resizing automatically - i.e. not responsive. Some quick Googling and one of the top results was a nice elegant solution via CSS. I pretty much just added a margin and it was good to go.

draft: false
type: post
---

So when I actually loaded my website on my phone I saw that my embedded Youtube videos weren't resizing automatically - i.e. not responsive. Some quick Googling and one of the top results was a [nice elegant solution via CSS](http://avexdesigns.com/responsive-youtube-embed/ "Responsive Youtube Embed"). I pretty much just added a margin and it was good to go.

{{< highlight css >}}

.video-container {
  position: relative;
  margin: 30px;
  padding-bottom: 50%;
  padding-top: 30px;
  height: 0; 
  overflow: hidden;
}
 
.video-container iframe,
.video-container object,
.video-container embed {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}

{{< / highlight >}}

It's now one less thing to keep me awake at night. Now if I could just find the time to make my website not look like the default [bootstrap](http://getbootstrap.com/ "Bootstrap") theme that would be real nice - maybe I'll extend one of the [these themes](http://www.blacktie.co/category/themes/ "Black Tie - Free Bootstrap Themes"). And maybe I'll get a [favicon](http://en.wikipedia.org/wiki/Favicon" "Favicon") too! One day...
