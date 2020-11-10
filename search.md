---
layout: default
permalink: "search"
title: "Search"
css: "/css/search.css"
---

## Search DevOps Club

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
  window.onload = function()
  {
    var searchBox =  document.getElementById("gsc-i-id");
    searchBox.placeholder="Search DevOps Club";
    searchBox.title="Search DevOps Club";   
   }
</script>
<gcse:searchbox></gcse:searchbox>
<gcse:searchresults></gcse:searchresults>
</div>