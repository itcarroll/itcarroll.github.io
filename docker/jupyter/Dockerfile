ARG UBUNTU_VERSION=20.04
ARG PROJ_VERSION=8.1.1
ARG GDAL_VERSION=3.3.2

FROM ubuntu:${UBUNTU_VERSION} AS base

ARG DEBIAN_FRONTEND=noninteractive
RUN apt-get update && \
    apt-get -qq install --no-install-recommends \
        curl \
	libgeos-3.8.0 \
	libgeos-c1v5 \
	libhdf5-103 \
        libhdf5-cpp-103 \  
	libnetcdf-c++4 \
	libnetcdf15 \
	libtiff5 \
	libxml2 \
        python3 \
        python3-pip \
	python3-venv \
	sqlite3 \
	udunits-bin && \
    rm -rf /var/lib/apt/lists/*

ARG POETRY_HOME=/usr/local
RUN curl -sSL https://raw.githubusercontent.com/python-poetry/poetry/master/install-poetry.py | python3

FROM base AS builder

WORKDIR /src
ARG DEBIAN_FRONTEND=noninteractive
RUN apt-get update && \
    apt-get -qq install --no-install-recommends \
	automake \
        build-essential \
	ca-certificates \
	libcurl4-gnutls-dev \
	libgeos-dev \
	libhdf5-dev \ 
        libnetcdf-dev \
	libsqlite3-dev \
	libtiff5-dev \
	libtool \
	libxml2-dev \
	make \
	pkg-config \
	python3-dev

FROM builder AS proj

ARG PROJ_VERSION
RUN curl -sL https://github.com/OSGeo/PROJ/archive/refs/tags/${PROJ_VERSION}.tar.gz | \
        tar -xz --strip-components=1 && \
    ./autogen.sh && \
    ./configure --with-curl=/usr/bin/curl-config && \
    make && \
    make install

WORKDIR /
RUN tar -cf proj.tar \
    	usr/local/lib/libproj.so* \
	usr/local/lib/pkgconfig/proj.pc \
	usr/local/share/proj \
	usr/local/bin

FROM builder AS gdal

COPY --from=proj /usr/local/include /usr/local/include/
COPY --from=proj /proj.tar /
RUN tar --skip-old-files -xf /proj.tar -C / && rm /proj.tar && \
    ldconfig

ARG GDAL_VERSION
RUN curl -sL https://github.com/OSGeo/gdal/archive/refs/tags/v${GDAL_VERSION}.tar.gz | \
        tar -xz --strip-components=1 && \
    cd gdal && \
    ./configure && \
    make && \
    make install

WORKDIR /
RUN tar -cf gdal.tar \
    	usr/local/lib/libgdal.so* \
    	usr/local/lib/pkgconfig/gdal.pc \
	usr/local/lib/gdalplugins \
	usr/local/share/gdal \
	usr/local/bin

FROM builder AS wheels

COPY --from=proj /usr/local/include /usr/local/include/
COPY --from=proj /proj.tar /
RUN tar --skip-old-files -xf /proj.tar -C / && rm /proj.tar && \
    ldconfig

COPY --from=gdal /usr/local/include /usr/local/include/
COPY --from=gdal /gdal.tar /
RUN tar --skip-old-files -xf /gdal.tar -C / && rm /gdal.tar && \
    ldconfig

RUN poetry init --quiet --no-interaction && \
    poetry config --local virtualenvs.create false && \
    poetry add --lock \
        cartopy \
	datashader \
	GDAL \
	geopandas \
	geoviews \
	hvplot \
	ipywidgets \
	jupyterlab \
	jupyterlab-geojson \
	jupyterlab-git \
	memory_profiler \
	netcdf4 \
	statsmodels \
	sympy \
	tensorflow \
	xarray \
	zarr && \
    poetry export --without-hashes --output=requirements.txt

RUN pip wheel --wheel-dir=wheels -r requirements.txt \
        --no-binary cartopy \
	--no-binary GDAL \
	--no-binary shapely

RUN tar -cf /wheels.tar wheels requirements.txt

FROM base AS jupyter

COPY --from=proj /proj.tar /
RUN tar --skip-old-files -xf proj.tar -C / && rm /proj.tar && \
    ldconfig

COPY --from=gdal /gdal.tar /
RUN tar --skip-old-files -xf gdal.tar -C / && rm /gdal.tar && \
    ldconfig

COPY --from=wheels /wheels.tar /
RUN tar --skip-old-files -xf /wheels.tar && rm /wheels.tar && \
    pip install --no-index --find-links=wheels --no-deps -r requirements.txt && \
    rm -rf wheels requirements.txt

# FIXME remove next line after nbdime#602 is merged
RUN pip install --no-deps colorama

ARG DEBIAN_FRONTEND=noninteractive
RUN apt-get update && \
    apt-get -qq install --no-install-recommends \
        git \
	python-is-python3 && \
    rm -rf /var/lib/apt/lists/*

RUN jupyter labextension disable @jupyterlab/extensionmanager-extension

# TODO RUN projsync -q --system-directory --all
# TODO warning in GDAL python bindings build suggested no numpy once, and didn't fix but gone now, maybe gdal-config?
