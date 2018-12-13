---
title: {{ title }}
date: {{ date }}
tags:
---
<% for (var link in site.data.menu) { %>
  <a href="<%= site.data.menu[link] %>"> <%= link %> </a>
<% } %>