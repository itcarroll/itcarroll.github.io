FROM debian:testing

ENV DEBIAN_FRONTEND noninteractive

ADD https://download2.rstudio.org/server/bionic/amd64/rstudio-server-1.4.1106-amd64.deb /

RUN apt-get update && \
    apt-get install -yq --no-install-recommends \
	git \
	gdebi-core \
	libopenblas-base \
	lsb-release \
	locales \
	make \
	rsync \
	ruby \
	ssh-client \
	xz-utils && \
    localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8

ENV LANG=en_US.UTF-8

RUN apt-get update && \
    apt-get install -yq --no-install-recommends \
    	r-base

RUN gdebi -nq rstudio-server-*-amd64.deb && \
    rm rstudio-server-*-amd64.deb

RUN apt-get install -yq --no-install-recommends \
#        r-cran-gstat \
#        r-cran-mapview \
        r-cran-reticulate \
	r-cran-rgdal \
        r-cran-sf \
        r-cran-sp \
        r-cran-spdep \
	r-cran-spatialreg \
	r-cran-stars \
	r-cran-tidyverse && \
    apt-get clean

COPY disable_auth_rserver.conf /etc/rstudio/rserver.conf

ARG RSTUDIO

ENV USER=${RSTUDIO}

RUN useradd -ms /bin/bash ${USER}
