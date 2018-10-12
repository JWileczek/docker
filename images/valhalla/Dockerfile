# base image
FROM pelias/baseimage

RUN apt-get update

# grab all of the valhalla software from ppa
RUN apt-get install -y software-properties-common python3-software-properties && \
    apt-get update

RUN add-apt-repository -r ppa:valhalla-core/valhalla && \
    apt-get update

# Install packages for valhalla
RUN apt-get install -y cmake g++ gcc jq lcov protobuf-compiler vim-common libboost-all-dev libboost-all-dev zlib1g-dev liblz4-dev libprotobuf-dev nodejs npm

# Install packages for prime_server
RUN apt-get install -y libgeos-dev libgeos++-dev liblua5.2-dev libspatialite-dev libsqlite3-dev lua5.2 wget
RUN apt-get install -y python-all-dev
RUN apt-get install -y libcurl4-openssl-dev libzmq3-dev libczmq-dev

# Clone and build prime server
RUN mkdir -p /code/prime_server
WORKDIR /code/prime_server

RUN git clone https://github.com/kevinkreiser/prime_server.git /code/prime_server
RUN git submodule update --init --recursive
RUN ./autogen.sh
RUN ./configure
RUN make test -j8 && \
    make install

# Clone and build valhalla
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


# Create data output directory for tiles
RUN mkdir -p /data/valhalla/valhalla_tiles

# generate valhalla config
RUN valhalla_build_config \
  --mjolnir-tile-dir '/data/valhalla/valhalla_tiles' \
  --mjolnir-tile-extract '/data/valhalla/valhalla_tiles.tar' \
  --mjolnir-timezone '/data/valhalla/valhalla_tiles/timezones.sqlite' \
  --mjolnir-admin '/data/valhalla/valhalla_tiles/admins.sqlite' > valhalla.json

# build script to build tiles and export polyline data.
RUN echo 'valhalla_build_tiles -c valhalla.json /data/openstreetmap/*.osm.pbf; valhalla_export_edges --config valhalla.json > /data/polylines/extract.0sv;' > ./docker_build.sh