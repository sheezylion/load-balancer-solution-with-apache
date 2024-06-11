# Load Balancer Solution With Apache
A Load Balancer (LB) distributes clients' requests among underlying Web Servers and makes sure that the load is distributed in an optimal way.

The diagram below shows the architecture of the solution

<img width="600" alt="architecture-diag" src="https://github.com/sheezylion/load-balancer-solution-with-apache/assets/142250556/907f45e3-1f07-4a84-8884-6d16d7368df6">


This project is a continuation of project: "https://github.com/sheezylion/Devops-tooling-website-solution"

We will continue using "devops-tooling-website-solution" EC2 instances to complete this project, but instead of 3 webservers we are working with 2 webservers, follow the git repo from above, configure all necessary requirements to follow along.

### Prerequisites
Ensure that the following servers are installedd and configure already.

- Two RHEL9 Web Servers
- One MySQL DB Server (based on Ubuntu 24.04)
- One RHEL9 NFS Server

### Configurations
- Apache (httpd) is up and running on both Web Servers.
- /var/www directories of both Web Servers are mounted to /mnt/apps of the NFS Server.
- All neccessary TCP/UDP ports are opened on Web, DB and NFS Servers.
- Client browsers can access both Web Servers by their Public IP addresses or Public DNS names and can open the Tooling Website (e.g, http://<Public-IP-Address-or-Public-DNS-Name>/index.php)

### Step 1 - Configure Apache As A Load Balancer
1. Create an Ubuntu Server 24.04 EC2 instance and name it Project-8-apache-lb

Result:
