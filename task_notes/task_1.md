
## Misc Notes

Related GitHub issue https://github.com/gocd/docker-gocd-server/issues/88

## Step 1: install docker in a DigitalOcean droplet. Done.

Create a Docker Droplet from DigitalOcean.

* https://cloud.digitalocean.com/marketplace/5ba19751fc53b8179c7a0071?i=7ccf73&referrer=droplets%2Fnew
* Canada
* $5/month
* SSH with existing SSH key `root@ipaddress`

## Step 2: build the GoCD Server image on that droplet. Done.

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

## Step 3: start the GoCD Server container on that droplet.

docker run -d -p8153:8153 -p8154:8154 gocd/docker-gocd-server

Then visit: http://165.22.235.94:8153/

## Step 4: build the GoCD Agent image on that droplet. Done.

```
docker build https://github.com/gocd/docker-gocd-agent-ubuntu-18.04.git -t gocd/docker-gocd-agent
```
`-t` specifies the name of the docker image 

## Step 5: start the GoCD agent container on that droplet.

Find the name of the server container

`docker container ls` (ex. serene_joliot)

Run the agent
```
docker run -d -e GO_SERVER_URL=https://$(docker inspect --format='{{(index (index .NetworkSettings.IPAddress))}}' serene_joliot):8154/go gocd/docker-gocd-agent
```
## What have we learned

The GoCD Server/Agent hangs when trying to run a simple build. We are seeing errors in the Docker logs.

## Based on our experience, what is the next step?

Keep going? Take a different approach? 

Some alternatives:

1. Install GoCD Agent and Server directly in a droplet.
1. Install Docker in a droplet and `docker run` both a GoCD Server container and GoCD Agent container.
1. Create our own container with both an GoCD Server and GoCD Agent and docker run that one container.
1. Investigate the RAM requirements for running a GoCD Server Container and Agent Container.

Related: 

https://forums.docker.com/t/minimum-hardware-requirement-to-run-docker/28072/2
https://docs.docker.com/config/containers/resource_constraints

## Investigation of the RAM requirements for running GoCD Server and GoCD Agent

This is a follow-up to the closed issue https://github.com/gocd/docker-gocd-server/issues/88

What did we do?

We installed Docker and did `docker run` on both a GoCD Server and GoCD Agent in a DigitalOcean droplet. We were able to connect to the GoCD Server through the web browser and to configure a simple pipeline. The pipeline started running on the GoCD Agent. This was all good.

What went wrong?

The pipeline hung and did not complete. 

**Question**: We think the hang might be due to RAM requirements. The droplet had 1 GB RAM. 

We have seen this forum post on docker hardware requirements: https://forums.docker.com/t/minimum-hardware-requirement-to-run-docker/28072/2. It says: "Minimum for “comfortable” usage - 2GB'. We have not seen official documentation on this, though.

We have also read this GoCD documentation on hardware requirements: https://docs.gocd.org/current/installation/system_requirements.html. It says: 

GoCD Server requirements

* RAM - minimum 1GB, 2GB recommended
* CPU - minimum 2 cores, 2GHz
* Disk - minimum 1GB free space

GoCD Agent requirements

* RAM - minimum 128MB, 256MB recommended
* CPU - minimum 2GHz

Based on the above, a droplet with 2 GB might be sufficient.

