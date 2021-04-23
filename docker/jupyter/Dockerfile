FROM osgeo/gdal:ubuntu-full-3.2.1 AS build_0

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && \
    apt-get install -yq --no-install-recommends \
    	python3-pip

FROM build_0 AS build_1

RUN apt-get install -yq --no-install-recommends \
        build-essential \
    	libgeos-dev \
    	python3-dev

# FIXME: why necessary? why doesn't proj installation do this?
RUN ln -s libproj.so.19 /usr/local/lib/libproj.so

RUN python -m pip wheel --wheel-dir=/wheels \
    cartopy \
    psutil

FROM build_0 AS build_2

RUN python -m pip install \
    jupyterlab==2.* \
    jupyterlab-git

FROM build_2 AS build_3

RUN curl -sL https://deb.nodesource.com/setup_lts.x | bash -

RUN apt-get install -yq --no-install-recommends \
        nodejs

RUN jupyter labextension disable @jupyterlab/extensionmanager-extension && \
    jupyter labextension install \
        @jupyter-widgets/jupyterlab-manager \
    	@jupyter-widgets/jupyterlab-sidecar \
    	@jupyterlab/geojson-extension \
    	@pyviz/jupyterlab_pyviz

FROM build_2

COPY --from=build_3 /usr/local/share/jupyter/ /usr/local/share/jupyter/

COPY --from=build_3 /usr/local/etc/jupyter/ /usr/local/etc/jupyter/

RUN apt-get install -yq --no-install-recommends \
        git && \
    rm -rf /var/lib/apt/lists/*

COPY --from=build_1 /wheels /wheels

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
	panel \
	statsmodels \
	sympy \
	xarray \
	zarr

ARG JOVYAN

ENV USER=${JOVYAN}

RUN useradd -ms /bin/bash ${USER}

USER ${USER}
