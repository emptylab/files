# Alternatives:

- Password
- LDAP
- GitHub auth

# Resources:

* https://docs.gocd.org/current/configuration/dev_authentication.html
* https://www.gocd.org/plugins/#authorization
* https://github.com/gocd-contrib/github-oauth-authorization-plugin/blob/master/INSTALL.md

# Action: Spend 20-minutes setting up GitHub Auth for GoCD Server

## Installation

Clone and build the plugin source
```
git clone git@github.com:gocd-contrib/github-oauth-authorization-plugin.git
cd github-oauth-authorization-plugin/
./gradlew clean test assemble
```

Start the SSH agent and load the SSH key
```
eval `ssh-agent -s`
ssh-add ~/.ssh/emptylab
```

SCP the build result (.jar) to the droplet
```
scp build/libs/github-oauth-authorization-plugin-3.0.2-74.jar root@165.22.235.94:/tmp
```

SSH into the droplet that contains the GoCD server & agent
```
ssh root@165.22.235.94
```

Copy the build result (.jar) to the docker container
```
docker cp /tmp/github-oauth-authorization-plugin-3.0.2-74.jar <containerId>:/go-working-dir/plugins/external
```

Optionally: check that the copy worked by opening bash in the GoCD container (as the go user)
```
docker ps
docker exec -it -u go -w /home/go <container_id> /bin/bash
cd /go-working-dir/plugins/external
ls
exit
```

Restart the GoCD server 
```
docker container restart <containerId>
```

After restart completes, check that the plugin is available: http://165.22.235.94:8153/go/admin/plugins

## Configuration

Following the remaining instructions here for configuration: 
https://github.com/gocd-contrib/github-oauth-authorization-plugin/blob/master/INSTALL.md#configuration


