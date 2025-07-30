FROM ghcr.io/osgeo/gdal:ubuntu-small-latest

ARG SSH_PRIVATE_KEY

RUN apt-get update && apt-get install -y \
    curl \
    nano \
    git \
    wget \
    openjdk-21-jre \
    perl \
    pipx \
    gdal-bin \
    python3-gdal \
    python3-pip \
    python3-venv \
    python3-dev \
    openssh-client \
 && pipx ensurepath \
 && pipx install poetry

ENV PYTHONPATH="/usr/lib/python3/dist-packages:$PYTHONPATH"
ENV POETRY_HOME="/root/.local"
ENV PATH=$POETRY_HOME/bin:$PATH

# Create the project and constrain Python version to fit rdflib and gdal
WORKDIR /app
RUN /root/.local/bin/poetry init --name app \
    --dependency fiona \
    --dependency rdflib \
    --dependency pyproj \
    --dependency pyogrio \
    --dependency geopandas \
    --dependency shapely \
    --dependency lxml \
    --python ">=3.12,<4.0.0" \
    --no-interaction

RUN /root/.local/bin/poetry install --no-root

# Clone Tegel
RUN git clone https://github.com/ogcincubator/cityrdf.git

# Apache Jena install
ENV JENA_VERSION 5.1.0
ENV JENA_HOME /jena
ENV PATH="$PATH:$JENA_HOME/bin"

RUN cd /tmp && \
    wget https://archive.apache.org/dist/jena/binaries/apache-jena-${JENA_VERSION}.tar.gz && \
    tar -xzf apache-jena-${JENA_VERSION}.tar.gz && \
    mv apache-jena-${JENA_VERSION} /jena && \
    rm apache-jena-${JENA_VERSION}.tar.gz && \
    rm -rf /jena/*javadoc* /jena/*src* /jena/bat

# XSPARQL JAR and wrapper
RUN mkdir -p /rdf && \
    wget -O /rdf/xsparql-cli-jar-with-dependencies.jar \
        https://sourceforge.net/projects/xsparql/files/xsparql/20140909/xsparql-cli-jar-with-dependencies.jar/download && \
    echo '#!/bin/sh\njava -Xmx6G -Dfile.encoding=UTF-8 -jar /rdf/xsparql-cli-jar-with-dependencies.jar "$@"' \
        > /usr/local/bin/xsparql && chmod +x /usr/local/bin/xsparql

CMD ["/bin/bash"]

# Commands to build and run the docker image
# Build
# At the host machine: 
# docker build -t cityrdf .
# Run
# docker run -v ~/work/gml3/cityrdf:/app/cityrdf -ti cityrdf