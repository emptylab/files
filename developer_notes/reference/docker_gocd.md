
# Useful Docker commands
You can find the container ID, image name, instance name, ports, status etc..  using `docker ps`.

# Getting into the container
Sometimes, you need a shell inside the container, docker provides an easy way to do that:
```
docker exec -i -t CONTAINER-ID bash
```
To check the server logs, you can do this:
```
docker exec -i -t CONTAINER-ID tail -f /godata/logs/go-server.log
```

# Exiting the container
The `exit` command terminates your session.  

Because the exec command opens a new TTY terminal you must enter `Ctrl^P^Q` to switch back to your previous terminal. 
