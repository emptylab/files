
Problem: Deployment is failing with the following error message:

```
[go] Task: /bin/sh -c "ssh root@138.197.140.250 \"cd /tmp; docker ps -q | xargs docker stop; docker system prune -f; docker load -i farm_app_image.tar; docker run -d --name farm_app_container -p 80:80 farm_app_image\""took: 2.989sexited: 125
    "docker stop" requires at least 1 argument.
    See 'docker stop --help'.

    Usage:  docker stop [OPTIONS] CONTAINER [CONTAINER...]

    Stop one or more running containers
    Deleted Containers:
    506d69cc20559d0baa027f8eae35bd9448bfeb546b9ae7b4ee208258a11e2550

    Deleted Images:
    deleted: sha256:e926b7a095b87a2c45a606c9f7db9b9b7f1630560e817e268733b3952f889238
    deleted: sha256:a3a1f8390fabc34d26ced9d4dea44e4244ca8d1b725f6fde0200ee2ce90428d9
    deleted: sha256:cc8fb973cd5dc927f124bec6cabd3fb003fb0e9ac2094e969642f979392d53e3

    Total reclaimed space: 2.167MB
    The image farm_app_image:latest already exists, renaming the old one with ID sha256:69fac2b2d3f85d328d5e2822d749c58772ce4ad09cd2598a8ab2920f0defce1c to empty string
    Loaded image: farm_app_image:latest
    93b80c93285085dfe53c9ec4c1dc6fc7d1f77096006d1bad2fd04bb431eab500
    docker: Error response from daemon: driver failed programming external connectivity on endpoint farm_app_container (3701ddae579e103bcf7f9a61dd5bd65bfe843d8ae06b79af7da59f176d638ab3): Error starting userland proxy: listen tcp 0.0.0.0:80: bind: address already in use.
    [go] Task status: failed, took: 2.989s, exited: 125
```

Cause: The `docker run` command was mapping the container's port `80` to the host's port `80`, which is already in use by NGINX.

Solution: Map the container's port `80` to the hosts port `8080` instead like this: 

```
-p 8080:80
```

See https://docs.docker.com/config/containers/container-networking/