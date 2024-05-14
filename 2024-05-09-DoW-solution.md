# Solution for Description of work for 2024-05-09

## Clean up docker
Make sure your docker system is clean.
- Stop all running containers with 'docker ps' and 'docker remove &lt;id&gt;'.
- Remove all stopped containers with 'docker ps -a' and 'docker remove of the '&lt;id&gt;'.
- Remove all images with 'docker images' and 'docker image remove &lt;id&gt;'
- Remove all volumnes with
  - Remove anonymous volumes
    - docker volume prune
  - Remove named volumnes
    - docker volume ls
    - docker volume remove '&lt;id&gt;'

## Start with clean git clones
```
mkdir git-scicat
cd git-scicat
#
```
## Get the original backend and build the image for your platform.
```
#
git clone https://github.com/SciCatProject/scicat-backend-next
#
cd scicat-backend-next
#
docker build -t backendnext .
#
cd ..
```
##  Get our Scicat Live Mini and replace the official 'backend' with our image.
```
git clone https://github.com/PatrickFuhrmann-HTWBerlin/scicatlive-mini.git
#
cd scicatlive-mini
cd services/backendv4
#
```
## Edit docker-compose.yaml
- replace *image: ghcr.io/scicatproject/backend-next:v4.5.0*
- with *image: backendnext*
- make sure you insert the correct number of blanks or 'tabs' before 'image:'
## Start mongodb/backend and mongo-express
```
cd ../../
docker compose up -d backend
docker compose up -d express
```
Check if things are running with *docker ps*, otherwise use 'docker logs ID'

## Run checks

### Check the raw mongodb with mongo-express and a browser
To verify the content of the mongo databases, connect to the mongo-express service with a browser. URL on your laptop:
```
http://localhost:7010/
```
URL in Juelich:
```
http://scicat.webhub.net:7010/
```
- User and password are 'view' 'view'
- **NOTICE**: Starting up the mongodb-express can take up to a minute. You can watch the log output with express logfiles with 'docker logs scicatlive-mini-express-1' or 'docker logs &lt;ID of the Express Container&gt;'. For some time the logfile will complain about
```
/docker-entrypoint.sh: line 15: /dev/tcp/mongo/27017: Invalid argument
```
But after a timeout it will process and the browser will show the express webpage.
The web page shows the different databases and collections. Most importantly: 
- Database: 'dacat-next'
  - Collections:
    - *Dataset*: The registered dataset metadata
    - *PublishedData*: The subset for publication with oai-pmh

### Check the REST-full service of the backend.
The backend generates the scicat data model and makes it available through a RESTfull interface. This interface is used by the frontend. You can check if the REST service works with:
```
curl -L --silent -H "Accept: application/json" "http://localhost:7009/api/v3/Datasets"
```
or a nicer display with the 'jq' command.
```
curl -L --silent -H "Accept: application/json" "http://localhost:7009/api/v3/Datasets" | jq .
```
or if you are only interested in the ID's
```
curl -L --silent -H "Accept: application/json" "http://localhost:7009/api/v3/Datasets" | jq '.[].id'
```

# The OAI-PMH Part
Although we want to run the oai-pmh service inside a docker compose system, we begin with starting it with docker but w/o docker-compose. To do that, we only have to place the oai-pmh container into the docker compose network, we created above. 

### Get our oai-pmh service 
```
git clone https://github.com/PatrickFuhrmann-HTWBerlin/oai-pmh-service
cd oai-pmh-service
```

## Create the image
```
docker build -t oai-pmh-service .
```
## Run the image in the docker-compose network, but w/o docker compose yet.
Get the docker compose networks with
```
docker network ls
```
This will return something like:
```
NETWORK ID     NAME                      DRIVER    SCOPE
a05664f9c45b   bridge                    bridge    local
045862b12457   host                      host      local
5803d4092074   none                      null      local
f7d437590153   proxy_default             bridge    local
89b8214000c5   scicatlive-mini_default   bridge    local
```
So the network seems to be **scicatlive-mini_default**.

