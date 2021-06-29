A far from prefect but functional way to get sqlite3 database into a Postgres database & an example of using s3 to place media files back to object storage

# Commands to fetch linode objects and prepare

Copy one of the example s3cfg files depending on the storage location.

    cp example.s3cfg-us .s3cfg
    or
    cp example.s3cfg-eu .s3cfg

Add the access and secret keys to .s3cfg before running..

    s3cmd -c .s3cfg get s3://[bucket]/[file] --recursive ./objects
    
    cd objects
    tar -xzvf [tarfile]

### Move sqlite3 file
    cp [sqlite3] ../data 

### Move media files
Move the media folder contents to /media

    cp -r [mediafolder]/ ../media

# Import sqlite3 into postgres

and create .sql and .dump files ready for import to heroku and dokku

### Get the sqlite3 database into postgres 

    docker-compose up -d

### Convert and import sqlite3 data into postgres

    docker exec -it loader pgloader db.sqlite3 postgres://postgres:example@db:5432/postgres

### Dump the postgres data to data/data.sql

As sql file (heroku)

    docker exec postgres pg_dump -U postgres postgres > data/dump.sql

or as dump file (dokku)

    docker exec postgres pg_dump -Fc -U postgres postgres > data/data.dump

Shut down and remove these containers

    docker-compose down -v
    docker stop loader
    docker container rm loader

# Get data to staging and production instances

Staging and production instances should be created already with postgres databases

### Staging

Load the postgres dump into heroku database
    
    heroku pg:reset DATABASE -a [app-name]
    heroku pg:psql DATABASE_URL --app [app-name] < data/dump.sql

## Production
Load the postgres dump into dokku postgres database, which must be empty.

If not empty:

    dokkub dokku postgres:unlink [postgres_db] [app-name]
    dokkub dokku postgres:destroy [postgres_db]

    dokkub dokku postgres:create [postgres_db]
    dokkub dokku postgres:import [postgres_db] < data/data.dump
    dokkub dokku postgres:link [postgres_db] [app-name]

dokkub is an alias in .zshrc

    alias dokkub="ssh -t dokku@[dokku-instance-domain or IP address]"


# Moving media to linode objects

This needs to be done for both staging and production using 2 separate locations

    s3cmd -c .s3cfg-eu put --recursive media/ s3://[staging-bucket]

    s3cmd -c .s3cfg-eu put --recursive media/ s3://[production-bucket]