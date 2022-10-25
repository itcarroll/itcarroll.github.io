# Contents

1. Docker Containers
1. GitHub Page
1. Jupyter Notebooks

## Docker Containers

The `docker` sub-directory contains configuration for dockerized
(i.e. having easily replicated environments) services that I use for
multiple projects. The `docker-compose.yaml` file defines three
[services].

1. `gh-pages` builds static websites using the same software versions
   used by [GitHub Pages] builds
1. `conda` is a anaconda3 container that launches a Jupyter server on
   startup.
1. `jupyter` launches a Jupyter server with a custom suite of packages,
   defined by the `Dockerfile` in the `jupyter` folder, for data science
   projects
1. `rstudio` launches a WIP RStudio Server

Each service runs with the `$HOME` directory mounted to `/root` and
the `$PWD` mounted to `/src`. I recommend setting the `COMPOSE_FILE`
variable in your shell environment to the `docker-compose.yaml` file's
path, so that you may easily launch a service from the directory
containing data you want accessible in `/src`.

## GitHub Page

The `docs` sub-directory contains the files for the static website
published at https://itcarroll.github.io, my online CV/resume.

## Jupyter Notebooks

The `notebooks` folder contains the occasional Jupyter notebook that
I'm interested in publicly sharing.

[services]: https://docs.docker.com/compose/compose-file/compose-file-v3/#service-configuration-reference
[GitHub Pages]: https://pages.github.com/
