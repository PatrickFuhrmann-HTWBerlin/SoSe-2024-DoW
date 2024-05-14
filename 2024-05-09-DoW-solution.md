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