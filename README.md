# dokku sphinx (beta) [![Build Status](https://img.shields.io/travis/dokku/dokku-sphinx.svg?branch=master "Build Status")](https://travis-ci.org/dokku/dokku-sphinx) [![IRC Network](https://img.shields.io/badge/irc-freenode-blue.svg "IRC Freenode")](https://webchat.freenode.net/?channels=dokku)

Official sphinx plugin for dokku. Currently defaults to installing [sphinx 5.7.10](https://hub.docker.com/_/sphinx/).

## requirements

- dokku 0.4.0+
- docker 1.8.x

## installation

```shell
# on 0.3.x
cd /var/lib/dokku/plugins
git clone https://github.com/dokku/dokku-sphinx.git sphinx
dokku plugins-install

# on 0.4.x
dokku plugin:install https://github.com/dokku/dokku-sphinx.git sphinx
```

## commands

```
sphinx:clone <name> <new-name>  Create container <new-name> then copy data from <name> into <new-name>
sphinx:connect <name>           Connect via sphinx to a sphinx service
sphinx:create <name>            Create a sphinx service with environment variables
sphinx:destroy <name>           Delete the service and stop its container if there are no links left
sphinx:export <name> > <file>   Export a dump of the sphinx service database
sphinx:expose <name> [port]     Expose a sphinx service on custom port if provided (random port otherwise)
sphinx:import <name> < <file>   Import a dump into the sphinx service database
sphinx:info <name>              Print the connection information
sphinx:link <name> <app>        Link the sphinx service to the app
sphinx:list                     List all sphinx services
sphinx:logs <name> [-t]         Print the most recent log(s) for this service
sphinx:promote <name> <app>     Promote service <name> as DATABASE_URL in <app>
sphinx:restart <name>           Graceful shutdown and restart of the sphinx service container
sphinx:start <name>             Start a previously stopped sphinx service
sphinx:stop <name>              Stop a running sphinx service
sphinx:unexpose <name>          Unexpose a previously exposed sphinx service
sphinx:unlink <name> <app>      Unlink the sphinx service from the app
```

## usage

```shell
# create a sphinx service named lolipop
dokku sphinx:create lolipop

# you can also specify the image and image
# version to use for the service
# it *must* be compatible with the
# official sphinx image
export SPHINX_IMAGE="sphinx"
export SPHINX_IMAGE_VERSION="5.5"

# you can also specify custom environment
# variables to start the sphinx service
# in semi-colon separated forma
export SPHINX_CUSTOM_ENV="USER=alpha;HOST=beta"

# create a sphinx service
dokku sphinx:create lolipop

# get connection information as follows
dokku sphinx:info lolipop

# a sphinx service can be linked to a
# container this will use native docker
# links via the docker-options plugin
# here we link it to our 'playground' app
# NOTE: this will restart your app
dokku sphinx:link lolipop playground

# the following environment variables will be set automatically by docker (not
# on the app itself, so they won’t be listed when calling dokku config)
#
#   DOKKU_SPHINX_LOLIPOP_NAME=/lolipop/DATABASE
#   DOKKU_SPHINX_LOLIPOP_PORT=tcp://172.17.0.1:3306
#   DOKKU_SPHINX_LOLIPOP_PORT_3306_TCP=tcp://172.17.0.1:3306
#   DOKKU_SPHINX_LOLIPOP_PORT_3306_TCP_PROTO=tcp
#   DOKKU_SPHINX_LOLIPOP_PORT_3306_TCP_PORT=3306
#   DOKKU_SPHINX_LOLIPOP_PORT_3306_TCP_ADDR=172.17.0.1
#
# and the following will be set on the linked application by default
#
#   DATABASE_URL=sphinx://sphinx:SOME_PASSWORD@dokku-sphinx-lolipop:3306/lolipop
#
# NOTE: the host exposed here only works internally in docker containers. If
# you want your container to be reachable from outside, you should use `expose`.

# another service can be linked to your app
dokku sphinx:link other_service playground

# since DATABASE_URL is already in use, another environment variable will be
# generated automatically
#
#   DOKKU_SPHINX_BLUE_URL=sphinx://sphinx:ANOTHER_PASSWORD@dokku-sphinx-other-service:3306/other_service

# you can then promote the new service to be the primary one
# NOTE: this will restart your app
dokku sphinx:promote other_service playground

# this will replace DATABASE_URL with the url from other_service and generate
# another environment variable to hold the previous value if necessary.
# you could end up with the following for example:
#
#   DATABASE_URL=sphinx://sphinx:ANOTHER_PASSWORD@dokku-sphinx-other_service:3306/other_service
#   DOKKU_SPHINX_BLUE_URL=sphinx://sphinx:ANOTHER_PASSWORD@dokku-sphinx-other-service:3306/other_service
#   DOKKU_SPHINX_SILVER_URL=sphinx://sphinx:SOME_PASSWORD@dokku-sphinx-lolipop:3306/lolipop

# you can also unlink a sphinx service
# NOTE: this will restart your app and unset related environment variables
dokku sphinx:unlink lolipop playground

# you can tail logs for a particular service
dokku sphinx:logs lolipop
dokku sphinx:logs lolipop -t # to tail

# you can dump the database
dokku sphinx:export lolipop > lolipop.sql

# you can import a dump
dokku sphinx:import lolipop < database.sql

# you can clone an existing database to a new one
dokku sphinx:clone lolipop new_database

# finally, you can destroy the container
dokku sphinx:destroy lolipop
```

## Changing database adapter

It's possible to change the protocol for DATABASE_URL by setting
the environment variable SPHINX_DATABASE_SCHEME on the app:

```
dokku config:set playground SPHINX_DATABASE_SCHEME=sphinx2
dokku sphinx:link lolipop playground
```

Will cause DATABASE_URL to be set as
sphinx2://sphinx:SOME_PASSWORD@dokku-sphinx-lolipop:3306/lolipop

CAUTION: Changing SPHINX_DATABASE_SCHEME after linking will cause dokku to
believe the service is not linked when attempting to use `dokku sphinx:unlink`
or `dokku sphinx:promote`.
You should be able to fix this by

- Changing DATABASE_URL manually to the new value.

OR

- Set SPHINX_DATABASE_SCHEME back to its original setting
- Unlink the service
- Change SPHINX_DATABASE_SCHEME to the desired setting
- Relink the service
