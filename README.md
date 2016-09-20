# dokku solr (beta)

Unofficial solr plugin for dokku. Currently defaults to installing [solr 6.2.0](https://hub.docker.com/_/solr/).

## requirements

- dokku 0.4.x+
- docker 1.8.x

## installation

```shell
# on 0.4.x+
sudo dokku plugin:install https://github.com/IlyaSemenov/dokku-solr.git solr
```

## commands

```
solr:clone <name> <new-name>  NOT IMPLEMENTED
solr:connect <name>           NOT IMPLEMENTED
solr:create <name>            Create a solr service with environment variables
solr:destroy <name>           Delete the service and stop its container if there are no links left
solr:enter <name> [command]   Enter or run a command in a running solr service container
solr:export <name> > <file>   NOT IMPLEMENTED
solr:expose <name> [port]     Expose a solr service on custom port if provided (random port otherwise)
solr:import <name> <file>     NOT IMPLEMENTED
solr:info <name>              Print the connection information
solr:link <name> <app>        Link the solr service to the app
solr:list                     List all solr services
solr:logs <name> [-t]         Print the most recent log(s) for this service
solr:promote <name> <app>     Promote service <name> as SOLR_URL in <app>
solr:restart <name>           Graceful shutdown and restart of the solr service container
solr:start <name>             Start a previously stopped solr service
solr:stop <name>              Stop a running solr service
solr:unexpose <name>          Unexpose a previously exposed solr service
solr:unlink <name> <app>      Unlink the solr service from the app
```

## usage

```shell
# create a solr service named lolipop
dokku solr:create lolipop

# you can also specify the image and image
# version to use for the service
# it *must* be compatible with the
# official solr image
export SOLR_IMAGE="solr"
export SOLR_IMAGE_VERSION="5.3.2"
dokku solr:create lolipop

# you can also specify custom environment
# variables to start the solr service
# in semi-colon separated forma
export SOLR_CUSTOM_ENV="USER=alpha;HOST=beta"
dokku solr:create lolipop

# get connection information as follows
dokku solr:info lolipop

# you can also retrieve a specific piece of service info via flags
dokku solr:info lolipop --config-dir
dokku solr:info lolipop --data-dir
dokku solr:info lolipop --dsn
dokku solr:info lolipop --exposed-ports
dokku solr:info lolipop --id
dokku solr:info lolipop --internal-ip
dokku solr:info lolipop --links
dokku solr:info lolipop --service-root
dokku solr:info lolipop --status
dokku solr:info lolipop --version

# a bash prompt can be opened against a running service
# filesystem changes will not be saved to disk
dokku solr:enter lolipop

# you may also run a command directly against the service
# filesystem changes will not be saved to disk
dokku solr:enter lolipop ls -lah /

# a solr service can be linked to a
# container this will use native docker
# links via the docker-options plugin
# here we link it to our 'playground' app
# NOTE: this will restart your app
dokku solr:link lolipop playground

# the following environment variables will be set automatically by docker (not
# on the app itself, so they won’t be listed when calling dokku config)
#
#   DOKKU_SOLR_LOLIPOP_NAME=/random_name/SOLR
#   DOKKU_SOLR_LOLIPOP_PORT=tcp://172.17.0.1:9200
#   DOKKU_SOLR_LOLIPOP_PORT_9200_TCP=tcp://172.17.0.1:9200
#   DOKKU_SOLR_LOLIPOP_PORT_9200_TCP_PROTO=tcp
#   DOKKU_SOLR_LOLIPOP_PORT_9200_TCP_PORT=9200
#   DOKKU_SOLR_LOLIPOP_PORT_9200_TCP_ADDR=172.17.0.1
#
# and the following will be set on the linked application by default
#
#   SOLR_URL=http://dokku-solr-lolipop:9200
#
# NOTE: the host exposed here only works internally in docker containers. If
# you want your container to be reachable from outside, you should use `expose`.

# another service can be linked to your app
dokku solr:link other_service playground

# since SOLR_URL is already in use, another environment variable will be
# generated automatically
#
#   DOKKU_SOLR_BLUE_URL=http://dokku-solr-other-service:9200

# you can then promote the new service to be the primary one
# NOTE: this will restart your app
dokku solr:promote other_service playground

# this will replace SOLR_URL with the url from other_service and generate
# another environment variable to hold the previous value if necessary.
# you could end up with the following for example:
#
#   SOLR_URL=http://dokku-solr-other-service:9200
#   DOKKU_SOLR_BLUE_URL=http://dokku-solr-other-service:9200
#   DOKKU_SOLR_SILVER_URL=http://dokku-solr-lolipop:9200

# you can also unlink an solr service
# NOTE: this will restart your app and unset related environment variables
dokku solr:unlink lolipop playground

# you can tail logs for a particular service
dokku solr:logs lolipop
dokku solr:logs lolipop -t # to tail

# finally, you can destroy the container
dokku solr:destroy lolipop
```

## Changing database adapter

It's possible to change the protocol for SOLR_URL by setting
the environment variable SOLR_DATABASE_SCHEME on the app:

```
dokku config:set playground SOLR_DATABASE_SCHEME=solr2
dokku solr:link lolipop playground
```

Will cause SOLR_URL to be set as
solr2://dokku-solr-lolipop:9200

CAUTION: Changing SOLR_DATABASE_SCHEME after linking will cause dokku to
believe the solr is not linked when attempting to use `dokku solr:unlink`
or `dokku solr:promote`.
You should be able to fix this by

- Changing SOLR_URL manually to the new value.

OR

- Set SOLR_DATABASE_SCHEME back to its original setting
- Unlink the service
- Change SOLR_DATABASE_SCHEME to the desired setting
- Relink the service

## todo

- implement solr:clone
- implement solr:import
