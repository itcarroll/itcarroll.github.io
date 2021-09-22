FROM osgeo/gdal:ubuntu-full-3.3.2 AS gdal

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
RUN ln -s libproj.so.22 /usr/local/lib/libproj.so

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

RUN python -m pip install "poetry==1.1.10"

RUN poetry init --quiet --no-interaction && \
    poetry config --local virtualenvs.create false

RUN poetry add \
	datashader \
	geopandas \
	geoviews \
	hvplot \
        ipywidgets \
        jupyterlab \
        jupyterlab-geojson \
        jupyterlab-git \
	matplotlib \
	memory_profiler \
	netcdf4 \
        sidecar \
	statsmodels \
	sympy \
	tensorflow \
	xarray \
	zarr

RUN jupyter labextension disable @jupyterlab/extensionmanager-extension

FROM jupyter
