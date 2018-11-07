# Docker Basics

#### CLI
```bash
# Show help
docker help

# Show docker version
docker version

# Show running containers
docker ps

# Show all containers containers (including stopped)
docker ps -a

# Show cached docker images
docker images
```

#### Run a basic container
```bash
docker run elasticsearch
```
This looks for an **image** from Docker hub called `elasticsearch` with a default tag of `latest`.

Problems:
* No way to communicate with this elasticsearch
* No persistence after container is killed

#### Open a port
```bash
docker run -p 9200:9200 elasticsearch:5
```
`-p 9200:9200` instructs docker to forward your host's TCP port 9200 to the container's TCP port 9200.


Interact with our new elasticsearch container
```bash
# Create a new document
curl -XPOST localhost:9200/mysite/books/001 -d '
{
  "title": "Java 8 Optional In Depth",
  "category":"Java",
  "published_date":"23-FEB-2017",
  "author":"Rambabu Posa"
}'

# Query
curl -s localhost:9200/mysite/books/_search?q=java | jq
curl -s localhost:9200/mysite/books/_search?q=python | jq
```

Problems:
* Persistence of data is only in the container and is not portable

#### Add a persistent volume
```bash
docker run -p 9200:9200 -v /opt/elasticsearch:/usr/share/elasticsearch/data elasticsearch:5
```
We are adding persistence by mounting our host's `/opt/elasticsearch` directory into the 
container's `/usr/share/elasticsearch/data` directory.

### Useful Docker commands
* `docker exec ...`
* `docker logs ...`

