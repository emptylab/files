
## Misc Notes

Related GitHub issue https://github.com/gocd/docker-gocd-server/issues/88

Digital Ocean Docker Droplet Welcome Email: https://go.digitalocean.com/index.php/email/emailWebview?aliId=j0hXccfLLgkP5D0HkpLrQN6A6E6oH%2BXE2dEPaZqBu6ZdpS8KDpW%2BTw%3D%3D

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

## Step 3: start the GoCD Server container on that droplet. Done.

```
docker run -d -p8153:8153 -p8154:8154 gocd/docker-gocd-server
```

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
1. Investigate the RAM requirements for running a GoCD Server Container and Agent Container. Done.
1. Retry the above steps with 2 GB RAM. Done.

Related: 

* https://forums.docker.com/t/minimum-hardware-requirement-to-run-docker/28072/2
* https://docs.docker.com/config/containers/resource_constraints

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

## Retry the above steps with 2 GB RAM. Done.

The pipeline succeeded. :-)

## Step 6: Build an ASP.NET Core Docker image. 

There are basic instructions here: https://docs.docker.com/engine/examples/dotnetcore/

There is a sample here: https://github.com/dotnet/dotnet-docker/tree/master/samples/aspnetapp

We emulated that sample like this: 

```
$ cd ~/mylocalfarm/mylocalfarm/
$ docker build --pull -t farm-land-lasanga .
```

This did not work until installing NodeJS in the Dockerfile. 

For `apt-get` to work in the Dockerfile, I needed to update my system clock.

On WSL this is one approach:

```
sudo dpkg-reconfigure tzdata // select America > Vancouver
```

Then restart Docker from Windows.

## Step 7: Install Docker in the GoCD Agent

I opened an issue about that here: https://github.com/gocd/docker-gocd-agent-ubuntu-18.04/issues/2

```
# run the GoCD agent image (named hungry_lalande)
$ docker run -d gocd/docker-gocd-agent

# attach to the GoCD agent container as root
$ sudo docker exec --user root -i -t hungry_lalande /bin/bash

# install docker in the GoCD agent
root@2308f39c23fb:/# curl -fsSL https://get.docker.com -o get-docker.sh
root@2308f39c23fb:/# sh get-docker.sh

# add the GoCD user to the docker group
root@2308f39c23fb:/# usermod -aG docker go

# start docker
root@2308f39c23fb:/# service docker start 

# exit 
root@2308f39c23fb:/# exit

# save the new container
$ docker commit hungry_lalande gocd/docker-gocd-agent-with-docker
```

## Step 8: Build the mylocalfarm/Dockerfile from GoCD in Digital Ocean. Done.

This mostly worked, but the GoCD server would crash. It might be related to resource constraints. So, we have constrained the containers like this: 

```
docker [build|run] --memory=500m --memory-swap=-1 --cpu-period=100000 --cpu-quota=50000 ...
```

That means the build/run command can use up to 500 MB of RAM and only 50% of the CPU time. The percentage of CPU time is calculated as quote/period. The period must be between 100 and 10,000 microseconds; choose the period based on how much flexibility to give the scheduler.

Resource     | Minimum RAM     | CPU quota/period
--           |                 |
GoCD Server  | 1 GB - 2 GB     | 
GoCD Agent   | 128 MB - 256 MB | 

The `npm run build` takes its memory in part from `--max-old-space-size=`

For more on `--max-old-space-size` see 
* https://erikcorry.blogspot.com/2012/11/memory-management-flags-in-v8.html
* https://stackoverflow.com/questions/48387040/nodejs-recommended-max-old-space-size

This is the command that I have not been able to run due to memory constraints.
```
docker build --build-args NODE_OPTIONS="--max-old-space-size=325" --memory=900m --memory-swap=-1 --cpu-period=100000 --cpu-quota=50000 --no-cache --tag farm_app_image:latest --file Dockerfile .
```

### Try to use Swap

https://www.linux.com/news/all-about-linux-swap-space/

* How can we use the swap?
* How much swap space does the machine have?
  * Swap file - special file
  * Swap partition - partition soley for swapping
* What if `swapon -s` has no output?

My SO Questions: 

* https://stackoverflow.com/q/58895103/1108891 
* https://stackoverflow.com/q/58894667/1108891


The final solution was to optimize the npm run build command.

## Step 9: Push the Docker image to the production host

Add SSH key to GoCD server: https://docs.gocd.org/current/faq/docker_container_ssh_keys.html


In the GoCD Machine

```
docker save farm_app_image > farm_app_image.tar     
scp farm_app_image.tar root@138.197.140.250:/tmp    
```

In the Remote Server
```
ssh root@138.197.140.250 "cd /tmp; docker stop $(docker ps -q); docker load -i farm_app_image.tar; docker run -d --name farm_app_container -p 80:80 farm_app_image"
```

# Rebuild the World (work in progress)

```
# ssh into the droplet that contains the GoCD server & agent

eval `ssh-start -s`
ssh-add ~/.ssh/emptylab
ssh root@165.22.235.94

# stop and prune all containers

docker ps -q | xargs -r docker stop
docker system prune

# start the GoCD server

docker run -d -p8153:8153 -p8154:8154 gocd/docker-gocd-server

# catpure the GoCD server IP Address

GO_SERVER_IP=$(docker inspect --format='{{(index (index .NetworkSettings.IPAddress))}}' $(docker ps -q))
echo $GO_SERVER_IP

# start a GoCD agent, 
# binding it to the host's docker.sock, 
# and giving it the GoCD Server's IP

docker run -v /var/run/docker.sock:/var/run/docker.sock -d -e GO_SERVER_URL=https://$GO_SERVER_IP:8154/go gocd/docker-gocd-agent

# Checkpoint
# Visit http://165.22.235.94:8153/ in a browser
# 1. Check that the server loads.
# 2. Check that an agent is available

# Restore the config
# Copy the `~/emptylab/files/task_notes/task_1/gocd_config_20191611.xml`
# Paste it here http://165.22.235.94:8153/go/admin/config_xml
# Important: do NOT paste over the opening <server> tag.

# setup SSH for the GoCD server

# exec into the GoCD container (as the go user)
docker exec -it -u go -w /home/go <container_id> /bin/bash

# generate an `~/.ssh/id_rsa` and `~/.ssh_rsa.pub` key (for the go user)
ssh-keygen -t rsa -b 4096 -C gocd-server

# copy the public key and then paste it into DigitalOcean
cat ~/.ssh/id_rsa.pub

# add the mylocal.farm IP as a trusted host
ssh-keyscan 138.197.140.250 > /home/go/.ssh/known_hosts

# ssh into the mylocal.farm IP
ssh root@138.197.140.250 

# paste the public key to the trusted keys
vim ~/.ssh/authorized_keys

# test the connection
ssh go@138.197.140.250 

# other helpful commands

# get the id of the GoCD server container
echo $(docker ps -q --filter ancestor=gocd/docker-gocd-server)
```
