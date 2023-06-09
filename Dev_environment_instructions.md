# Instructions for generating docker images from VRK-YTI repository

## 1. Prerequisites

Below are the packages (installed eg with homebrew) that would be required:

- gradle
- jenv
- maven
- nvm
- openjdk@17 & openjdk@11
- pyenv
- yarn
- temurin 11 & temurin8 (casks)

If you're using homebrew, best to install it under you default user profile (https://www.youtube.com/watch?v=RT8Rh8yJy-w).

On top of that, you need Docker.

## 2. (Building) Images

### yti-spring-security
yti-spring-security is used to build .jar files, and not used to build any images per se. Change the version of a jar to be generated using the following commands when at the root of the folder:
```
git checkout tags/{{latest tag}}
./gradlew publishToMavenLocal
```
When switching branch, you will see a message in console telling you which java version to use (you can switch it for your current shell with eg jenv). Then the needed jar should be in your m2 folder of your home directory. This will allow you to build certain images that require those .jar files.

### yti-terminology-ui
cd terminology-ui && ./build.sh

### yti-fuseksi 
Comment out the code *after* FROM statement and *until* the first COPY statement. Change FROM statement to  ```FROM stain/jena-fuseki:3.14.0```

### yti-datamodel-api 
For that you first had to build a correct image through yti-docker-java. 
When in yti-docker-java directory, make sure you’re on the master branch and run ```docker build --platform linux/arm64 -t yti-docker-java17-base:corretto .``` to ensure that the image will be built for the M1 processor.

After that, run ```./build.sh in yti-datamodel-api```

### yti-datamodel-ui
After you have built the correct java17 image for the M1 processor with ```docker build --platform linux/arm64 -t yti-datamodel-api .``` use ```./build.sh```

### yti-comments-api
```docker build -f docker/Dockerfile -t yti-comments-api  .```

### yti-messaging-api
```docker build -f docker/Dockerfile -t yti-messaging-api .```

### yti-groupmanagement
You would need node 14.18.2 installed locally. Change node-sass’ package version in package.json to 5.0.0 and set ```download = false``` in the ‘node’ object in the _build.gradle_ file. Then you rebuild the package

### yti-codelist-ui
Run the following command 
``` yti-codelist-ui % docker build -f Dockerfile -t yti-codelist-ui . ```

### yti-terminology-api
You will need to have built spring jar 0.2.0 before running ./build.sh in yti-spring-security in the corresponding branch. Refer to the instruction above.


### yti-postgres

### yti-codelist-common-model 
Contains jar files, a shared library

### yti-codelist-public-api-service

### yti-codelist-content-intake-service

### yti-activemq 
### yti-terminology-termed-docker

*** at some point comment out checksum check in Dockerfile ***

``` # && echo "${SHA} /tmp/apache-maven.tar.gz" | sha512sum -c - ```

then run ```./build.sh```

### yti-comments-api

### yti-comments-ui


## 3. After the images have been built – Docker final touches

### Changing images' names
**NB!** These instructions have been written with Apple's M1 processors in mind. If you use a machine with a different processor, please adjusts the tags (part after the colon sign) below accordingly.

In docker-compose.yml, change names of images from eg *yti-example-name:latest*, *yti-example-name:latest-build* , *yti-example-name:somthing_else* to *docker-registry.rahti.csc.fi/mscr-test/yti-example-name:latest-arm64-v8*. Do it for all images.


### Adding OpenSearch

Add the following code in /mscr-compose/docker-compose.yml to add OpenSearch.
```
yti-datamodel-opensearch:
    image: opensearchproject/opensearch:latest
    container_name: "yti-datamodel-opensearch"
    restart: always
    ports:
    - "9003:9200"
    - "9013:9300"
    environment:
    - plugins.security.disabled=true
    - cluster.name=yti-datamodel-opensearch
    - http.host=0.0.0.0
    - network.publish_host=127.0.0.1
    - discovery.type=single-node
    volumes:
    - ${DATADIR}/data/logs/yti-datamodel-opensearch:/usr/share/opensearch/logs:z
    networks:
    - yti-network
```

### Updating yti-datamodel-api

Add the following code in /mscr-compose/docker-compose.yml to add new links and dependencies to yti-datamodel-api.

