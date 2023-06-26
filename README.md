# Docker's Tutorial for containers.  

**This README just documents the commands that were ran, and *hopefully* helps me remember them.**  

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

	** *OR* **  

	- `docker rm -f <CONTAINER_TO_STOP_ID>` *in one line.*  

- **Then restart updated container as in the [Starting a Container](#starting-a-container) section.**  