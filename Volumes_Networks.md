## Docker Volumes and Networks - Beginner's Guide

Docker volumes persist data outside containers, while networks enable containers to communicate with each other. This guide provides simple, complete examples you can run on your Windows PC.[1][2]

## Docker Volumes

### Creating and Using a Volume

**Step 1:** Open Command Prompt or PowerShell on your Windows PC.[1]

**Step 2:** Create a volume:
```
docker volume create my-data
```

**Step 3:** List all volumes to verify creation:
```
docker volume ls
```

**Step 4:** Run a container with the volume attached:
```
docker run -it --name my-container --mount source=my-data,destination=/data ubuntu
```

This creates an Ubuntu container with the volume mounted at `/data` inside the container. Any files you create in `/data` will persist even after the container is deleted.[2]

### Complete Working Example

**Step 1:** Create volume and run container:
```
docker volume create demo-vol
docker run -it --name test1 --mount source=demo-vol,destination=/app ubuntu bash
```

**Step 2:** Inside the container, create a test file:
```
echo "Hello from container" > /app/myfile.txt
exit
```

**Step 3:** Remove the container and create a new one with the same volume:
```
docker rm test1
docker run -it --name test2 --mount source=demo-vol,destination=/app ubuntu bash
cat /app/myfile.txt
```

You'll see "Hello from container" - the data persisted.[2][1]

### Volume Management Commands

- **Inspect volume details:** `docker volume inspect my-data`[1]
- **Remove a volume:** `docker volume rm my-data`
- **Remove all unused volumes:** `docker volume prune`[3]

## Docker Networks

### Creating and Using a Network

**Step 1:** Create a custom bridge network:
```
docker network create my-network
```

**Step 2:** List all networks:
```
docker network ls
```

**Step 3:** Run containers on the network:
```
docker run -d --name web1 --network my-network nginx
docker run -d --name web2 --network my-network nginx
```

These containers can now communicate using their container names.[4][3]

### Complete Working Example

**Step 1:** Create network and run two containers:
```
docker network create demo-network
docker run -it --name container1 --network demo-network busybox sh
```

**Step 2:** In a new Command Prompt window, run a second container:
```
docker run -it --name container2 --network demo-network busybox sh
```

**Step 3:** From container1, ping container2 by name:
```
ping container2
```

The containers can communicate using their names as hostnames.[3]

### Network Management Commands

- **Connect existing container to network:** `docker network connect demo-network container1`[3]
- **Disconnect container from network:** `docker network disconnect demo-network container1`[3]
- **Remove a network:** `docker network rm demo-network`[3]
- **Remove all unused networks:** `docker network prune`[3]

[1](https://docs.docker.com/engine/storage/volumes/)
[2](https://earthly.dev/blog/docker-volumes/)
[3](https://www.datacamp.com/tutorial/docker-networking)
[4](https://docs.docker.com/reference/cli/docker/network/create/)
[5](https://www.youtube.com/watch?v=dpVaSUamHto)
[6](https://www.docker.com/101-tutorial/)
[7](https://www.baeldung.com/ops/docker-volumes)
[8](https://stackoverflow.com/questions/66624752/best-practice-docker-volumes-on-windows)
[9](https://www.tutorialspoint.com/docker-add-network-drive-as-volume-on-windows)
[10](https://stackoverflow.com/questions/42287408/how-to-mount-network-volume-in-docker-for-windows-windows-10)
