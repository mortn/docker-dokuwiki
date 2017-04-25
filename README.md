xlrl/dokuwiki
==================

[![Docker Stars](https://img.shields.io/docker/stars/xlrl/dokuwiki.svg)](https://hub.docker.com/r/xlrl/dokuwiki/)
[![Docker Pulls](https://img.shields.io/docker/pulls/xlrl/dokuwiki.svg)](https://hub.docker.com/r/xlrl/dokuwiki/)
[![Docker Build](https://img.shields.io/docker/automated/xlrl/dokuwiki.svg)](https://hub.docker.com/r/xlrl/dokuwiki/)
[![Layers](https://images.microbadger.com/badges/image/xlrl/dokuwiki.svg)](https://microbadger.com/images/xlrl/dokuwiki)
[![Version](https://images.microbadger.com/badges/version/xlrl/dokuwiki.svg)](https://microbadger.com/images/xlrl/dokuwiki)
[![Join the chat at https://gitter.im/xlrl/docker-dokuwiki](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/xlrl/docker-dokuwiki?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

Docker container image with [DokuWiki](https://www.dokuwiki.org/dokuwiki) and nginx

### How to run

Assume your docker host is localhost and HTTP public port is 8000 (change these values if you need).

First, run new dokuwiki container:

    docker run -d -p 8000:80 --name dokuwiki xlrl/dokuwiki:2.0

Then setup dokuwiki using installer at URL `http://localhost:8000/install.php`

### How to make data persistent

To make sure data won't be deleted if container is removed, create an empty container named `dokuwiki-data` and attach DokuWiki container's volumes to it. Volumes won't be deleted if at least one container owns them.

    # create data container
    docker run --volumes-from dokuwiki --name dokuwiki-data busybox

    # now you can safely delete dokuwiki container
    docker stop dokuwiki && docker rm dokuwiki

    # to restore dokuwiki, create new dokuwiki container and attach dokuwiki-data volume to it
    docker run -d -p 8000:80 --volumes-from dokuwiki-data --name dokuwiki xlrl/dokuwiki:2.0

### Persistent plugins

Dokuwiki installs plugins to `lib/plugins/`, but this folder isn't inside persistent volume storage by default, so all plugins will be erased when container is re-created.  The recommended way to make plugins persistent is to create your own Docker image with `xlrl/dokuwiki` as a base image and use shell commands inside the Dockerfile to install needed plugins.

Example (install [Dokuwiki ToDo](https://www.dokuwiki.org/plugin:todo) plugin):

    FROM xlrl/dokuwiki
    MAINTAINER Ilya Stepanov <dev@ilyastepanov.com>

    # this is an example Dockerfile that demonstrates how to add Dokuwiki plugins to xlrl/dokuwiki image

    RUN apt-get update && \
        apt-get install -y unzip && \
        apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

    # add todo plugin
    RUN curl -O -L "https://github.com/leibler/dokuwiki-plugin-todo/archive/stable.zip" && \
        unzip stable.zip -d /var/www/lib/plugins/ && \
        mv /var/www/lib/plugins/dokuwiki-plugin-todo-stable /var/www/lib/plugins/todo && \
        rm -rf stable.zip

### How to backup data

    # create dokuwiki-backup.tar.gz archive in current directory using temporaty container
    docker run --rm --volumes-from dokuwiki -v $(pwd):/backup ubuntu tar zcvf /backup/dokuwiki-backup.tar.gz /var/dokuwiki-storage

**Note:** only these folders are backed up:

* `data/pages/`
* `data/meta/`
* `data/media/`
* `data/media_attic/`
* `data/media_meta/`
* `data/attic/`
* `conf/`

### How to restore from backup

    #create new dokuwiki container, but don't start it yet
    docker create -p 8000:80 --name dokuwiki xlrl/dokuwiki:2.0

    # create data container for persistency (optional)
    docker run --volumes-from dokuwiki --name dokuwiki-data busybox

    # restore from backup using temporary container
    docker run --rm --volumes-from dokuwiki -w / -v $(pwd):/backup ubuntu tar xzvf /backup/dokuwiki-backup.tar.gz

    # start dokuwiki
    docker start dokuwiki