Now start the docker image directly with docker and not yet with docker-compose, like this:
```
docker run \
     --network scicatlive-mini_default \
     -e DB_HOST=mongodb \
     -p 7005:3001  \
     oai-pmh-service 
```
Explaination:
- **--network scicatlive-mini_default** places this docker container in the network which was previously created by the docker-compose system.
- **-e DB_HOST=mongodb**: The oai-pmh system normally connects to localhost:27017, when running within a single host. But as in a docker-compose system the different containers behave like individual hosts, we have to set the hostname of the mongo database to *mongodb* as defined in the  *services/mongodb/docker-compose.yaml* file.
- **-p 7005:3001**: The oai-phm services is listening to the local portnumber 3001. This is defined in the *production/host_config.json* file. The command option **-p 7005:3001** redirects the outside portnumber *7005* to the inside portnumber *3001*.
- Remark: We are using the portrange from 7000 to 7020, because those ports are open for ingres to the Juelich VMs.

## Check if the service is running.

#### Using docker commands.
Check with 
```
docker ps
``` 
that an image named **oai-pmh-service** is active. The port column should indicate that the outside port 7005 is redirected to the inside portnumber of 3001.

If you connect to this docker container with:
```
docker exec -it <ID> /bin/sh
```
**ID** is the container id of the *oai-pmh-service* image.
You can check the process table and the network listen ports with
```
ps -edf
netstat -an
```
You should see the 'node src/index.js' process and a port listening on 3001

#### Using RESTful calls

The easiest way to check if the oai-pmh service is (formally) running is to request the root directory.
```
curl http://localhost:7005
```
It should return a json string, containing the start date/time and the uptime of the serivce, e.g.:
```
{"started":"2024-05-14T19:10:24Z","uptime":331763.837}
```
This only indicates that the main webserver is running but not that the service is producing reasonable results. For that we need to check the requests defined by the OAI-PMH RESTful definitions.
We start with the request to the service to identify itself.
```
curl -s "http://localhost:${PORT}/scicat/oai?verb=Identify"
```
The result should be a more or less cryptic xml string. The string can be nicely formated with the *xmllint* command. The command can be installed on Ubuntu-like systems with 
```
sudo apt install libxml2-utils
```
and on MAC e.g. with homebrew.
```
curl -s "http://localhost:${PORT}/scicat/oai?verb=Identify" | xmllint --format -
```
It returns the namespaces of the XML document and some identification attributes, like
```
<?xml version="1.0" encoding="UTF-8"?>
<OAI-PMH xmlns="http://www.openarchives.org/OAI/2.0/" ... ">
  <responseDate>2024-05-14T19:10:24Z</responseDate>
  <request verb="Identify">http://localhost/scicat/oai</request>
  <Identify>
    <repositoryName>Scicat Provider</repositoryName>
    <baseURL>http://localhost/scicat/oai</baseURL>
    <protocolVersion>2.0</protocolVersion>
    <adminEmail>nobody@htw-berlin.de</adminEmail>
    <earliestDatestamp>2017-01-00T03:24:00Z</earliestDatestamp>
    <deletedRecord>no</deletedRecord>
    <granularity>YYYY-MM-DDThh:mm:ssZ</granularity>
  </Identify>
</OAI-PMH>
```
The next REST call is supposed to return the supported metadata formats. (**verb=ListMetadataFormats**)
```
curl "http://localhost:${PORT}/scicat/oai?verb=ListMetadataFormats" | xmllint --format -
```
It should return a xml document, reporting the following metadata formats:
- oai_dc
- panosc
- oai_datacite
with its corresponding links to the definitions.

## Conslusion up to now.
Up to now the oai-pmh service is responding as expected. However, none of the REST calls really queried the content of the Published Database.

One of the REST calls which queries the database is **verb=ListIdentifiers**. It lists the identifiers of all records in the publishing database.
- Command: **verb=ListIdentifiers**
- MetadataFormat selection: **metadataPrefix=oai_dc** (which is one of the formats returned below)
```
curl "http://localhost:3000/scicat/oai?verb=ListIdentifiers&metadataPrefix=oai_dc"
```
The result is a bit disappointing. Although the **mongodb express** webpage is showing 14 documents of meta data in the database:
- Database: **dacat-next**
- Collection: **PublishedData**
Our RESTful call returns an empty array: '{}'

## So our next job is to get the code to run properly.
We need to look into:
- Did we properly configure the system to find the data in the database
- As we only formally compiled the code after we switched from the 'node.js' mongodb driver 3.5 to 6.5 we might need to have to adjust the code to the new driver.

