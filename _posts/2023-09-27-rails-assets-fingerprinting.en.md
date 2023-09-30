---
title: "What are these things at the end of the asset file URLs?"
tags: [TIL, Rails]
layout: post
permalink: 2023-09-26-rails-assets-fingerprinting.html
lang: en
---


## This is my favorite cat picture
![My favorite cat](/assets/images/eepy-cat.jpeg)

### While publish this cat photo in my Rails application's production environment, I noticed something like this in the image URL:
<br>

```
my-site.com/assets/cuties/eepy-cat-99b7082082fe58d58fe828581d4a47aedf277e1cf0e47a660529d9ca923feece.jpeg
```
<br>

### What is this chaos and random strings/integers at the end of the asset file's URL?

This things known as `fingerprints`, Rails applies these unique identifiers the CSS, JS and image assets.

The main reason of Rails (or any other framework) using this method is to create fast web pages by caching the asset files.

As is known, when you request a page from the Client in Rails, the browser goes to the server and gets the resources of the relevant page.
However, instead of fetching the entire page each time, HTTP/Caching works between the browser and the server. This is where fingerprint helps us.

In the request it makes, if the browser has previously cached certain assets, it checks their version/updity through fingerprints and by checking the identity of the current data on the server and the cached file in the browser with fingerprints, if the file has not changed, a request is not made for that asset.

Thus, the browser uses bandwidth more efficiently.

<br>

### I realized the importance of fingerprinting when trying to pass an image location to the JS file.
I initially passed the image path to JS as `data-icon-url-value=assets/cuties/eepy-cat.png`. Yes, it was working on my local machine :) but it is failed in production.
I noticed the importance of fingerprinting and wanted to share the research results.

So, keep using the `image_url('cuties/eepy-cat')`
instead of `src='assets/cuties/eepy-cat'` for fingerprinting/caching and accessing your assets in the correct way.

<br>

-----------
<br>

Useful resources:

[Fingerprinting Images to Improve Page Load Speed](https://docs.imgix.com/best-practices/fingerprinting-images-to-improve-page-load-speed)

[What is Fingerprinting and Why Should I Care?](https://guides.rubyonrails.org/asset_pipeline.html#what-is-fingerprinting-and-why-should-i-care-questionmark)
