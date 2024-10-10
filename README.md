# Welcome to Zammad

> [!IMPORTANT]  
> This is a fork of Zammad's compose files with some opinionated defaults and documentation missing from Zammad's end.

> [!CAUTION]
> I cannot stress this enough, do not use Zammad's documentation for backup/restore!
>
> Instructions are provided below which won't make you trip up and screw up your install.
>
> Zammad seems to explicitly [not support this use-case](https://github.com/zammad/zammad-docker-compose/issues/68#issuecomment-381327989) and have done nothing since to rectify their documentation.

Zammad is a web based open source helpdesk/ticket system with many features
to manage customer communication via several channels like telephone, facebook,
twitter, chat and emails. It is distributed under the GNU AFFERO General Public
 License (AGPL). Do you receive many emails and want to answer them with a team of agents?
You're going to love Zammad!

## Use cases

This repository is the starting point if you want to:

- deploy Zammad in a containerized production environment
- test the current `stable` or `develop` versions of Zammad

## Backups

Zammad comes with a built-in tool that creates backups for you, however it will not help you restore your installation in the case of Docker-based installs.

As part of your regular backups, make a copy of:
- `/var/lib/docker/volumes/zammad_zammad-backup/_data/` or the latest `.psql.gz` file available
- `/var/lib/docker/volumes/zammad_zammad-storage`

### Restoring

Clone this repository and restore your `.env` file (which contains version information too!).

Restore the `/var/lib/docker/volumes/zammad_zammad-storage` volume from old host.

Restoring a database backup:

```bash
# make sure everything is stopped
docker compose down

# start the db
docker compose up -d zammad-postgresql

# decompress your database dump
gunzip -c 20241010101010_zammad_db.psql.gz > 20241010101010_zammad_db.psql

# wipe database if exists
docker compose exec zammad-postgresql dropdb zammad_production -U zammad
docker compose exec zammad-postgresql createdb zammad_production -U zammad

# restore to postgres database
docker exec -i zammad-zammad-postgresql-1 psql -U zammad -d zammad_production < /path/to/your/20241010101010_zammad_db.psql

# start everything
docker compose up -d
```

## Upgrading

For upgrading instructions see our [Releases](https://github.com/zammad/zammad-docker-compose/releases).

## Status

[![ci-remote-image](https://github.com/zammad/zammad-docker-compose/actions/workflows/ci-remote-image.yaml/badge.svg)](https://github.com/zammad/zammad-docker-compose/actions/workflows/ci-remote-image.yaml) [![Dockerhub Pulls](https://badgen.net/docker/pulls/zammad/zammad-docker-compose?icon=docker&label=pulls)](https://hub.docker.com/r/zammad/zammad-docker-compose/)

## Using a reverse proxy

In environments with more then one web applications it is necessary to use a reverse proxy to route connections to port 80 and 443 to the right application.
To run Zammad behind a reverse proxy, we provide `docker-compose.proxy-example.yml` as a starting point.

1. Copy `./.examples/proxy/docker-compose.proxy-example.yml` to your own configuration, e.g. `./docker-compose.prod.yml`
    `cp ./.examples/proxy/docker-compose.proxy-example.yml ./docker-compose.prod.yml`
2. Modify the environment variable `VIRTUAL_HOST` and the name of the external network in `./docker-compose.prod.yml` to fit your environment.
3. Run docker-composer commands with the default and your configuration, e.g. `docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d`

See `.examples/proxy/docker-compose.yml` for an example proxy project.

Like this, you can add your `docker-compose.prod.yml` to a branch of your Git repository and stay up to date by merging changes to your branch.

## Using Rancher

```console
RANCHER_URL=http://RANCHER_HOST:8080 rancher-compose --env-file=.env up
```

## Running without Elasticsearch

Elasticsearch is an optional, but strongly recommended dependency for Zammad. More details can be found in the [documentation](https://docs.zammad.org/en/latest/prerequisites/software.html#elasticsearch-optional). There are however certain scenarios when running without Elasticsearch may be desired, e.g. for very small teams, for teams with limited budget or as a temporary solution for an unplanned Elasticsearch downtime or planned cluster upgrade.

Elasticsearch is enabled by default in the example `docker-compose.yml` file. It is also by default required to run the "zammad-init" command. Disabling Elasticsearch is possible by setting a special environment variable: `ELASTICSEARCH_ENABLED=false` for the `zammad-init` container and removing all references to Elasticsearch everywhere else: the `zammad-elasticsearch` container, its volume and links to it.
