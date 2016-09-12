# Running Render and Mongo in a Docker Container

### Build the Docker Image
```
docker build -t render:mongo .
```

#### Ephemeral Data

To run the entire stack in a single docker container, execute:

```
docker run -d -p 8080:8080 render:mongo
```

Note that upon termination of the container (or reboot of the host), all data stored in render's mongo instance will be lost. To start a container with persistent data, see below...

#### Persistent Data

First, we need to create a Data Volume Container

```
docker create -v /data --name dbstore render:mongo /bin/true
```

With the data volume container, we can persist the mongo volume from render container to render container. Meaning, if we restart our render container (or our system reboots), the mongo data will persist thanks to the data volume container.

Now, we start a new render docker container, telling docker to mount volumes from the data volume container:

```
docker run -d -p 8080:8080 --volumes-from dbstore --name render1 render:mongo
```

#### Backups

Note that the data volume container can (and should) be periodically backed up. See the [Docker reference](https://docs.docker.com/engine/tutorials/dockervolumes/#/backup-restore-or-migrate-data-volumes) for further details.



### Attach to the running Docker container
Get the list of currently running containers using `docker ps` and find the `render:mongo` container:

```
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                          PORTS                               NAMES
21312d7ecee8        render:mongo     "/usr/bin/supervisord"   18 minutes ago      Up 18 minutes                   0.0.0.0:8080->8080/tcp, 27017/tcp   jolly_bhaskara
```

Execute a bash shell on the running container:
```
docker exec -it jolly_bhaskara /bin/bash
```
(where `jolly_bhaskara` is the container name)
