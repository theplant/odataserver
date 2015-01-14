Simple OData server for MySQL
==============================


[![Build Status][travis-image]][travis-url]

Usage
======

Copy `src/setenv.template` to `src/setenv` and update with the database
credentials. An admin user with privileges to create tables, users etc. is
needed for these features to work.

Run the tests to make sure everything is ok: `npm install; npm test`.

The simplest way to start the server is: `npm start`. First you need install
the node modules though: `npm install` (if you didn't run the tests above).
A MySQL server needs to be running on
the same host and port 80 must not be in use. Also the `ADMIN_USER` and
`ADMIN_PASSWORD` environment variables needs to be set.


Test the installation
---------------------

Here are some queries, just to get you started:

    # Get all rows in wp_options
    curl -H "user:wp" -H "password:wp" http://[IP]/wp/wp_options

    # Select a few columns
    curl -H "user:wp" -H "password:wp" http://[IP]/wp/wp_options\?\$select=option_id,option_name


Run using docker
===============

Prerequisites:

 * docker needs to be installed.

Installation: `docker build --rm -t odataserver .`

Run the container in daemon mode: `docker run -d -p 80:80 -p 81:81 -p 443:443 --name odataserver odataserver`.
The 80 and 443 ports that have been exposed from the container will be routed from the host to the container
using the `-p` flag.

When developing using docker, it is usefull to connect to the containers shell:

    # Start a container and connect to the shell (remove when stopped)
    docker run -t -i --rm -p 80:80 -p 81:81 -p 443:443 -e ADMIN_USER="root" -e ADMIN_PASSWORD="secret" odataserver /bin/bash

    # Start the services
    supervisord &> /tmp/out.txt &

    # Check that all processes are up
    ps -ef

    # Tun the tests
    npm test

Login to MySQL using phpMyAdmin at: `http://[IP]:PORT/phpMyAdmin-4.0.8-all-languages`


Production
----------

The included MySQL server should not be used for production. Disable it by commenting out the
`[program:mysql]` parts with `#` in supervisord.conf

MySQL credentials for an external server should be passed as environment variables that are set when starting the container.

Here is an example: `docker run -d -p 80:80 -p 443:443 -e ADMIN_USER="admin", ADMIN_PASSWORD="secret", HOSTNAME="hostname" base /bin/bash`


Development
===========

Load some test data with the script `./src-sql/init-mysql2.sh`

Run the extended test suite that depends on the data loaded above:
`npm run-script test2`

Generate the documentation: `npm run-script docco`


dtrace
======

The excellent tool `dtrace` can be used for tracing and debugging purposes.
Node and mysql has built in probes, see [howto use](tests/DTRACE.md).

The odata probes are viewed like this:
`sudo dtrace -Z -n 'nodeapp*:::probe{ trace(copyinstr(arg0)); }'``


MS SQL Server - WORK IN PROGRESS
=================================

The `win` folder contains a `package.json` for installation on Windows. The
[mssql module](https://www.npmjs.org/package/mssql) is used instead of the
[mysql module](https://www.npmjs.org/package/mysql). This version don't include support
for BLOB storage in leveldb since the [leveldb package](https://www.npmjs.org/package/leveldb)  
don't support windows.

Run `npm test` to validate that the setup is ok.


Known issues
============

1. Docker build fails during `npm install`. This is rather common and is probably related to problems with the npm package servers. Just build again until it works.


[travis-image]: https://img.shields.io/travis/gizur/odataserver.svg?style=flat
[travis-url]: https://travis-ci.org/gizur/odataserver
