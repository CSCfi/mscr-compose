# mscr-compose

#### Running the Containers

    docker-compose up yti-terminology-api

#### logs of postgres container

    docker logs yti-postgres

#### If there is permission issue of data directory

    sudo chown :999 mscr-data/data/logs/yti-postgres/
    sudo chmod 770 mscr-data/data/logs/
    sudo chmod g+s mscr-data/data/logs/

#### Server strats at

    localhost:9302
