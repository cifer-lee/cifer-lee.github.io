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

To better understand with these three phrases, the first thing you need to keep in mind is that **same-origin policy is about restricting access the loaded result from one origin to another origin, within your *browser***, so it does nothing directly to do with cross-origin requesting, which involves requesting to the server.

So what does that mean? it means that you have loaded two websites A and B with different origins, in website A there's a javascript snippet that wants to read the loaded content in website B, the browser - our safeguard, will block this action. This is the cross-origin reads case.

Let's see how the cross-origin writes and embedding cases look like:

1. the HTML `<a href=...` tag that directs the browser the request another resource
2. the HTML `<form action=... method="POST">` tag that makes the browser to post a form to another website
3. the `<link rel="stylesheet" href="…">` tag which loads CSS style from another site and embeds it to our own webpage
4. the `<img>` tag which loads the image from elsewhere and embeds it to our own webpage
5. the `<script>` tag which loads the javascript code snippet from elsewhere and embeds into our own webpage

See the difference? Both *Cross-origin writes* and *Cross-origin embedding* initiate a **new web request** to the server of the other origin, instead of reading the **already loaded result** from within the browser! Also it's important to note that whether the **new web request** results in a backend change doesn't matter, we don't care whether it's a `GET` or `POST` request, as long as it causes a new interaction with the server, it is *Cross-origin writes*(or *Cross-origin embedding*).

So, to conclude:

* **Cross-origin reads disallowed** means disallowing reading the loaded result of originB from originA within the browser
* **Cross-origin writes(and embedding) allowed** means allowing requesting to originB from originA within the browser, this is surely allowed, because this is how the WWW works, imagine the nightmare of the WWW if websites are not able to link to each other?


  [1]: https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy#cross-origin_network_access
