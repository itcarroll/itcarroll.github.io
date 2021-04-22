FROM osgeo/gdal:3.2.0 AS osgeo

ENV DEBIAN_FRONTEND noninteractive

ADD https://download2.rstudio.org/server/bionic/amd64/rstudio-server-1.4.1106-amd64.deb /

RUN apt-get update && \
    apt-get install -yq --no-install-recommends \
        dirmngr \
	git \
	gdebi-core \
	gpg-agent \
	make \
	rsync \
	ruby \
	ssh-client \
        software-properties-common && \
    apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E298A3A825C0D65DFD57CBB651716619E084DAB9 && \
    add-apt-repository "deb https://cloud.r-project.org/bin/linux/ubuntu $(lsb_release -cs)-cran40/" && \
    add-apt-repository ppa:c2d4u.team/c2d4u4.0+

RUN apt-get update && \
    apt-get install -yq --no-install-recommends \
    	r-base \
	r-cran-tidyverse

RUN gdebi -nq rstudio-server-*-amd64.deb && \
    rm rstudio-server-*-amd64.deb


RUN useradd -ms /bin/bash rstudio && \
    echo "rstudio:rstudio" | chpasswd

## COPY disable_auth_rserver.conf /etc/rstudio/rserver.conf

RUN apt-get install -yq --no-install-recommends \
        r-cran-gstat \
        r-cran-mapview \
        r-cran-reticulate \
	r-cran-rgdal \
        r-cran-sf \
        r-cran-sp \
        r-cran-spdep \
	r-cran-spatialreg \
	r-cran-stars && \
    apt-get clean

## FIXME: does R actually use the system GDAL?
