FROM ruby:2.5-stretch

# Install System libraries
RUN apt-get update && apt-get install -y \
    git \
    imagemagick \
    nginx \
    supervisor \
    awscli \
  && rm -rf /var/lib/apt/lists/*

# gpg keys listed at https://github.com/nodejs/node#release-team
RUN set -ex \
  && for key in \
    4ED778F539E3634C779C87C6D7062848A1AB005C \
    B9E2F5981AA6E0CD28160D9FF13993A75599653C \
    94AE36675C464D64BAFA68DD7434390BDBE9B9C5 \
    B9AE9905FFD7803F25714661B63B535A4C206CA9 \
    77984A986EBC2AA786BC0F66B01FBB92821C587A \
    71DCFD284A79C3B38668286BC97EC7A07EDE3FC1 \
    FD3A5288F042B6850C66B31F09FE44734EB7990E \
    8FCCA13FEF1D0C2E91008E09770F7A9A5AE15600 \
    C4F0DFFF4E8C1A8236409D08E73BC641CC11F4C8 \
    DD8F2338BAE7501E3DD5AC78C273792F7D83545D \
    A48C2BEE680E841632CD4E44F07496B3EB3C1762 \
  ; do \
    gpg --batch --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys "$key" || \
    gpg --batch --keyserver hkp://ipv4.pool.sks-keyservers.net --recv-keys "$key" || \
    gpg --batch --keyserver hkp://pgp.mit.edu:80 --recv-keys "$key" ; \
  done

ENV NPM_CONFIG_LOGLEVEL info
ENV NODE_VERSION 6.14.4

RUN curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.tar.xz" \
  && curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc" \
  && gpg --batch --decrypt --output SHASUMS256.txt SHASUMS256.txt.asc \
  && grep " node-v$NODE_VERSION-linux-x64.tar.xz\$" SHASUMS256.txt | sha256sum -c - \
  && tar -xJf "node-v$NODE_VERSION-linux-x64.tar.xz" -C /usr/local --strip-components=1 \
  && rm "node-v$NODE_VERSION-linux-x64.tar.xz" SHASUMS256.txt.asc SHASUMS256.txt \
  && ln -s /usr/local/bin/node /usr/local/bin/nodejs

# Install foreman
RUN gem install foreman --no-document

# App Support tools
RUN mkdir -p /app_support/geocoder \
  && touch /app_support/geocoder/geodb.mmdb \
  && curl -s "http://geolite.maxmind.com/download/geoip/database/GeoLite2-City.mmdb.gz" | gunzip -c > /app_support/geocoder/geodb.mmdb \
  && chown -R www-data:www-data /app_support

# Setup
RUN mkdir -p /app /var/log/app \
  && rm -f /etc/nginx/sites-enabled/* \
  && chown -R www-data:www-data /app /var/log/app

# Configuration
COPY shared/nginx.conf /etc/nginx/
COPY shared/app.conf /etc/nginx/sites-enabled/
COPY shared/supervisord.conf /etc/supervisor/

# Expose 80
EXPOSE 80

# Healthcheck
COPY shared/healthcheck.sh /usr/local/bin/
HEALTHCHECK --timeout=5s CMD bash /usr/local/bin/healthcheck.sh

# Entrypoint command
COPY shared/docker-entrypoint.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/docker-entrypoint.sh
ENTRYPOINT ["/usr/local/bin/docker-entrypoint.sh"]
CMD [ "/usr/bin/supervisord", "-c", "/etc/supervisor/supervisord.conf" ]