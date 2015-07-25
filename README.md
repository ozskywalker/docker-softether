# Docker image for SoftEther VPN

Credit to cnf, this image is a fork from https://github.com/cnf/docker-softether

Will deploy a fully functional [SoftEther VPN](https://www.softether.org) server as a docker image and is intended for deployment on Amazon Web Service's EC2 Container Service (ECS), but could be deployed anywhere.

![overview diagram](https://raw.githubusercontent.com/failathon/docker-softether/master/misc/overview.png)

## Download

    (todo) docker pull failathon/softether

## Run in ECS

### Create a ECS Task Definition

#### Container Definition

1. set Container Name to desired name
2. set Image to failathon/softether
3. set CPU units to 500
4. set Memory to 750
5. enable Essential
6. set Port Mappings to 443:443 TCP (Host:Container) and 1194:1194 UDP
        
#### EC2 Instance for ECS 101
For those who have not used ECS before, ensure that:

* Use an ECS-optimized AMI (ie. ami-27212417 in Community AMIs)
* IAM role selected should contain:
  * AmazonEC2ContainerServiceFullAccess
  * AmazonEC2ContainerServiceRole
  * Trusted Entity is ec2.amazonaws.com
* *Security Group* must allow SSH (22/tcp) and HTTPS (443/tcp) are allowed for Inbound traffic
* Set *User Data* is set to
```bash
#!/bin/bash
echo ECS_CLUSTER=cluster_name_here >> /etc/ecs/ecs.config
```        

### Cluster Creation
1. Create a cluster with any name you prefer
2. Within the cluster, now create a service, selecting the task definition you created before (i.e. softether-task:1)
3. Set the service name to a name you prefer (i.e. softether-service)
4. Set the number of tasks to 1 - this will create a single VPN service hosted on a single 
5. Leave the Load Balancer set to __No ELB__

Wait for ECS to spawn your new service


### First-user creation
* Login to the EC2 instance as ec2-user
* 


## Running from EC2

Simplest version:

    docker run -d --net host --name softether failathon/softether

With external config file:

    touch /etc/vpnserver/vpn_server.config
    docker run -d -v /etc/vpnserver/vpn_server.config:/usr/local/vpnserver/vpn_server.config --net host --name softether failathon/softether

If you want to keep the logs in a data container:

    docker run -d --name softether-logs --volume /var/log/vpnserver busybox:latest /bin/true
    docker run -d --net host --name softether --volumes-from softether-logs failathon/softether

All together now:

    touch /etc/vpnserver/vpn_server.config
    docker run -d --name softether-logs --volume /var/log/vpnserver busybox:latest /bin/true
    docker run -d -v /etc/vpnserver/vpn_server.config:/usr/local/vpnserver/vpn_server.config --volumes-from softether-logs --net host --name softether failathon/softether

