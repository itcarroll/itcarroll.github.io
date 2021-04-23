FROM continuumio/anaconda3:2020.11

ARG JOVYAN

ENV USER=${JOVYAN}

RUN useradd -ms /bin/bash ${USER}

USER ${USER}
