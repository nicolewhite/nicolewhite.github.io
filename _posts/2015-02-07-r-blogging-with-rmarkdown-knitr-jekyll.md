---
title: "Blogging About R Code with R Markdown, Knitr, and Jekyll."
layout: post
comments: true
---

# Blogging About R Code with R Markdown, Knitr, and Jekyll.

{% raw %}

I've been blogging with [Jekyll](http://jekyllrb.com/) for a while now, where most of my blogs contain snippets of R code. Previously, my workflow was somewhat sloppy: I would copy-paste the snippets of R code into my `.md` file along with the expected output of the code (e.g. printing a data.frame or number). However, I often ran into problems where I would change the code snippet but forget to change the output, thus creating disagreements between what the code snippet was doing and the expected output I had copy-pasted in previously. There's also the possibility that the R code had errors, but I wouldn't know because my blog posts weren't derived directly from the R code itself.

Now, I've started writing my blog posts in R markdown (a `.Rmd` file), then using [knitr](http://yihui.name/knitr/) to convert that `.Rmd` file into a `.md` file. This solves the main two problems described above. With this new workflow:

* The output is ensured to agree with what the R code is doing.
* There will not be any errors in my R code, because `knitr` will stop and throw an error instead of creating the `.md` file.

The rest of this blog post assumes a basic knowledge of Jekyll and that you plan to use it with [GitHub pages](https://pages.github.com/).

The following walkthrough refers to [this live example](http://nicolewhite.github.io/r-knitr-jekyll/2015/02/07/exploring-the-cars-dataset.html). My new workflow is as follows:

Begin with a `.Rmd` file in the `_drafts` directory. Name it according to Jekyll standards, which is lowercase words separated by hyphens. I'll name mine `exploring-the-cars-dataset.Rmd` and start with the default content that's there when you create a new R Markdown file in RStudio.

Edit the front matter as needed. I remove `output: html_document` and add the necessary `layout: default` setting that I'm using in my example blog.

Make sure to set the `fig.path` if you'll be generating any figures. In my example, a plot of the `cars` dataset is generated and I want Jekyll to look in my `images` directory for this `.png` file. So, I'll set the `fig.path` to `{{ site.url }}/images/exploring-the-cars-dataset-` as a global chunk option:


```r
knitr::opts_chunk$set(fig.path='{{ site.url }}/images/exploring-the-cars-dataset-')
```

This will name any generated `.png` files `exploring-the-cars-dataset-{chunklabel}`, where the `chunklabel` is whatever you labeled the R chunk that generated the plot. Read more about chunk options [here](http://yihui.name/knitr/options/). Getting these `.png` files into the correct directory is explained in the next step and script.

Convert the `.Rmd` file to a `.md` file with `knitr`. The following script, which I've named `r2jekyll.R` and placed in the `_drafts` directory, takes a specificied `.Rmd` file sitting in the `_drafts` directory and places a knitted `.md` file in the `_posts` directory with the current date appended to the front (this is the Jekyll standard for naming posts). It also moves any `.png` files that were generated to the `images` directory.


```r
#!/usr/bin/env Rscript
library(knitr)

# Get the filename given as an argument in the shell.
args = commandArgs(TRUE)
filename = args[1]

# Check that it's a .Rmd file.
if(!grepl(".Rmd", filename)) {
  stop("You must specify a .Rmd file.")
}

# Knit and place in _posts.
dir = paste0("../_posts/", Sys.Date(), "-")
output = paste0(dir, sub('.Rmd', '.md', filename))
knit(filename, output)

# Copy .png files to the images directory.
fromdir = "{{ site.url }}/images"
todir = "../images"

pics = list.files(fromdir, ".png")
pics = sapply(pics, function(x) paste(fromdir, x, sep="/"))
file.copy(pics, todir)
```

Be sure to make `r2jekyll.R` executable with `chmod +x r2jekyll.R`. Then, to convert my `.Rmd` file to a `.md` file and take care of any `.png` file housekeeping, I navigate to my `_drafts` directory and execute in the terminal:

```
./r2jekyll.R exploring-the-cars-dataset.Rmd
```

If you find the `"{{ site.url }}"` directory within the `_drafts` directory terribly unsightly, you can add the following line to the above script:


```r
unlink("{{ site.url }}", recursive=T)
```

Preview your post with `jekyll serve`.

```
jekyll serve
```

This allows me to open my browser and preview the post at `http://localhost:4000` before pushing it to my repository.

See the code for this example blog [here](https://github.com/nicolewhite/r-knitr-jekyll/tree/gh-pages).

{% endraw %}
