# What you should have learned until now.

* The Importance of the scicat service for Open and FAIR Data (e.g. in the PhotonScience Domain)
* Docker Infrastructure
  - The advantage of docker and docker compose to build a OS independent system composed of various components.
  - Docker terms like 'images', 'containers', 'docker network'
  - How to let components within a docker network to talk to each other.
  - How to connect the outside with the internal docker world (resp. docker compose world)
    - Port number forwarding (publishing or with traefik.
    - Persitent volumes outside of the container.
    - Environment variables to steer docker containers.

* Design of Simple Database Web Applications within docker.
  - Our [OpenData](https://github.com/PatrickFuhrmann-HTWBerlin/OpenData) project
  - Design a database web service with
    - non-sql (mongodb) backend
    - Backend service with defined REST API for the business logic using nodes.js.
    - Simple Web Frontend with html/css/javascipt only.  (no Framework)
   
* SciCat
  - Mapping our own OpenData design to the scicat design.
  - Looking into the 'scicatlive' subproject.
  - Getting and discussing suggestions from Carlo M. (from PSI) on what we could work.
  - Understanding the oai-pmh service in scicat.
