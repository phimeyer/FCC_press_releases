---
title: "Scraping Press Release Texts from the German Federal Constitutional Court"
author: "Philipp Meyer"
date: "11/07/2019"
output: 
html_document
---

## Disclaimer

This project is still a "work in progress". Unfortunately, not all steps are very efficient. Please contact me if you have ideas for a smoother workflow and also befor using it (p.meyer@ipw.uni-hannover.de)  

## Description

The target website is https://www.bundesverfassungsgericht.de/DE/Presse/Pressemitteilungen/pressemitteilungen_node.html. 

The Court also provides english translations of selected press releases. (https://www.bundesverfassungsgericht.de/EN/Presse/Pressemitteilungen/pressemitteilungen_node.html). Since the basic structure of the english version is the same, this script should also work on the english text.  

We start by loading some relevant packages.

```{r}
# packages
library(rvest)
library(magrittr)
library(stringr)

```

## Start

The court press release texts are stored in seperate pages with 10 decisions on one page. We need to set a factor with the range of the total page numbers. Unfortunatly, this needs to be done manually.

```{r}
pages <- c(1:123)
```

Next we text whether the identified html_node (extracted with the help of the selector gadget program) works. Each press release text is stored in a seperate link (e.g. https://www.bundesverfassungsgericht.de/SharedDocs/Pressemitteilungen/DE/2019/bvg19-045.html).

```{r}
html <- read_html("http://www.bundesverfassungsgericht.de/DE/Presse/Pressemitteilungen/pressemitteilungen_node.html?cms_gtp=5399872_list%253D1")
html %>% html_nodes("td") %>% html_nodes("a") %>% html_attr("href")
```

Since it works fine, we select all individual court decision links.

```{r}
links <- c()
for (i in 1:length(pages)){
  html <- read_html(paste("http://www.bundesverfassungsgericht.de/DE/Presse/Pressemitteilungen/pressemitteilungen_node.html",i, sep = ""))
  href <- html %>% html_nodes("td") %>% html_nodes("a") %>% html_attr("href")
  links <- c(links, href)
  }

# select the press releases' links
presslinks <- links[grep("^SharedDocs/Pressemitteilungen/",links)] %>% strsplit(";") %>% sapply("[", 1)

# saving the list for backup
write.csv(presslinks, file = "presslink.txt", row.names = FALSE)
```

Now we extract basic information to create filenames for each press release text. Since the Court provides general information on each press release in its links (bvg98-013), we extract these information to create the filenames. The press release information are the year of the press release and the German acronym of the court (bvg98) and the runing number of the press release (013).

```{r}
filenames <- presslinks %>% strsplit("DE/") %>% sapply("[", 2) %>% 
  strsplit("/") %>% sapply("[", 2) %>% 
  strsplit("\\.") %>% sapply("[", 1)
```

Finally we can extract the full texts of each press release and store them as as .txt-file.

```{r}
# Testing for one text
 press <- read_html("http://www.bundesverfassungsgericht.de/SharedDocs/Pressemitteilungen/DE/2017/bvg17-046.html")
 presstext <- press %>% html_nodes("#wrapperContent") %>% html_text()
 cat(presstext, file="testing.txt", sep="", append=FALSE)
 
# Scrape all texts
 for (i in 1:length(presslinks)) {
   press <- read_html(paste("http://www.bundesverfassungsgericht.de/", presslinks[i], sep = "", Sys.sleep(sample(seq(1, 3, by=0.005), 1))))
  press %>% html_nodes("#wrapperContent") %>% html_text() %>% 
  cat(file=paste(filenames[i], ".txt", sep = ""), sep="", append=FALSE)
  }
```
