---
title: Analysis
layout: default
nav_order: 2
---
For a basic, initial analysis of the contents of a carrier or transfer, we use PowerShell:
```
gci -file -recurse -force | group extension | select name,count,@{n='Size(GB)';e={ ((($_.group | measure length -sum).sum)/1gb).ToString('N2')}} 
```
This gives the number of total file size (in GB) of all files in and under the present working directory, sorted by extension.