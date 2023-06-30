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

## Volumes  
- Provides ability to connect specific filesystem paths of container back to host machine.  
- Two main volume types: **Named Volumes** and **Bind Mounts**  
	- **Named Volumes**: Buckets of data.

### Persisting data (Named Volumes)
- On this app, data is stored at `/etc/todos/todo.db` in an SQLite DB. They are stored in a single file.  
- `docker volume create todo-db`  *same **todo-db** as below*
- `docker run -dp 3000:3000 -v todo-db:/etc/todos getting-started`  

	- to check:  
		- Add some task(s).  
		- Remove the container, and rerun **second command** above.  
		- Task should be persisting between containers.

### Bind Mounts  
- We have control of exact mountpoint on host.  
	- Can be used to persist data.  
	- But often used to provide additional data into containers.  

- In the root of the app's directory:  
```	
docker run -dp 3000:3000 \
	-w /app -v "$(pwd):/app" \
	node:18-alpine \
	sh -c "yarn install && yarn run dev"
```
- Explanation: 
	- `w /app` *sets container's present working directory.*
	- `-v "$(pwd):/app` *(pwd) is host's working dir, which is the app. Binds (pwd) to container's /app dir.*  
	- `node:18-alpine` *image to use*  
	- `sh -c "yarn install && yarn run dev"` *this command runs on /app. Runs a shell (since alpine doesn't have a bash) and runs commands to install dependencies as well as run the **dev** script*  
	- `yarn run dev` runs `nodemon src/index.js` as seen in **package.json**.
	- Logs from above can be seen with `docker logs -f <CONTAINER_ID>`.

## Multi Container Applications  

- **"Each container should do one thing and do it well."**
- **Networking** will be used for inter container communication.

### Network creating (2 ways)
1. Assign at start.
2. Connect to existing container.

#### Doing 1. Creating the network first then attach the container.

##### Network and MySQL container creation.
- `docker network create todo-app` *creates network*
- Then: 
```
docker run -d \
	--network todo-app --network-alias mysql \
	-v todo-mysql-data:/var/lib/mysql \
	-e MYSQL_ROOT_PASSWORD=secret \
	-e MYSQL_DATABASE=todos \
	mysql:8.0
```
- Explanation:
	- Starts a MySQL container, and attaches it to created network.
	- `-v todo-mysql-data:/var/lib/mysql` *makes a `todo-mysql-data` volume and mounts it at `/var/lib/mysql` (place where MySQL stores data).*
	- Also creates some environment variables `-e`.
	- `docker volume create` not needed since Docker recognizes need and creates one automatically.

- `docker exec -it <CONTAINER_ID> mysql -p` *the `-p` flag, propmts for password.*
- After entering password, you'll be inside of mysql's terminal.
- `SHOW DATABASES;` *will confirm our database is working.*
- `exit` *to exit mysql terminal.*

##### Connecting to MySQL's container.
- `docker run -it --network todo-app nicolaka/netshoot` *use another container `nicolaka/netshoot` that contains a lot of tools for networking issues.*
- Once inside of netshoot: use `dig mysql` to look up the IP address.
	- `mysql` is from the `--network-alias` flag when we [started the MySQL database](#network-and-mysql-container-creation).
- Note/remember IP, then `exit` netshoot when ready.

## Finally running App container with MySQL container.
- Environment variables are ok in development but NOT OK in production.
- Command below specificies environment variables, also connects container to the network.
```
docker run -dp 3000:3000 \
	-w /app -v "$(pwd):/app" \
	--network todo-app \
	-e MYSQL_HOST=mysql \
	-e MYSQL_USER=root \
	-e MYSQL_PASSWORD=secret \
	-e MYSQL_DB=todos \
	node:18-alpine \
	sh -c "yarn install && yarn run dev"
```
- Explanations:
	- connects to `todo-app` network.
	- `MYSQL_HOST` *hostname for the running MySQL server. `mysql` alias resolves to the runnign MySQL server.*
	- `MYSQL_DB` *the database to use once connected.*
	- Rest is hopefully intuitive, or has been explained before.

- `docker exec -it <CONTAINER_ID> mysql -p todos` *runs MySQL terminal.*
- Once inside `select * from todo_items;`