# Load Balancer Solution With Apache
A Load Balancer (LB) distributes clients' requests among underlying Web Servers and makes sure that the load is distributed in an optimal way.

The diagram below shows the architecture of the solution

<img width="600" alt="architecture-diag" src="https://github.com/sheezylion/load-balancer-solution-with-apache/assets/142250556/907f45e3-1f07-4a84-8884-6d16d7368df6">


This project is a continuation of project: "https://github.com/sheezylion/Devops-tooling-website-solution"

We will continue using "devops-tooling-website-solution" EC2 instances to complete this project, but instead of 3 webservers we are working with 2 webservers, follow the git repo from above, configure all necessary requirements to follow along.

NOTE: The load balancer solution can be implemented with 2 or more web servers.

### Uniform Resource Locators, IP Addresses and Domain Name System
When we access a website on the Internet we use a URL (Uniform Resource Locator). We do not really know how many servers are out there serving our requests, this complexity is hidden from a regular user, but in case of websites that are being visited by millions of users per day (like Google or Reddit) it is impossible to serve all the users from a single web server (it is also applicable to databases, but for now we will not focus on distributed DBs).

Each URL contains a domain name part, which is translated (resolved) to the IP address of a target server that will serve requests when we open a website on the Internet.

NOTE: Broadly speaking, Internet Protocol (IP) address is a unique number that identifies servers and computers on a network. It is assigned by your ISP (Internet Service Provider) to your computer.

Translation (resolution) of domain names is performed by Domain Name System (DNS) servers. The most commonly used one has a public IP address 8.8.8.8 and belongs to Google. We can try to query it with the nslookup command:

```
nslookup 8.8.8.8
```

Result:


<img width="666" alt="Screenshot 2024-06-11 at 21 14 32" src="https://github.com/sheezylion/load-balancer-solution-with-apache/assets/142250556/71990160-b18a-409b-a52c-a0689147e756">

### Load Balancers and Scalability
When we just have one web server and load increases, we want to serve more and more customers.

To distribute increased traffic, we can add more CPU and RAM or completely replace the server with a more powerful one:

This is called "vertical scaling". However, vertical scalling has limitations because at some point we will reach the maximum capacity of CPU and RAM that can be installed into our server.
We can also distribute increased traffic across multiple web servers:

This is called "horizontal scaling" which consists in adapting the current load by adding (scaling out) or removing (scaling in) web servers.

Horizontal scalling is much more common because it can be applied almost seamlessly and almost infinitely (you can imagine how many servers Google has to serve billions of search requests).

Horizontal scaling allows to know the adjustment of number of servers that we need which can be done manually or automatically (for example, based on some monitored metrics like CPU and Memory Load).


NOTE: the property of a system (in our case it is web tier) that handles growing load by adding resources, is called "Scalability".

In our set up for project "devops-tooling-website-solution" we had 3 web servers and each of them had its own public IP address and public DNS name. So, in order for web clients to access our web servers they would need to use 3 different URLs that would contain 3 different DNS names or 3 different IP addresses. Now, it would not be a nice user experience to remember 3 different IP addresses or 3 different DNS names to make requests to 3 different servers that serve the same website, let alone, imagine doing this with millions of Google servers:

So, in order to hide all of this complexity from the regular user we can use a Load Balancer (LB) to have a single point of access with a single public IP address or DNS name.

A LB distributes web clients’ requests among underlying web servers and makes sure that the load is distributed in an optimal way.

### Prerequisites
Before starting, we need to make sure that we have the following servers installed and configured based on the instruction of the following GitHub repository: https://github.com/sheezylion/Devops-tooling-website-solution

NOTE: We used 3 web servers on project: "devops-tooling-website-solution". However, for simplicity we will use 2 web servers instead for this project.

- Two RHEL9 Web Servers
- One MySQL DB Server (based on Ubuntu 24.04)
- One RHEL9 NFS Server

### Prerequisites Configurations
- Apache (httpd) is up and running on both Web Servers.
- /var/www directories of both Web Servers are mounted to /mnt/apps of the NFS Server.
- All neccessary TCP/UDP ports are opened on Web, DB and NFS Servers.
- Client browsers can access both Web Servers by their Public IP addresses or Public DNS names and can open the Tooling Website (e.g, http://<Public-IP-Address-or-Public-DNS-Name>/index.php)

### Step 1 - Configure Apache As A Load Balancer

1. Create an Ubuntu Server 24.04 EC2 instance and name it Project-8-apache-lb

Result:

<img width="1662" alt="Screenshot 2024-06-13 at 17 23 25" src="https://github.com/sheezylion/load-balancer-solution-with-apache/assets/142250556/c90eefea-f9ad-4a1f-98dd-6240c2123f77">

2. Open TCP port 80 on Project-8-apache-lb by creating an Inbounb Rule in Security Group

Result:

<img width="1650" alt="Screenshot 2024-06-13 at 17 24 58" src="https://github.com/sheezylion/load-balancer-solution-with-apache/assets/142250556/b19452fc-5e1a-41b6-9827-76bc9bfcfb4f">

3. Instal Apache Load Balancer on Project-8-apache-lb and configure it to point traffic coming to LB to both Web Servers.

i. Install Apache2

- Access the instance

```
 ssh -i ~/Downloads/demo-pair.pem ec2-user@35.172.214.34
```

Result:

<img width="847" alt="Screenshot 2024-06-13 at 17 26 45" src="https://github.com/sheezylion/load-balancer-solution-with-apache/assets/142250556/7fe8fdec-3eb3-4f36-bdff-6a489ff8f7e2">




