# AWS CloudFormation Templates

This repository gathers some CloudFormation templates I've coded, parameterized for your own use. Work in progress. Feedback welcome.

## docker-swarm-cluster.json
Creates a totally independant, isolated Docker Swarm cluster in your AWS account.
### Parameters
When you submit the template to AWS it will ask you to fill out the following:
- KeyPair: This EC2 keypair will be added to all the instances in the cluster, allowing you to log in for maintenance or training purposes.
- AvailabilityZone: Where to create the cluster. By now all the instances - directors and nodes - live in the same availability zone. In the future I might consider spreading them around 3 different zones in the same AWS region.
- DirectorInstanceType: EC2 instance type for the 3 directors - for exampe, t2.nano
- DirectorInstanceRole: The name of the IAM role that will be assigned to the director instances. For CloudWatch logging to work, give this role CloudWatch write access.
- NodeInstanceType: EC2 instance type for the cluster nodes - for example, m4.large
- NodeInstanceRole: The name of the IAM role that will be assigned to the node instances. For CloudWatch logging to work, give this role CloudWatch write access.
- NodesMin: Minimum number of nodes to keep alive at any time.
- NodesMin: Maximum number of nodes to scale up to.
### Resources
When you submit it to Amazon AWS, this template will create the following resources in your AWS account:
- 1 VPC. All EC2 instances and network traffic are isolated here. The VPC works with the IP Range 10.0.0.0/16, but you don't need to worry about that.
- 2 Subnets. They live in the VPC, one is for the 3 director instances and the other is for the nodes. The nodes subnet assigns nodes IPs between 10.0.0.1 and 10.0.127.254, so this subnet's mask is effectively 10.0.0.0/17. The directors subnet is much smaller, ranging from 10.0.128.1 to 10.0.128.254, with a submask of 10.0.128.0/24. In the directors subnet DHCP isn't used; all nodes will have known IPs. Also, both subnets will provide one public IP to all the instances.
- 1 Internet gateway. It will be linked to both subnets, allowing Internet access.
- 2 Security Groups, one for directors and another one for the nodes. By default outbound traffic is unrestricted, and inbound traffic is only allowed on port 22. For a production deployment I would close port 22 and force people to join the VPC to login to the servers.
- 3 EC2 instances, as Directors. They will be loaded with the necessary userdata and run the Consul service and Docker Swarm manager services.
- 1 EC2 AutoScaling group, which will initially grow to reach your minimum desired nodes. The autoscaling group will create new EC2 instances as needed, initializing the Consul client and connecting it to the Directors, and starting the Docker Swarm agent there.
