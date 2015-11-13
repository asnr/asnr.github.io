---
layout:     post
title:      Your own R package repository, quick smart
date:       2015-11-13
summary:    
categories: [R, windows]
---


Make a directory, call it anything. (I called it `R`.)

Init state:

```
R
|-- bin
|   `-- windows
|       `-- contrib
|           `-- 3.2
`-- src
    `-- contrib
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
