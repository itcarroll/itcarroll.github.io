FROM osgeo/gdal:ubuntu-full-3.2.1 AS gdal

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && \
    apt-get install -yq --no-install-recommends \
    	python3-pip

RUN python -m pip install -U \
        setuptools

FROM gdal AS patch

RUN apt-get install -yq --no-install-recommends \
        build-essential \
    	libgeos-dev \
    	python3-dev

# FIXME: why necessary? why doesn't proj installation do this?
RUN ln -s libproj.so.19 /usr/local/lib/libproj.so

RUN python -m pip wheel --wheel-dir=/wheels \
    cartopy \
    psutil

FROM gdal AS jupyter

RUN apt-get install -yq --no-install-recommends \
       git && \
   rm -rf /var/lib/apt/lists/*

COPY --from=patch /wheels /wheels

RUN python -m pip install --no-index --find-links=/wheels \
        cartopy \
        psutil && \
    rm -rf /wheels

RUN python -m pip install \
	bokeh \
	datashader \
	geopandas \
	geoviews \
	hvplot \
        ipywidgets \
        jupyterlab \
        jupyterlab-geojson \
        jupyterlab-git \
	matplotlib \
	netcdf4 \
	panel \
        sidecar \
	statsmodels \
	sympy \
	tensorflow \
	xarray \
	zarr

RUN jupyter labextension disable @jupyterlab/extensionmanager-extension

FROM jupyter

ARG JOVYAN

ENV USER=${JOVYAN}

RUN useradd -ms /bin/bash ${USER}

USER ${USER}
