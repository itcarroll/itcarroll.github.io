ARG PYTHON_VERSION=3.10
ARG PROJ_VERSION=8.1.1
ARG GDAL_VERSION=3.4.1
ARG VENV=/opt/venv

FROM python:${PYTHON_VERSION}-slim AS base

ARG DEBIAN_FRONTEND=noninteractive
RUN apt-get update && \
    apt-get -qq install --no-install-recommends \
        curl \
	libgeos-3.9.0 \
	libgeos-c1v5 \
	libhdf5-103 \
        libhdf5-cpp-103 \  
	libnetcdf-c++4-1 \
	libnetcdf18 \
	libtiff5 \
	libxml2 \
	sqlite3 \
	udunits-bin && \
    rm -rf /var/lib/apt/lists/*

ARG VENV
ARG PIPX_BIN_DIR=/usr/local/bin
RUN python -m venv ${VENV} && \
    ${VENV}/bin/pip install pipx && \
    ${VENV}/bin/pipx install poetry

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
	pkg-config

ARG VENV
# FIXME numpy constraint due to numba, what's up with that?
RUN ${VENV}/bin/pip install "numpy<1.22" wheel

FROM builder AS proj

ARG PROJ_VERSION
RUN curl -sL https://github.com/OSGeo/PROJ/archive/refs/tags/${PROJ_VERSION}.tar.gz | \
        tar -xz --strip-components=1 && \
    ./autogen.sh && \
    ./configure --with-curl=/usr/bin/curl-config && \
    make && \
    make install

WORKDIR /
RUN tar -c -fproj.tar \
    	usr/local/lib/libproj.so* \
	usr/local/lib/pkgconfig/proj.pc \
	usr/local/share/proj \
	usr/local/bin

FROM builder AS gdal

COPY --from=proj /usr/local/include /usr/local/include/
COPY --from=proj /proj.tar /
RUN tar --skip-old-files -x -f/proj.tar -C / && rm /proj.tar && \
    ldconfig

ARG GDAL_VERSION
RUN curl -sSL https://github.com/OSGeo/gdal/archive/refs/tags/v${GDAL_VERSION}.tar.gz | \
        tar -xz --strip-components=1 && \
    cd gdal && \
    ./autogen.sh && \
    ./configure && \
    make && \
    make install

WORKDIR /
RUN tar -c -fgdal.tar \
    	usr/local/lib/pkgconfig/gdal.pc \
	usr/local/lib/gdalplugins \
    	usr/local/lib/libgdal.so* \
	usr/local/share/gdal \
	usr/local/bin

FROM builder AS wheels

COPY --from=proj /usr/local/include /usr/local/include/
COPY --from=proj /proj.tar /
RUN tar --skip-old-files -x -f/proj.tar -C / && rm /proj.tar && \
    ldconfig

COPY --from=gdal /usr/local/include /usr/local/include/
COPY --from=gdal /gdal.tar /
RUN tar --skip-old-files -x -f/gdal.tar -C / && rm /gdal.tar && \
    ldconfig

ARG VENV
COPY pyproject.toml ./
RUN sed -e "/\.dependencies/a python = \"$(${VENV}/bin/python --version | cut -c8-)\"" -i pyproject.toml
RUN poetry config virtualenvs.create false && \
    poetry add --lock $(${VENV}/bin/pip freeze) && \
    poetry add --lock gdal==$(gdal-config --version) && \
    poetry export --without-hashes --output=requirements.txt

ARG VENV
RUN ${VENV}/bin/pip wheel --wheel-dir=wheels -rrequirements.txt \
        --no-binary=cartopy \
	--no-binary=gdal \
        --no-binary=netCDF4 \
	--no-binary=shapely

RUN tar -c -f/wheels.tar wheels requirements.txt

FROM base AS jupyter

COPY --from=proj /proj.tar /
RUN tar --skip-old-files -x -fproj.tar -C / && rm /proj.tar && \
    ldconfig

COPY --from=gdal /gdal.tar /
RUN tar --skip-old-files -x -fgdal.tar -C / && rm /gdal.tar && \
    ldconfig

ARG DEBIAN_FRONTEND=noninteractive
RUN apt-get update && \
    apt-get -qq install --no-install-recommends \
        git \
	pandoc \
	ssh-client && \
    rm -rf /var/lib/apt/lists/*

COPY --from=wheels /wheels.tar /
ARG VENV
RUN tar --skip-old-files -x -f/wheels.tar && rm /wheels.tar && \
    ${VENV}/bin/pip install --no-index --find-links=wheels --no-deps -rrequirements.txt && \
    rm -rf wheels requirements.txt

# FIXME remove stage after nbdime#602 is fixed
ARG VENV
RUN ${VENV}/bin/pip install --no-deps colorama

ARG VENV
RUN ${VENV}/bin/jupyter labextension disable @jupyterlab/extensionmanager-extension

#ARG VENV
#ENTRYPOINT ["source", "${VENV}/bin/activate", "&&"]

# TODO RUN projsync -q --system-directory --all
# FIXME GDAL VENV numpy and python-bindings
# TODO a tensorflow venv using python 3.9