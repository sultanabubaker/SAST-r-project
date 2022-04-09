# data.table continuous integration and deployment

On each Pull Request opened in GitHub we run Travis CI and Appveyor to provide prompt feedback about the status of PR. Our main CI pipeline runs on GitLab CI. GitLab repository automatically mirrors our GitHub repository and runs pipeline on `master` branch. It tests more environments and different configurations. It publish variety of artifacts.

## Environments

### [GitLab CI](./../.gitlab-ci.yml)

Test jobs:
- `test-rel-lin` - `r-release` on Linux, most comprehensive test environment, `-O3 -flto -fno-common -Wunused-result`, extra check for no compilation warnings, includes testing [_with other packages_](./../inst/tests/other.Rraw)
- `test-rel-cran-lin` - `--as-cran` on Linux, `-g0`, extra check for final status of `R CMD check` where we allow one NOTE (_size of tarball_).
- `test-dev-cran-lin` - `r-devel` and `--as-cran` on Linux, `--with-recommended-packages --enable-strict-barrier --disable-long-double`, tests for compilation warnings in pkg install and new NOTEs/Warnings in pkg check, and because it is R-devel it is marked as allow_failure
- `test-rel-vanilla-lin` - `r-release` on Linux, no suggested deps, no OpenMP, `-O0`, tracks memory usage during tests
- `test-310-cran-lin` - R 3.1.0 on Linux
- `test-344-cran-lin` - R 3.4.4 on Linux
- `test-350-cran-lin` - R 3.5.0 on Linux, no `r-recommended`
- `test-rel-win` - `r-release` on Windows
- `test-dev-win` - `r-devel` on Windows
- `test-old-win` - `r-oldrel` on Windows
- `test-rel-osx` - MacOSX build not yet deployed, see [#3326](https://github.com/Rdatatable/data.table/issues/3326) for status

Artifacts:
- [homepage](https://rdatatable.gitlab.io/data.table) - made with [pkgdown](https://github.com/r-lib/pkgdown)
- [html manual](https://rdatatable.gitlab.io/data.table/library/data.table/html/00Index.html)
- [pdf manual](https://rdatatable.gitlab.io/data.table/web/packages/data.table/data.table.pdf)
- [html vignettes](https://rdatatable.gitlab.io/data.table/library/data.table/doc/index.html)
- R packages repository for `data.table` and all _Suggests_ dependencies, url: `https://Rdatatable.gitlab.io/data.table`
  - sources
  - Windows binaries for `r-release`, `r-devel` and `r-oldrel`
- [CRAN-like homepage](https://rdatatable.gitlab.io/data.table/web/packages/data.table/index.html)
- [CRAN-like checks results](https://rdatatable.gitlab.io/data.table/web/checks/check_results_data.table.html) - note that all artifacts, including check results page, are being published only when all test jobs successfully pass, thus one will not see an _ERROR_ status there (unless error happened on a job marked as `allow_failure`).
- [docker images](https://gitlab.com/Rdatatable/data.table/container_registry) - copy/paste-able `docker pull` commands can be found at the bottom of our [CRAN-like homepage](https://rdatatable.gitlab.io/data.table/web/packages/data.table/index.html)

### [Travis CI](./../.travis.yml)

Test jobs:
- `r-release` on Linux, includes code coverage check
- _(might be disabled)_ `r-release` on OSX

Artifacts:
- R packages repository having `data.table` sources only, url: `https://Rdatatable.github.io/data.table`
- code coverage stats pushed to [codecov.io/gh/Rdatatable/data.table](https://codecov.io/gh/Rdatatable/data.table)

### [Appveyor](./../.appveyor.yml)

Test jobs:
- Windows `r-release`
- _(might be disabled)_ Windows `r-devel`

Artifacts:
- Windows `r-release` binaries accessed only via web UI

## Tools

### [`ci.R`](./ci.R)

Base R implemented helper script, [originally proposed to R](https://svn.r-project.org/R/branches/tools4pkgs/src/library/tools/R/packages.R), that ease the process of extracting dependency information from description files, also to mirror packages and their recursive dependencies from CRAN to local CRAN-like directory. It is widely used in our [GitLab CI pipeline](./../.gitlab-ci.yml).

### [`publish.R`](./publish.R)

Base R implemented helper script to orchestrate generation of most artifacts. It is being used only in [_integration_ stage in GitLab CI pipeline](./../.gitlab-ci.yml).

### [`Dockerfile.in`](./Dockerfile.in)

Template file to produce `Dockerfile` for, as of now, three docker images. Docker images are being built and published in [_deploy_ stage in GitLab CI pipeline](./../.gitlab-ci.yml).
- `r-base-dev` using `r-release`: publish docker image of `data.table` on R-release
- `r-builder` using `r-release`: publish on R-release and OS dependencies for building Rmarkdown vignettes
- `r-devel`: publish docker image of `data.table` on R-devel built with `--with-recommended-packages --enable-strict-barrier --disable-long-double`

### [`deploy.sh`](./deploy.sh)

Script used on Travis CI to publish CRAN-like repository of `data.table` sources. It publishes to `gh-pages` branch in GitHub repository. It depends on a token, which is provided based on `secure` environment variable in [.travis.yml](./../.travis.yml). It has been generated by @jangorecki.