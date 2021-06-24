---
# Common-Defined params
title: "A Clarification About the Same-Origin Policy"
date: "2021-06-24"
description: "Same-origin policy CORS"
categories:
  - "Web"
tags:
  - "CORS"
  - "Same-origin"
# menu: main # Optional, add page to a menu. Options: main, side, footer

# Theme-Defined params
#thumbnail: "img/placeholder.jpg" # Thumbnail image
#lead: "Gold grade is a term used in gold mining, and should be used as a measure of the quality of gold ore – that is the raw material obtained from mining." # Lead text
---

转自我在 [Stackoverflow 的回答](https://stackoverflow.com/a/68112216/1436289):

I finally made myself clear after having been struggling half a hour about [this saying][1]:

> Cross-origin writes are typically allowed....
>
> Cross-origin embedding is typically allowed....
>
> Cross-origin reads are typically not allowed...

---

Depending on whether interacting with the server, I'd prefer to classify the same-origin policy into two types, instead of three:

1. reading the **already loaded resource** of originA from within originB in the browser
2. requesting the remote resource of originA from within originB in the browser

The first is the "Cross-origin reads"'s case and the second is the "Cross-origin writes" and "Cross-origin embedding"'s.

The browser will definitely block the first case, so that a javascript snippet of websiteA cannot read the cookie set by websiteB - thus ensures the safety of our web accounts.

For the second case, there're many cases a request is initiated from within one origin to another, such like:

1. the HTML `<a href=...` tag that directs the browser the request another resource
2. the HTML `<form action=... method="POST">` tag that makes the browser to post a form to another website
3. the `<link rel="stylesheet" href="…">` tag which loads CSS style from another site and embeds it to our own webpage
4. the `<img>` tag which loads the image from elsewhere and embeds it to our own webpage
5. the `<script>` tag which loads the javascript code snippet from elsewhere and embeds into our own webpage
6. `XMLHttpRequest` AJAX request
7. Web Fonts (for cross-domain font usage in @font-face within CSS)

The browser allows the 1-5 scenarios while blocks 6 and 7, there're other scenarios the browser blocks too, you can check [this link](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#what_requests_use_cors) for all the scenarios blocked by the browser.


  [1]: https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy#cross-origin_network_access
