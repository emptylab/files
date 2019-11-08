
## Misc Notes

Related GitHub issue https://github.com/gocd/docker-gocd-server/issues/88

## Step 1: install docker in a DigitalOcean droplet. Done.

Create a Docker Droplet from DigitalOcean.

* https://cloud.digitalocean.com/marketplace/5ba19751fc53b8179c7a0071?i=7ccf73&referrer=droplets%2Fnew
* Canada
* $5/month
* SSH with existing SSH key `root@ipaddress`

## Step 2: install the GoCD Server on that droplet.

* docker pull gocd/gocd-server
* docker build git@github.com:gocd/docker-gocd-server.git (and then wait quite some time)

Some error messages related to locale set up.

```
failed to set locale!                                                
[warning] No definition for LC_PAPER category found                  
failed to set locale!                                                
[warning] No definition for LC_NAME category found                   
failed to set locale!                                                
[warning] No definition for LC_ADDRESS category found                
failed to set locale!                                                
[warning] No definition for LC_TELEPHONE category found              
failed to set locale!                                                
[warning] No definition for LC_MEASUREMENT category found            
failed to set locale!                                                
[warning] No definition for LC_IDENTIFICATION category found         
```

## Step 3: start the GoCD Server

docker run -d -p8153:8153 -p8154:8154 gocd/docker-gocd-server

Then visit: http://165.22.235.94:8153/

## Step 4: install/start a GoCD agent

```
docker build https://github.com/gocd/docker-gocd-agent-ubuntu-18.04.git -t gocd/docker-gocd-agent
```

Rename the docker instant -t appropriate-agent-name 

## Step 5:  Start a GoCD agent

Find the name of the server container
`docker container ls` (ex. serene_joliot)

Run the agent
```
docker run -d -e GO_SERVER_URL=https://$(docker inspect --format='{{(index (index .NetworkSettings.IPAddress))}}' serene_joliot):8154/go gocd/docker-gocd-agent
```
