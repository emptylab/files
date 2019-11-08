
Related GitHub issue https://github.com/gocd/docker-gocd-server/issues/88

Next step: install docker in a DigitalOcean droplet. Done.

Create a Docker Droplet from DigitalOcean.

* https://cloud.digitalocean.com/marketplace/5ba19751fc53b8179c7a0071?i=7ccf73&referrer=droplets%2Fnew
* Canada
* $5/month
* SSH with existing SSH key `root@ipaddress`

Next step: install the GoCD Server on that droplet.

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
