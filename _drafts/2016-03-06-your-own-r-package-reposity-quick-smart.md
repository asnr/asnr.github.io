---
layout:     post
title:      Your own R package repository, quick smart
date:       2016-03-06
summary:    You'll never have to `?install.packages` again.
categories: [R-at-work, R, windows]
---

Heyo! This is the first of what will be a short series of posts on how to develop and distribute R packages within a workplace.

This approach assumes all your colleagues are running Windows

This approach taken here is opinionated and is assumes that all your colleagues are running Windows.

This setup has been refined this workflow over a year of trawling mailing lists, stackoverflow.

I can't guarantee this is the best setup for everyone, but it 

This I've spent the past year or so trawling through 

This setup 

If you're writing R for as part of your day job, you'll inevitably find yourself wrapping commonly used code into a package, and if you're working in a team, you'll... what? Drop the source code into a directory everyone has access to and tell the team to `install.packages("file://some_dir/my_package", repos = NULL, type = "source")`? Point them to a local, github or bitbucket repo and tell them to `devtools::install_local`,  `devtools::install_github` or `devtools::install_bitbucket` respectively?

Let's set up a CRAN-like package repository to host your (or your company's) private packages. We might come in expecting that

We don't want to spend time fiddling with servers or networks, so we're going to host our repository on a 

Wouldn't it be nice if everyone could just `install.packages('my_package')`?

Make a directory, call it anything. (I called it `R`. I'm original like that.)

Fill it with empty directories, so that it looks like this (again, the top level directory can be called whatever you want):

```
R
|-- bin
|   `-- windows
|       `-- contrib
|           `-- 3.2   <- this is the R major.minor version number
`-- src
    `-- contrib
```

When you're ready to deploy a package, open up R and run:

```r
R_min = sub("([0-9]+)\\.[0-9]+", "\\1", R.Version()$minor)
R_ver = paste0(R.Version()$major, ".", R_min)

# Make sure the working directory is the root directory of the package source

# Build and deploy binary version of the package
devtools::build(
    path = file.path("path/to/repository", "bin/windows/contrib", R_ver),
    binary = TRUE
)
# Build and deploy source version of the package
devtools::build(path = file.path("path/to/repository", "src/contrib"))
```

At this point the repository will contain the the new package `.tar.gz` and `.zip` bundles, but calling `install.package()` will not find them, as `install.package()` first retrieves the repository's `PACKAGES` file to get the list of available packages, and the new package will not be on this list. To update the `PACKAGES` file, run

```r
# see previous code block for definition of `R_ver` variable
tools::write_PACKAGES(file.path("path/to/repository", "bin/windows/contrib",
                                R_ver),  
                      type = "win.binary", verbose = TRUE)
tools::write_PACKAGES(file.path("path/to/repository", "src/contrib"),
                      type = "source", verbose = TRUE)
```

Once you've been doing this for a while, your package repository will look something like this:

```
R
|-- bin
|   `-- windows
|       `-- contrib
|           |-- 3.1
|           |   |-- mypackage_0.1.0.zip
|           |   |-- PACKAGES
|           |   `-- PACKAGES.gz
|           `-- 3.2
|               |-- mypackage_0.1.1.zip
|               |-- mypackage_0.2.0.zip
|               |-- mypackage_0.2.1.zip
|               |-- PACKAGES
|               `-- PACKAGES.gz
`-- src
    `-- contrib
        |-- mypackage_0.1.0.tar.gz
        |-- mypackage_0.1.1.tar.gz
        |-- mypackage_0.2.0.tar.gz
        |-- mypackage_0.2.1.tar.gz
        |-- PACKAGES
        `-- PACKAGES.gz
```

Analysts should add user-level `.Rprofile` (preferably in their home folder, not in their program install. What's your home directory you ask? I did too, and R has your answer: just evaluate `path.expand('~')`):

```r
# We use local() to avoid name collisions with the variable `r`
local({r <- getOption("repos")
       r["my_repo"] <- "file:S:/R/"
       options(repos = r)})
```

Upon restarting R, analysts can then run

```r
install.packages("my_package")
```






build script:
```posh
"C:\Program Files\R\R-3.2.1\bin\x64\Rscript.exe" C:\git_repos\retror\build.R
```

build.R:
```r
# Determine R's major and minor version. Note R.Version()$minor returns minor
# and patch versions together.
R_maj = R.Version()$major
minor_patch_sep_idx = unlist(gregexpr("\\.", R.Version()$minor))
R_minor = substr(R.Version()$minor, 1, minor_patch_sep_idx - 1)
R_ver = paste0(R_maj, ".", R_minor)

pkg_src_root_dir = "C:/my_package"  # directory of source code
pkg_repo_dir = "S:/R"  # where S is shared drive

binary_dir = file.path(pkg_repo_dir, "\\bin\\windows\\contrib", R_ver)
devtools::build(pkg = pkg_src_root_dir, path = binary_dir, binary = TRUE)

src_dir = file.path(pkg_repo_dir, "src\\contrib")
devtools::build(pkg = pkg_src_root_dir, path = src_dir)
```


repo refresh script (rewrite as big R script? Hmm... extra level of indirection...) `build_PACKAGES.bat`:
```bat
SET latest_RScript="C:\Program Files\R\R-3.2.0\bin\x64\Rscript.exe"

S:
cd "\R\src\contrib"
%latest_RScript% -e tools::write_PACKAGES(\".\",type=\"source\",verbose=TRUE)

cd "\R\bin\windows\contrib\3.2"
%latest_RScript% -e tools::write_PACKAGES(\".\",type=\"win.binary\",verbose=TRUE)

cd "\R\bin\windows\contrib\3.1"
%latest_RScript% -e tools::write_PACKAGES(\".\",type=\"win.binary\",verbose=TRUE)
```


Final state:

```
R
|-- bin
|   `-- windows
|       `-- contrib
|           |-- 3.1
|           |   |-- PACKAGES
|           |   |-- PACKAGES.gz
|           |   `-- mypackage_0.1.0.zip
|           `-- 3.2
|               |-- PACKAGES
|               |-- PACKAGES.gz
|               |-- mypackage_0...zip
|               |-- mypackage_0...zip
|               `-- mypackage_0...zip
`-- src
    `-- contrib
        |-- PACKAGES
        |-- PACKAGES.gz
        |-- mypackage_0...tar.gz
        |-- mypackage_0...tar.gz
        |-- mypackage_0...tar.gz
        `-- mypackage_0...tar.gz
```

what analysts put on their user-level `.Rprofile` (preferably in their home folder, not in their program install):
```r
# Add the R package repository
local({r <- getOption("repos")
       r["my_repo"] <- "file:S:/R/"
       options(repos=r)})
```

then install as per normal:
```r
install.packages("my_package")
```
