FROM osgeo/gdal:3.2.0 AS osgeo

# FIXME: likely incorrect solution to ld not finding lproj at cartopy build
COPY --from=osgeo/proj:6.3.2 /usr/lib/libproj* /usr/lib/
COPY --from=osgeo/proj:6.3.2 /usr/include/ /usr/include/

ENV DEBIAN_FRONTEND noninteractive

RUN apt-get update && \
    apt-get install -yq --no-install-recommends \
        build-essential \
    	libgeos-dev \
    	python3-pip \
    	python3-dev

RUN python -m pip wheel --wheel-dir=/wheels \
    cartopy \
    psutil


FROM osgeo/gdal:3.2.0 AS jupyterlab

ENV DEBIAN_FRONTEND noninteractive

RUN curl -sL https://deb.nodesource.com/setup_lts.x | bash -

RUN apt-get update && \
    apt-get install -yq --no-install-recommends \
        nodejs \
    	python3-pip

RUN python -m pip install \
    jupyterlab \
    jupyterlab-git

RUN jupyter labextension disable @jupyterlab/extensionmanager-extension && \
    jupyter labextension install \
        @jupyter-widgets/jupyterlab-manager \
    	@jupyter-widgets/jupyterlab-sidecar \
    	@jupyterlab/geojson-extension \
    	@pyviz/jupyterlab_pyviz


FROM osgeo/gdal:3.2.0

COPY --from=osgeo /wheels /wheels
# FIXME: kinda think i'm gonna need the next line, but cartopy does import without
# COPY --from=osgeo /usr/lib/libproj* /usr/lib/

ENV DEBIAN_FRONTEND noninteractive

RUN apt-get update && \
    apt-get install -yq --no-install-recommends \
        python3-pip \
        git && \
    rm -rf /var/lib/apt/lists/*

RUN python -m pip install --no-index --find-links=/wheels \
    cartopy \
    psutil

RUN python -m pip install \
    jupyterlab \
    jupyterlab-git
    
RUN python -m pip install \
    bokeh \
    datashader \
    geopandas \
    geoviews \
    hvplot \
    panel \
    xarray

RUN rm -rf /wheels

COPY --from=jupyterlab /usr/local/share/jupyter/ /usr/local/share/jupyter/
COPY --from=jupyterlab /usr/local/etc/jupyter/ /usr/local/etc/jupyter/

# TODO
#  - tini init
