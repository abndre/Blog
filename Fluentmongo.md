# Docker + mongodb + fluentd

Dockerfile para fluend com plugin do mongo
````
FROM fluent/fluentd

# add mongo plugin
RUN apk add --no-cache bash make gcc libc-dev ruby-dev \
    && gem install fluent-plugin-mongo \
    && apk del make gcc libc-dev ruby-dev \
    && rm -rf /var/cache/apk/* \
    && rm -rf /tmp/* /var/tmp/* /usr/lib/ruby/gems/*/cache/*.gem

````

````
docker run -t fluentmongo.
````

Arquivo fluent.conf

````
<source>

@type http

port 9880

bind 0.0.0.0

</source>

  

<match sample.**>

@type stdout

</match>

  

<match mongo.**>

# plugin type

@type mongo

  

# mongodb db + collection

database fluentd

collection logs

  

# mongodb host + port

host 172.17.0.3

port 27017

  

# interval

<buffer>

flush_interval 10s

</buffer>

  

# make sure to include the time key

<inject>

time_key time

</inject>

</match>
````

# Criando container fluent

Executar na pasta do arquivo fluentd.conf
````
docker run -d \
  -p 9880:9880 -v $(pwd):/fluentd/etc -e FLUENTD_CONF=fluentd.conf \
  fluentmongo
````

### Ip do container

````
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' 7888
````

# curl para teste

````
curl -X POST -d 'json={"json":"message"}' http://localhost:9880/sample.test

curl -X POST -d 'json={"json":"message"}' http://localhost:9880/mongo.test

````
