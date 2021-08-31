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
  - excerpt: "The content on this website is (and will always be) free for everyone to access at any time and from anywhere. If you enjoyed the content, consider sending me a message instead.  Now, if you are feeling generous and are in a stable financial situation, then use one of the addresses below to donate. The addresses change every once in a while for multiple reasons."
---
***
{% include feature_row id="intro" type="center" %}
{% assign author = page.author | default: page.authors[0] | default: site.author %}
{% assign author = site.data.authors[author] | default: author %}

- **Bitcoin** (SegWit) address:
  
  ```
  {{ author.btc }}
  ```
  [![BTC SegWit 01 QR code](/assets/img/site/bitcoin-segwit-01.png){:.PostImage}](bitcoin:{{ author.btc }})

- **Litecoin** address:

  ```
  {{ author.ltc }}
  ```  
  [![LTC 01 QR code](/assets/img/site/litecoin-01.png){:.PostImage}](litecoin:{{ author.ltc }})

