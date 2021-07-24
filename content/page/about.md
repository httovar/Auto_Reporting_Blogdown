---
aliases:
- about-us
description: Automated Online Reporting with Blogdown and Dynamic Markdown Generation
menu:
  main:
    pre: user
    weight: -90
title: About
---


This website hosts a weekly summary report for the German Top 100 Single Charts. The data gathering, report generation and subsequent posting of the report takes places fully automatically through [`R`](https://cran.r-project.org/). This is possible by combining the [`blogdown` package](https://bookdown.org/yihui/blogdown/) that is focused on creating website from within `R` with dynamic markdown generation, version control, and task scheduling with the [`taskscheduleR` package](https://cran.r-project.org/web/packages/taskscheduleR/vignettes/taskscheduleR.html).