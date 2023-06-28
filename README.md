# Docker's Tutorial for containers.  

**This README just documents the commands that were ran from the tutorial by Docker, and *hopefully* helps me remember them.**  

- `docker run -d -p 80:80 docker/getting=started`  
	- *`-d`: detached*  
	- *`-p`: maps host's port 80 to container's port 80*  
	- *`docker/getting-started`: is the image to use*  

## Building the Container Image

After creating the **Dockerfile** in the same directory as **package.json**.

- `docker build -t getting-started .`  
	- *`-t` give image a human readable name (in this case, getting-started).*  
	- *`.` specificies location of Dockerfile.*

## Starting a Container  

- `docker run -dp 3000:3000 getting-started`  
	- *same/similar as first command.*
	- *`getting-started` is the name tag given with `-t`.*  

**subsequent starting of containers can be done with the following command**  
- `docker start <CONTAINER_ID_OR_NAME>` *the ID can be partial as long as it does not conflict with another container*

## Updating App within Container

**After changes in source code:**  
- `docker build -t getting-started` *builds updated version of image.*  
*below will throw an error, since the port 3000 is already in use*  
- `docker run -dp 3000:3000 getting-started`  

- **To fix we need to remove the old container.**  
	- `docker ps` *to list running containers, get the ID of container stop.*  
	- `docker stop <CONTAINER_TO_STOP_ID>`  
	*once container stops, remove it by:*  
	- `docker rm <CONTAINER_TO_STOP_ID>`

	***OR***  

	- `docker rm -f <CONTAINER_TO_STOP_ID>` *in one line.*  

	***OR***  

	- You can also remove it from Docker Desktop.  

- **Then restart updated container as in the [Starting a Container](#starting-a-container) section.**  

## Pushing our Image  
- Log into Docker Hub (website) and create a repository.  
- `docker login -u <USER_NAME>` or *Sign-in* from Docker Desktop.  
- `docker tag getting-started <USER_NAME>/getting-started`  
- `docker push <USER_NAME>/getting-started`  

#### Aside. Playing with newly created image.  
- Go to the [Lab Environment](https://labs.play-with-docker.com/) from Play with Docker.
- Sign in if needed.
- `docker run -dp 3000:3000 <USER_NAME>/getting-started`  

## Container's Filesystem  
- `docker run -d ubuntu bash -c "shuf -i 1-10000 -n 1 -o /data.txt && tail -f /dev/null"` *starts an Ubuntu container that creates a /data.txt file containing one random number between 1-10000*  
- Run `cat data.txt` either via: 
	- Docker Desktop: Three dots beside container -> Open in Terminal.  
	OR  
	- `docker exec <CONTAINER_ID> cat data.txt`  

