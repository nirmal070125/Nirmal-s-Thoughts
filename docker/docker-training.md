curl -L https://github.com/docker/machine/releases/download/v0.7.0/docker-machine-`uname -s`-`uname -m` > /usr/local/bin/docker-machine && chmod +x /usr/local/bin/docker-machine

docker-machine version

docker-machine create -d virtualbox --engine-insecure-registry dockerhub.private.wso2.com --virtualbox-disk-size "40000" --virtualbox-memory "8098" dev

docker-machine env dev

eval $(docker-machine env dev)

docker ps (if this doesn’t work, install the client)

docker-machine ssh dev

docker run -i -t dockerhub.private.wso2.com/ubuntu


run the application in the fore ground so that you can see the logs using docker logs -f <id>

docker exec -i -t 861355a34cdc '/bin/bash'  (without having SSH inside the container)


every time you ran RUN command, it’ll create a docker layer


docker compose - a way to build patterns
 
