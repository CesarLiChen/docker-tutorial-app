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


