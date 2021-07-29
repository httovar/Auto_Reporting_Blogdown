# Automated Reporting with `Blogdown`

This is the more applied and technical documentation of the portfolio I created for the Advanced Data Analysis with R Class. Check out the [website](https://kind-wescoff-aaca29.netlify.app/about/) for a  short introduction to the portfolio. The basic idea of the project is to create an online reporting infrastructure that automatically goes through the steps of gathering the data, generating a (standardized) report, and publish the report online. Additionally, I found it desirable to find a solution that automatically initiates each iteration of the reporting cycle without having to source the R Script manually. As the class is focused on `R`, the entire project is meant to be controlled from within the `R` ecosystem. This ReadMe document goes over the core ideas of the project and also introduces the basic `R` commands and steps to reproduce a similar project. 

## Creating Websites with `blogdown`

The central infrastructure for this project is provided by the [`blogdown` package](https://bookdown.org/yihui/blogdown/). The package provides an interface between `R`/`RStudio` and the static website generator `hugo`. Static websites as opposed to dynamic websites, consist of a collection of files (markdown, HTML, CSS, etc.) that are then processed to create a website. The website is static in the sense that it looks the same for every user and does not allow interaction that goes beyond accessing different parts of the website. Dynamic websites, on the other hand, require server-side scripts (like PHP) that allow for a dynamic interaction between the user and the website. For a reporting tool, however, a static website is sufficient, as there is no need for dynamic interaction.

In practice, the `blogdown` package requires a special folder structure that will convey where a certain file will be featured on the website. However, the package comes with the infrastructure to create this folder structure in the workflow of setting up a new blog. The basic workflow follows the following steps:

- Create a new Project within `RStudio` (THis is optional but makes things much easier)
- Pick a `hugo` theme for your website from https://themes.gohugo.io/ . Go to the download page of the respective theme and save the GitHub URL.
- Use `blogdown::new_site(theme = GITHUB_URL, sample = T, to_yaml = T, empty_dirs = T)` to create the folder structure and files necessary for the theme. The arguments in addition to the `theme` argument are not necessary but used for convenience.
- Start writing blog posts. Use `blogdown::new_post()` to create a new document (and the necessary directories) that you can fill with content. Note that by default, `blogdown::new_post()` will open a new Markdown document. You can change this either by specifying the exact file name (including the extension) with the `file` argument or by setting `ext` equal to the desired file extension. Markdown documents are useful for write ups but `blogdown` also supports `.Rmd` files that allow for statistical programming.
- Build the website using `blogdown::build_site()`. Note that you have to set the `build_rmd` equal to `TRUE` if RMarkdown documents are supposed to be processed. Alternatively, it is possible to use `blogdown::serve_site()` to create a local preview of the website. 
- Host your website on a platform like https://netlify.com to make it publicly accessible. This will require a remote git repository (like GitHub) for the project. Further, git has to be installed on the computer.

When all these steps are completed, the blog is a fully live website. When creating new posts, use the `blogdown::new_post()` command, fill the document with content and then commit and push it to the remote repository. This will then be communicated to netlify and in turn lead to changes on the website. There are a few things to keep in mind when creating a new website with `blogdown`. When picking a `hugo` theme it is often advisable to pick a simpler theme rather than a complicated one. The more complicated the resulting website is, the more complicated the files are that organize the website. Little tweaks to the website can often return in unexpected changes. For experienced users that are familiar with web development and the `yaml` and `toml` markdown language, this might not be a substantial problem. For new users, however, choosing a complicated, fancy theme can result in substantial frustrations. 

Further, in my personal experience, it is a easier to mix up the process detailed above. When creating a new site, follow these steps:

- Create a new remote git repository (e.g. on GitHub/GitLab). Save the URL.
- Create a new Project in `RStudio`. Choose to create the project from Version Control and then specify the remote repository that was just created.
- Add two entries to the .gitignore file: The name and extension of the `RProject` (this does not really provide any value on your repo), and `public`. The public folder contains all the HTML files that are used when hosting the website. However, netlify will create this itself and thus, the public folder does not provide any value on your repo.
- Download the theme and create the file structure as detailed above.
- Stage, Commit, and Push the files to your remote repository. I find it advisable to use git commands in the `RStudio` Terminal rather than the manual Git Pane (also in `RStudio`). When staging a lot of files, the git pane can crash easily. The following commands will push all changes to the remote repository from the Terminal Command Line
  - `git add -A`: This will stage all changed files
  - `git commit -m "Initial Commit"`: This will commit all changes and add "Initial Commit" as message.
  - `git push`: This will push all changes to the remote repo.
- If desired, use `blogdown::new_post` to create new blog posts now.
- Connect to netlify by connecting remote git repo and netlify.

## Automating Report Creation with Dynamic `RMarkdown` Writing

Within the `blogdown` structure, the core idea of the project is to create a standardized blog post that is automatically published in regular intervals. This idea poses two problems outlined below. The following section will go over the solutions to both of these problems. 

- The `blogdown::new_post` function only opens a new document but does not allow to fill a document with text and code. Under different circumstances, a parameterized Rmarkdown document would be the solution but this approach does not work with the `blogdown` infrastructure. The `.rmd` file produces a specified file type as output. However, `blogdown` processes the actual `.rmd` file (and also can't supply the parameters). Instead the `.rmd` files have to be written dynamically from a predefined R script and then placed into the the folder structure that is usually created with `blogdown::new_post`.
- The website that is hosted by netlify only updates when the remote git repository is updated. Thus, for a truly automated reporting infrastructure, the changes to the folder structure have to be pushed to the repo automatically as well.

The process of generating a Rmarkdown file dynamically, actually takes place from within an R Script rather than an `.rmd` file. The idea is to create a long character string that mimics the syntax of an `.rmd` file within the R script and then use the `write` function to write this character string to a file with an `.rmd` extention (and thus creating an `RMarkdown` file). The following code chunk exemplifies this idea with the standard example content of a new `RMarkdown` file. This process makes extensive use of the `paste()` and `paste0()` functions to create a character string that is readable when creating it within R. The `sep` argument of the `paste()` function also allows to closely follow the syntax of an `.rmd` file, which is especially important when creating the `yaml` section that is sensitive to indentations and white space.

```
yaml_header <- paste("---",
                     paste("title:", "'Example'"),
                     paste("date:", Sys.Date()),
                     paste("author:","'Henning Tovar'"),
                     paste("output:", "html_document"),
                     "---",
                     sep = "\n")


code_chunk1 <- paste("```{r setup, include=FALSE}",
                     "knitr::opts_chunk$set(echo = TRUE)",
                     "```",
                     sep = "\n")
                     
markdown_body1 <- paste("## R Markdown",
                        paste("This is an R Markdown document. Markdown is a simple formatting syntax for authoring HTML,",
                        "PDF, and MS Word documents. For more details on using R Markdown see <http://rmarkdown.rstudio.com>.",
                        "When you click the **Knit** button a document will be generated that includes both content as well as",
                        "the output of any embedded R code chunks within the document. You can embed an R code chunk like this:"),
                        sep = "\n")

code_chunk2 <- paste("```{r cars}",
                     "summary(cars)",
                     "```",
                     sep = "\n")
                     
markdown_body2 <- paste("## Including Plots",
                        "You can also embed plots, for example:",
                        sep = "\n")

code_chunk3 <- paste("```{r pressure, echo=FALSE}",
                     "plot(pressure)",
                     "```",
                     sep = "\n")
                     
markdown_body3 <- paste("Note that the `echo = FALSE` parameter was added to the code chunk to prevent printing of the R",
                        "code that generated the plot.")

complete_doc <- paste(yaml_header,code_chunk1, markdown_body1, code_chunk2, markdown_body2,
                      code_chunk3, markdown_body3, sep = "\n")
                      
write(complete_doc,
      paste0(path,"/","example_doc.Rmd"))
```

For the actual report, a few steps preceded the writing of the `.rmd` file. The entire Script can be found in this [GitHub repositoty](https://github.com/httovar/auto_reporting_dyn_rmd). Note that for the intended directory structure, the two repositories are stacked. The folder with the dynamic markdown script is the parent directory to the blogdown folders. This allows for easy access to sub-directories from within the dynamic R script, which is important for automated blog creation (as will become more obvious below). The file structure looks similar to this structure:

```
- Dynamic_Markdown_Folder
  - Dynamic_Markdown_R_Script.R
  - Blogdown_Folder
    - content
      - post
        - folders_for_each_blog_post
      - page
      - categories
    - other_Blogdown_specific_folders_and_files
```
In order to produce the report, the R script first scrapes data from the web, then processes this data and saves the data to a `.csv` file. Then the `.rmd` files is produced that uses the scraped data to generate tables and plots. Since this project is not focused on web scraping, I am not detailing the exact procedure here (although it obviously is in the [R script](https://github.com/httovar/auto_reporting_dyn_rmd/blob/main/Dynamic_Reporting.R)). Essentially, the script scrapes the Top 100 Single charts and some additional meta information. This information is used for simple tables, plots and some additional descriptive text. The actual analysis is not particularly involved but could be further developed if so desired.
The crucial step is to place the `.rmd` and `.csv` files in the correct position for processing with `blogdown::build_site()`. Under the hood, `blogdown::new_post()` creates a new folder in the `post` directory for each new blog post. This folder is, subsequently, processed when building the website. The next step of the R script is, consequently, to create a new folder within the `post` directory and then populate this folder with the `.rmd` and `.csv` files. The `blogdown` folder structure is now ready to be processed and deployed.

## Automated Git Commands and Tasks Scheduling

The last two steps of this process are (i) staging, committing, and pushing the new folder and files to the remote git repository, (ii) automating the entire work flow. Since both steps require scripts that are tailored to my personal file directory, the actual scripts are not publicly accessible. However, this documentation shows the general code to reproduce the results.

In order to automate git commands, the `git2r` package is utilized. The package provides function to interact with git and remote git repos from within R. The `git_commands.R` script that is called at the end of the [dynamic MarkDDown  script](https://github.com/httovar/auto_reporting_dyn_rmd/blob/main/Dynamic_Reporting.R) contains the following commands. Note the comments for more information. To reproduce this step, personal access token or SSH key to the remote git repository are required.

```
#Set Up
library(git2r)

#Before the changed files can be pushed to github, they have to be created.
#Thus, the blogdown::build_site() function is called

#Setting the pandoc environment variable since call from different environment.
Sys.setenv(RSTUDIO_PANDOC="C:/Program Files/RStudio/bin/pandoc")

#Running the build site command
blogdown::build_site(build_rmd = T)

#completing git commands
git2r::add(path = file_path_to_changed_files)

git2r::commit(all=TRUE, message="Commit Mesage")
 
git2r::push(credentials = c(github_name, github_Peronsal_Accesss_Token))
```

Lastly, the entire procedure is automated with the Windows Task Scheduler (for Linux and Mac based system `Cron` does the same). As the name indicates, the task scheduler allows for the scheduled execution of certain tasks, including running scripts. The scripts are run from the Windows command line rather than from inside `RStudio` (or other IDEs). Thus, the exact path to the R script has to be indicated from the root directory (C: in my case). While it possible to run R scripts directly from the windows command line and schedule this, it is less error prone to create a Windows `batch` file that contains the command line command. Simply create a raw text file that contains the following code and after saving it, change the file extension to `.bat`. Note that this assumes that the `RScript.exe` utility is on the PATH environmental variable. The `batch` file will execute the specified routine when opening the file. 

```
@echo off
Rscript	PATH_TO_RSCRIPT
```

The Windows Task Scheduler has an instructive GUI that guides through the process of creating a task. Within this process refer to the new `.bat` file. It should be noted that scheduled tasks can only be run when the computer is running. It is not possible to execute them while the computer is powered off (or in sleep mode).

This step completes the process of automatically posting 


  