```
yti-datamodel-api:    
    image: "docker-registry.rahti.csc.fi/mscr-test/yti-datamodel-api:latest-arm64-v8"
    container_name: "yti-datamodel-api"
    expose:
     - "9001"
     - "9004"
    ports:
     - "9001:9001"
     - "9004:9004"
    environment:
     - SPRING_PROFILES_ACTIVE=docker
     - SPRING_CONFIG_LOCATION=/config/application.yml,/config/yti-datamodel-api.yml
    volumes:
     - ./config:/config:Z
     - ${DATADIR}/data/logs/yti-datamodel-api:/data/logs/yti-datamodel-api:Z
    depends_on:
     - yti-fuseki
     - yti-groupmanagement
     - yti-datamodel-opensearch
     - yti-terminology-termed-api
    links:
     - yti-fuseki
     - yti-groupmanagement
     - yti-datamodel-opensearch
     - yti-terminology-termed-api
    networks:
     - yti-network
```

### Composing containers

Run the following command in /mscr-compose: ```docker-compose up yti-datamodel-api```
Then delete yti-datamodel-api container, as you will be running it outside of containers, eg through Eclipse.

## 4. Eclipse setup


1. Clone https://github.com/CSCfi/mscr-datamodel-api.git repo
2. In the terminal of yti-postgres container run the following commands:
    psql -U postgres
    create user mscr with password 'msrc';
    create database mscr_datamodel with owner mscr;
3. Then in Eclipse go to Help -> Eclipse Marketplace and download Spring Boot Suite 4
4. Right click on the mscr-datamodel-api folder, go to Run As and click Run Configurations
5. Go to the Environment tab, add SPRING_PROFILES_ACTIVE variable with *local* value
6. Go to yti-fuseki container and notice automatically generated password
7. In mscr-datamodel-api/src/main/resources/config create an *application-local.properties* file
8. Copy the following code there:
```
server.port=9004

devMode=true
env=dev
spring.application.name=yti-datamodel-api
endpoint=http://localhost:3030
endpointPassword= ***YOUR AUTO GENERATED PASSWORD FROM ABOVE HERE***
defaultNamespace=http://uri.suomi.fi/datamodel/ns/
provenance=true
defaultGroupManagementAPI=http://localhost:9302/api/
privateGroupManagementAPI=http://localhost:9302/api/
defaultSuomiCodeServerAPI=https://koodistot.dev.yti.cloud.dvv.fi/codelist-api/api/
defaultTerminologyAPI=http://localhost:9102/terminology-api/api/
privateTerminologyAPI=http://localhost:9102/terminology-api/private/
publicGroupManagementAPI=http://localhost:9302

publicGroupManagementFrontend=http://localhost:9302
publicSuomiCodeServerFrontend=https://koodistot.dev.yti.cloud.dvv.fi
publicTerminologyFrontend=http://localhost:9302/
messagingEnabled=false

groupmanagement.url=http://localhost:9302
groupmanagement.public.url=http://localhost:9302

#Old elasticsearch properties

elasticHost=127.0.0.1
elasticPort=9300
elasticHttpPort=9002
elasticHttpScheme=http
elasticCluster=elasticsearch

openSearchHost=127.0.0.1
openSearchHttpPort=9003
openSearchHttpScheme=http
allowComplexOpenSearchQueries=false

migration.enabled=true
migration.packageLocation=fi.vm.yti.datamodel.api.migration.task

impersonate.allowed=true

fake.login.allowed=true
fake.login.mail=testi.testaaja@example.org
fake.login.firstName=Testi
fake.login.lastName=Testaaja

spring.elasticsearch.uris=http://localhost:9002
management.endpoints.web.exposure.include=health,info

spring.datasource.url=jdbc:postgresql://localhost:5432/mscr_datamodel
spring.datasource.username= *** TO ADD ***
spring.datasource.password= *** TO ADD ***

spring.config.import=optional:configserver:

```
9. Now right click on mscr-datamodel-api and run it as Spring Boot App
10. If you get an error about io.netty.resolver.dns.macos.MacOSDnsServerAddressStreamProvider feel free to skip it
11. With containers running, go to /mscr-compose/scripts directory (different project thant mscr-datamodel-api)
12. Run ./init-admin.sh and go to http://localhost:9302/. There you should find 'Impersonate user' or 'Esiinny käyttäjänä' button that allows you to generate a token
13. Go to http://localhost:9004/datamodel-api/swagger-ui/index.html, click Authorize and insert your token to use Swagger
14. Go to your local Apache Jena Fuseki instance at http://localhost:3030/. Click 'manage datasets' and create *persistent* 'schema' dataset. Now your PUT api calls to v2/schema will go through
