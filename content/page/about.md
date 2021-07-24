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


This website hosts a weekly summary report for the German Top 100 Single Charts. The data gathering, report generation and subsequent posting of the report takes places fully automatically through [`R`](https://cran.r-project.org/). This is possible by combining the [`blogdown` package](https://bookdown.org/yihui/blogdown/) that is focused on creating website from within `R` with dynamic markdown generation, version control, and task scheduling with the [`taskscheduleR` package](https://cran.r-project.org/web/packages/taskscheduleR/vignettes/taskscheduleR.html). `blogdown` is a Hugo-based website generator that creates static websites. In contrast to dynamic websites (requiring scripts on the server side), static websites are basically a special folder structure that contains markdown documents, CSS, and HTML files. With `blogdown` it is possible to create (R-)Markdown documents and then let the package do the work of converting them to the HTML files (plus additional auxiliary files) necessary for a simple blog.

This project utilizes the fact that simple RMarkdown documents can be converted into blog posts, by creating a standardized RMarkdown template that is then filled in with freshly scraped data. Under different circumstances, [parameterized Reports](https://bookdown.org/yihui/rmarkdown/parameterized-reports.html) are an appropriate tool for standardized reports. However, this approach only produces an output file (pdf, HTML, etc.) that uses the specified input parameters. They do not produce a separate Markdown file that can be passed to `blogdown`. The solution to this problem is to write the RMarkdown dynamically from within an R script. In a nutshell, the R script produces a long character string that follows the RMarkdown syntax and fills in the parameterized inputs as well. The string is then written to a file with the `.Rmd` ending and is placed within the folder structure utilized by `blogdown`. 

Lastly, `blogdown` will convert the new `.Rmd` file to a blog post. The website is hosted on `netlify` which in turn is connected to a external git repository (i.e. github/gitlab). So before, the report is accessible online, the new files have to pushed to GitHub. As the entire reporting structure is meant to be automated, the staging, committing and eventual pushing of the files is done through a small R Script that executes git commands through the RStudio Terminal. The entire reporting structure is automated with the `taskscheduleR` package. The package utilizes the Windows Task Scheduler to run specified R Scripts at a given interval (in this case weekly). 

For a more technical introduction, see the ReadMe page of the [GitHub repository](https://github.com/httovar/auto_reporting_blogdown)