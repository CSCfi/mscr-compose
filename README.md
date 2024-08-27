# mscr-compose

#### Running datamodel API from docker image


#### Clone the MSCR-Compose Repository
- clone the repository using either SSH or HTTPS.
- SSH link to repo:git@github.com:CSCfi/mscr-compose.git
- HTTPS link to repo: https://github.com/CSCfi/mscr-compose.git

#### Create data directory 
- Create a Data Directory name mscr-data
- If there is permission issue of data directory

    - sudo chown :999 mscr-data/data/logs/yti-postgres/
    - sudo chmod 770 mscr-data/data/logs/
    - sudo chmod g+s mscr-data/data/logs/


#### Running Datamodel API
- run docker-compose up yti-terminology-api.
- Server starts at localhost:9302. 
     
#### API Documentation
- Go to http://localhost:9004/datamodel-api/swagger-ui/index.html, click Authorize and insert your token to use Swagger
- You can try the api calls after you have been authorized and have own token.


#### Setting up the database
- In the terminal of yti-postgres container run the following commands:
        psql -U postgres
        create user mscr with password 'msrc';
        create database mscr_datamodel with owner mscr;

#### Create admin account
- go to scripts directory and run bash init-admin.sh command to create the admin user.

#### logs of postgres container

    docker logs yti-postgres




