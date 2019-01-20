---
title: {{ title }}
date: {{ date }}
tags:
categories:
---



---
<div id="container"></div>
<link rel="stylesheet" href="https://imsun.github.io/gitment/style/default.css">
<script src="https://imsun.github.io/gitment/dist/gitment.browser.js"></script>
<script>
var gitment = new Gitment({
  owner: 'tinyalley',
  repo: 'comments',
  oauth: {
    client_id: '4fc683c05fc3ba5bcf66',
    client_secret: 'eaf39a9547e39a17b74ee130e79385f8117827ae',
  },
})
gitment.render('container')
</script>