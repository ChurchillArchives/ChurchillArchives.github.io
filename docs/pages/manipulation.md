---
title: Manipulation
layout: default
nav_order: 7
---

```
ffmpeg -i "$in" -c:v libx264 -movflags faststart -pix_fmt yuv420p -vf yadif -c:a aac "$out" -hide_banner
```
