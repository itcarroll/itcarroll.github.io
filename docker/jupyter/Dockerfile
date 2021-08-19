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

FROM gdal AS openrefine

# TODO add openrefine

FROM openrefine AS jupyterlab

RUN python -m pip install \
    jupyterlab==2.* \
    jupyterlab-git

FROM jupyterlab AS jupyterlab-build

RUN curl -sL https://deb.nodesource.com/setup_lts.x | bash -

RUN apt-get install -yq --no-install-recommends \
        nodejs

RUN jupyter labextension disable @jupyterlab/extensionmanager-extension && \
    jupyter labextension install \
        @jupyter-widgets/jupyterlab-manager \
    	@jupyter-widgets/jupyterlab-sidecar \
    	@jupyterlab/geojson-extension \
    	@pyviz/jupyterlab_pyviz

FROM jupyterlab AS jupyter

COPY --from=jupyterlab-build /usr/local/share/jupyter/ /usr/local/share/jupyter/

COPY --from=jupyterlab-build /usr/local/etc/jupyter/ /usr/local/etc/jupyter/

RUN apt-get install -yq --no-install-recommends \
        git && \
    rm -rf /var/lib/apt/lists/*

COPY --from=patch /wheels /wheels

RUN python -m pip install --no-index --find-links=/wheels \
        cartopy \
        psutil && \
    rm -rf /wheels

RUN python -m pip install \
        ipywidgets \
	bokeh \
	datashader \
	geopandas \
	geoviews \
	hvplot \
	matplotlib \
	netcdf4 \
	panel \
	statsmodels \
	sympy \
	xarray \
	zarr

FROM jupyter

ARG JOVYAN

ENV USER=${JOVYAN}

RUN useradd -ms /bin/bash ${USER}

USER ${USER}
