# Getting into the container

Sometimes, you need a shell inside the container, docker provides an easy way to do that:
```
docker exec -i -t CONTAINER-ID bash
```
To check the server logs, you can do this:
```
docker exec -i -t CONTAINER-ID tail -f /var/log/go-server/go-server.log
```
You can find the container ID using `docker ps`.
