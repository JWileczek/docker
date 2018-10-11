# base image
FROM pelias/baseimage

RUN apt-get update

# grab all of the valhalla software from ppa
RUN apt-get install -y software-properties-common python3-software-properties && \
    apt-get update

RUN add-apt-repository -r ppa:valhalla-core/valhalla && \
    apt-get update

RUN apt-get install -y cmake g++ gcc jq lcov protobuf-compiler vim-common libboost-all-dev libboost-all-dev zlib1g-dev liblz4-dev libprotobuf-dev nodejs npm

RUN apt-get install -y libgeos-dev libgeos++-dev liblua5.2-dev libspatialite-dev libsqlite3-dev lua5.2 wget

RUN if [[ $(grep -cF xenial /etc/lsb-release) > 0 ]]; then apt-get install -y libsqlite3-mod-spatialite; fi

RUN apt-get install -y python-all-dev

RUN if [[ $(grep -cF trusty /etc/lsb-release) > 0 ]]; then \
      add-apt-repository -y ppa:kevinkreiser/libsodium; \
      add-apt-repository -y ppa:kevinkreiser/libpgm; \
      add-apt-repository -y ppa:kevinkreiser/zeromq3; \
      add-apt-repository -y ppa:kevinkreiser/czmq; \
      apt-get update; fi

RUN apt-get install -y libcurl4-openssl-dev libzmq3-dev libczmq-dev

RUN mkdir -p /code/prime_server
WORKDIR /code/prime_server

RUN git clone https://github.com/kevinkreiser/prime_server.git /code/prime_server
RUN git submodule update --init --recursive
RUN ./autogen.sh
RUN ./configure
RUN make test -j8
RUN make install

# change working dir
RUN mkdir -p /code/valhalla
WORKDIR /code/valhalla

RUN git clone https://github.com/valhalla/valhalla.git /code/valhalla

RUN git submodule update --init --recursive
RUN npm install --ignore-scripts
RUN mkdir -p /code/valhalla/build
WORKDIR /code/valhalla/build

RUN cmake .. -DCMAKE_BUILD_TYPE=Release
RUN make -j$(nproc) && \
    make install


RUN mkdir -p /data/valhalla/valhalla_tiles
# generate config
RUN valhalla_build_config \
  --mjolnir-tile-dir '/data/valhalla/valhalla_tiles' \
  --mjolnir-tile-extract '/data/valhalla/valhalla_tiles.tar' \
  --mjolnir-timezone '/data/valhalla/valhalla_tiles/timezones.sqlite' \
  --mjolnir-admin '/data/valhalla/valhalla_tiles/admins.sqlite' > valhalla.json

# build script
RUN echo 'valhalla_build_tiles -c valhalla.json /data/openstreetmap/*.osm.pbf; valhalla_export_edges --config valhalla.json > /data/polylines/pbf_extract.polyline;' > ./docker_build.sh