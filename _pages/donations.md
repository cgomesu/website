---
title: Donations
layout: single
permalink: /donations/
header:
  overlay_color: "#000"
  overlay_filter: "0.6"
  overlay_image: /assets/img/site/banner-page-07.jpeg
  caption: "by [**Rebecca Guay**](https://en.wikipedia.org/wiki/Rebecca_Guay)"
excerpt: "Cryptocurrencies and addresses used for donations."
intro: 
  - excerpt: "The content on this website is (and will always be) free for everyone to access at any time and from anywhere. If you enjoyed the content, consider sending me a message instead.  Now, if you are feeling generous, use one of the methods below."
---
***
{% include feature_row id="intro" type="center" %}
{% assign author = page.author | default: page.authors[0] | default: site.author %}
{% assign author = site.data.authors[author] | default: author %}

- **Ko-fi**: [ko-fi.com/cgomesu](https://ko-fi.com/cgomesu)
- **Bitcoin**: ```{{ author.btc }}```
- **Litecoin**: ```{{ author.ltc }}```
