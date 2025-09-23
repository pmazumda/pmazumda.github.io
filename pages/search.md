---
layout: default
permalink: "search"
title: "Search"
css: "/css/customsearch.css"
sitemap:
  lastmod: 2022-05-14
  priority: 0.7
  changefreq: 'monthly'
  exclude: 'yes'
---

## Search middleware-bytes

<div id="google-custom-search">
<script>
  (function() {
    var cx = '009752839592299086617:ra7nbqw5yfo';
    var gcse = document.createElement('script');
    gcse.type = 'text/javascript';
    gcse.async = true;
    gcse.src = (document.location.protocol == 'https:' ? 'https:' : 'http:') +
        '//www.google.com/cse/cse.js?cx=' + cx;
    var s = document.getElementsByTagName('script')[0];
    s.parentNode.insertBefore(gcse, s);
  })();
</script>
<gcse:searchbox></gcse:searchbox>
<gcse:searchresults></gcse:searchresults>
</div>