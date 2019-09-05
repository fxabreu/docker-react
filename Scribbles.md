Docker learn
docker ps

#image from a running container
docker commit -c 'CMD ["<default command>"]' <containerid>

# run a shell inside a running container (iteractive)
docker exec -it <containerid> sh
# attach the command to the stdin of the container 
docker exec -i
# formats the output as it comes out from the process 
docker exec -t 

#Lifecycle
docker run = docker create + docker start 
docker create <image name>
docker start <container id>
#(attach console to the output of the container)
docker start -a <container id> 
# list all containers created so far
docker ps --all

docker run -d, --detach //Run container in background and print container ID


# delete all stopped containers 
docker system prune
# get logs of a running container
docker logs <container id>

# build a container with a name (don't forget the dot at the end)
docker build -t <user>/<project-name>:<version | latest> .

# prevent the trigerring of the dependencies build 
# by splitting the copy instructions
COPY ./package.json ./
RUN npm install
COPY ./ ./

#docker compose
version: '3'
services: // containers
    redis-server:  //name of the container
        image: 'redis'  // image to use or
    node-app: //another container
        build: . // use the Dockerfile inside the folder specified
        ports:
            - "4001:8081"  // dash means array
                           // port mapping from 4001 local to 8081 in container

docker-compose up
docker-compose up --build // rebuild the container images

docker-compose down

# Restart policies
"no" - default (use quotes)
always - always restart policy in case of crash
on-failure - only if error codes (exit code greater then 0)
unless-stopped - unless stopped

services:
    redis-server:
        restart: always  // always restart container redis-server in case of crash

//docker-compose status
docker-compose ps

# docker build with a different file
docker build -f Dockerfile.dev (for development)

Docker Volumes
docker run -p 3000:3000 -v /app/node_modules -v $(pwd):/app <image_id>

## PRODUCTION
# Docker file for multiple step build process
# multiple base images
# FROM <base image> as <stage or phase> 
#Builder Phase
FROM node:alpine as builder # anything below will be refered to this stage
WORKDIR '/app'
COPY package.json .
RUN npm install
COPY . .

RUN npm run build

#start of second Phase
# FROM <base image>
FROM nginx
# copy something from a different phase
# COPY --from=<phase>
COPY --from=builder /app/build /usr/share/nginx/html

#Travis CI
language: generic, node_js # not included in the initial code, but added to fix build failure

sudo: required #tell travis we need super user access to run our tests
services:
  - docker # tell travis to initialize  docker instance

before_install: # steps for building the project
  - docker build -t fxabreu/docker-react -f Dockerfile.dev .

script: # steps to execute after the code is built and ready for testing
  - docker run fxabreu/docker-react npm run test -- --coverage
# added "-e CI=true" to attempt fix build failures
  - docker run -e CI=true fxabreu/docker-react npm run test -- --coverage
