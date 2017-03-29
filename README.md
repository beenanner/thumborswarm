# Swarm Mode on Docker for Mac
This is an example of thumbor in docker swarm using docker stack deploy.

Initialize the swarm cluster
- `docker swarm init`

Let preemptively pull the docker images we will be using for the demo. 
- `docker pull apsl/thumbor-nginx`
- `docker pull apsl/thumbor`

*Note: You can skip the above pull steps, but this will cause your services to take longer to start on the first deploy as it will need to pull down the images at deploy time.

Using the compose file in the repo let's deploy the services.
- `docker stack deploy -f docker-compose.yml img`

Display a list of service running.
- `docker service ls`

Test our new dynamic image resizing service which has nginx in front of it.
- Visit `http://localhost/unsafe/500x400/i.imgur.com/Q4FAyLw.png`
- Enjoy the puppy!


# Multi-host using Virtualbox

This example is the same as above but we will use multiple VMs to show how easy it is to cluster boxes. This requires [docker-machine](https://docs.docker.com/machine/install-machine/) and [virtualbox](https://www.virtualbox.org/) to be pre-installed.

We will start by using docker-machine to provision a manager for the swarm.
- `docker-machine create --driver virtualbox manager`

Now let's provision 3 more boxes that will be our worker nodes.
- `docker-machine create --driver virtualbox worker1`
- `docker-machine create --driver virtualbox worker2`
- `docker-machine create --driver virtualbox worker3`

Now we can create the cluster
- `eval $(docker-machine env manager)`
- `docker swarm init --advertise-addr eth1` 

Then run the resulting command in each worker. Here is an example output on my Mac.
- `eval $(docker-machine env worker1)`
- `docker swarm join \
    --token SWMTKN-1-0mfm860fo4snq5pt7zodvlcbh1irdbz4iixtz91kcs1my8d2hm-ets0olq240wa04agfqaf0tf9r \
    192.168.99.100:2377`
- `eval $(docker-machine env worker2)`
- `docker swarm join \
    --token SWMTKN-1-0mfm860fo4snq5pt7zodvlcbh1irdbz4iixtz91kcs1my8d2hm-ets0olq240wa04agfqaf0tf9r \
    192.168.99.100:2377`
- `eval $(docker-machine env worker3)`
- `docker swarm join \
    --token SWMTKN-1-0mfm860fo4snq5pt7zodvlcbh1irdbz4iixtz91kcs1my8d2hm-ets0olq240wa04agfqaf0tf9r \
    192.168.99.100:2377`
    
Now that we have a swarm of 4 VM let's connect back to the manager and deploy our img stack.
- `eval $(docker-machine env manager)`
- `docker stack deploy -c docker-compose.yml img`

`docker-machine ls` will give you the IP address of each node in the swarm cluster

You should be able to hit any of those IP address and enjoy the puppy
- Visit `http://192.168.99.100/unsafe/500x400/i.imgur.com/Q4FAyLw.png`

You can scale individual services by either modifying the docker-compose.yml file and running `docker stack deploy -c docker-compose.yml img` or running scaling commands as a one off `docker service scale img_nginx=3`


# Visualizer for Docker Swarm
If you are a more visual person you can run manomarks docker swarm visualizer on the manager
- `eval $(docker-machine env manager)`
- `docker run -it -d -p 8080:8080 -v /var/run/docker.sock:/var/run/docker.sock manomarks/visualizer`
- Visit `http://192.168.99.100:8080`
