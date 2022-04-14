ARG OPT=/opt
ARG PYTHON_VERSION=3.10
ARG PROJ_VERSION=8.1.1
ARG GDAL_VERSION=3.4.1


FROM python:$PYTHON_VERSION-slim AS base
ARG OPT
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

RUN python -m venv "$OPT/venv"

ENV PATH="$OPT/venv/bin:$OPT/bin:$PATH"

RUN pip install pipx && \
    PIPX_BIN_DIR="$OPT/bin" PIPX_HOME="$OPT/pipx" pipx install poetry


FROM base AS builder
ARG DEBIAN_FRONTEND=noninteractive

WORKDIR /src

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

# FIXME numpy constraint due to numba, what's up with that?
# FIXME but I don't need wheel in the final image
RUN pip install "numpy<1.22" wheel


FROM builder AS proj
ARG PROJ_VERSION

RUN curl -sL "https://github.com/OSGeo/PROJ/archive/refs/tags/$PROJ_VERSION.tar.gz" | \
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

# TODO RUN projsync -q --system-directory --all


FROM builder AS gdal
ARG GDAL_VERSION

COPY --from=proj /usr/local/include /usr/local/include/

COPY --from=proj /proj.tar /

RUN tar --skip-old-files -x -f/proj.tar -C / && rm /proj.tar && \
    ldconfig

RUN curl -sSL "https://github.com/OSGeo/gdal/archive/refs/tags/v$GDAL_VERSION.tar.gz" | \
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
ARG PYTHON_VERSION

COPY --from=proj /usr/local/include /usr/local/include/

COPY --from=proj /proj.tar /

RUN tar --skip-old-files -x -f/proj.tar -C / && rm /proj.tar && \
    ldconfig

COPY --from=gdal /usr/local/include /usr/local/include/

COPY --from=gdal /gdal.tar /

RUN tar --skip-old-files -x -f/gdal.tar -C / && rm /gdal.tar && \
    ldconfig

COPY pyproject.toml ./

RUN poetry config virtualenvs.create false && \
    poetry add --lock $(pip freeze) && \
    poetry add --lock gdal==$(gdal-config --version) && \
    poetry export --without-hashes --output=requirements.txt

RUN pip install --no-deps -rrequirements.txt \
        --no-binary=cartopy \
	--no-binary=gdal \
        --no-binary=netCDF4 \
	--no-binary=shapely

# FIXME remove stage after nbdime#602 is fixed
RUN pip install --no-deps colorama


FROM base AS jupyter
ARG DEBIAN_FRONTEND=noninteractive

COPY --from=proj /proj.tar /

RUN tar --skip-old-files -x -fproj.tar -C / && rm /proj.tar && \
    ldconfig

COPY --from=gdal /gdal.tar /

RUN tar --skip-old-files -x -fgdal.tar -C / && rm /gdal.tar && \
    ldconfig

COPY --from=wheels $OPT $OPT

RUN apt-get update && \
    apt-get -qq install --no-install-recommends \
        git \
	pandoc \
	ssh-client && \
    rm -rf /var/lib/apt/lists/*

RUN jupyter labextension disable @jupyterlab/extensionmanager-extension

FROM jupyter
ARG JOVYAN

RUN useradd -ms /bin/bash ${JOVYAN}

USER ${JOVYAN}
