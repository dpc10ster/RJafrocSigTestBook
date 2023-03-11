--- 
title: "The RJafroc Significance Testing Book"
author: "Dev P. Chakraborty, PhD"
date: "2023-03-11"
site: bookdown::bookdown_site
output: bookdown::html_document
documentclass: book
bibliography: [packages.bib, myRefs.bib]
biblio-style: apalike
link-citations: yes
github-repo: dpc10ster/RJafrocSigTestBook
description: "Artificial intelligence and observer performance book based on RJafroc."
---





# Preface {-}

TBA


## Rationale and Organization

* Intended as an online update to my print book [@chakraborty2017observer].
* All references in this book to `RJafroc` refer to the R package with that name (case sensitive) [@R-RJafroc].
* Since its publication in 2017 `RJafroc`, on which the `R` code examples in the print book depend, has evolved considerably causing many of the examples to "break" if one uses the most current version of `RJafroc`. The code will still run if one uses [`RJafroc` 0.0.1](https://cran.r-project.org/src/contrib/Archive/RJafroc/) but this is inconvenient and misses out on many of the software improvements made since the print book appeared.
* This gives me the opportunity to update the print book.
* The online book has been divided into 3 books.
    + The [RJafrocQuickStartBook](https://dpc10ster.github.io/RJafrocQuickStart/) book.
    + The [RJafrocRocBook](https://dpc10ster.github.io/RJafrocRocBook/) book.
    + **This book:** [RJafrocSigTestBook](https://dpc10ster.github.io/RJafrocSigTestBook/). 
    + The [RJafrocFrocBook](https://dpc10ster.github.io/RJafrocFrocBook/) book.


## TBA Acknowledgements

Dr. Xuetong Zhai

Dr. Peter Phillips

Online Latex Editor [at this site](https://latexeditor.lagrida.com/) 

Dataset contributors



## Temporary comments

This is intended to allow successful builds when a needed file is not in the build. These are indicated by, for example:

Chapter `TempComment \@ref(proper-roc-models)`

Fix these on final release.

