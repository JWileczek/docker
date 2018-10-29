FROM pelias/baseimage

RUN git clone https://github.com/JWileczek/schema.git /code/pelias/schema

WORKDIR /code/pelias/schema

RUN git checkout polygon-testing

# install npm dependencies
RUN npm install