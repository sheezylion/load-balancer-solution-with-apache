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

<img width="1670" alt="Screenshot 2024-06-13 at 17 30 47" src="https://github.com/sheezylion/load-balancer-solution-with-apache/assets/142250556/9465d0fb-5828-4f13-a56f-f901380cae72">


2. Open TCP port 80 on Project-8-apache-lb by creating an Inbound Rule in Security Group

Result:

<img width="1635" alt="Screenshot 2024-06-13 at 17 31 28" src="https://github.com/sheezylion/load-balancer-solution-with-apache/assets/142250556/1dfdccb2-5695-4b93-9275-cc775ed29576">

3. Instal Apache Load Balancer on Project-8-apache-lb and configure it to point traffic coming to LB to both Web 
Servers.

i. Install Apache2

- Access the instance

```
 ssh -i ~/Downloads/demo-pair.pem ubuntu@34.228.142.236
```

Result:

<img width="853" alt="Screenshot 2024-06-13 at 17 32 19" src="https://github.com/sheezylion/load-balancer-solution-with-apache/assets/142250556/222ea430-c5f1-4c36-a646-d1a6a08e51d5">

- Update and upgrade Ubuntu

```
sudo apt update && sudo apt upgrade
```

Result:

<img width="837" alt="Screenshot 2024-06-13 at 17 32 56" src="https://github.com/sheezylion/load-balancer-solution-with-apache/assets/142250556/356711fd-3d64-4374-be3a-d31ea76d85a3">

- Install Apache

```
sudo apt install apache2 -y
```

Result:

<img width="860" alt="Screenshot 2024-06-13 at 17 40 29" src="https://github.com/sheezylion/load-balancer-solution-with-apache/assets/142250556/264010a0-f1fc-4926-99c9-9aa24ed2ff09">

```
sudo apt-get install libxml2-dev
```

Result:

<img width="843" alt="Screenshot 2024-06-13 at 17 41 24" src="https://github.com/sheezylion/load-balancer-solution-with-apache/assets/142250556/721f736b-e5c1-41cc-a509-e327ce4eb917">

ii. Enable the following modules

```
sudo a2enmod rewrite

sudo a2enmod  proxy

sudo a2enmod  proxy_balancer

sudo a2enmod  proxy_http

sudo a2enmod  headers

sudo a2enmod  lbmethod_bytraffic
```

Result:

<img width="847" alt="Screenshot 2024-06-13 at 17 42 21" src="https://github.com/sheezylion/load-balancer-solution-with-apache/assets/142250556/d42dc8c2-5df1-4bcc-b7b4-483c1b45d034">

iii. Restart Apache2 Service

```
sudo systemctl restart apache2
sudo systemctl status apache2
```

Result:

<img width="854" alt="Screenshot 2024-06-13 at 17 43 09" src="https://github.com/sheezylion/load-balancer-solution-with-apache/assets/142250556/e448bfee-3d23-45d3-aab8-8c51bc9a6f4e">


### Configure Load Balancing

i. Open the file 000-default.conf in sites-available

```
sudo vi /etc/apache2/sites-available/000-default.conf
```

ii. Add this configuration into the section <VirtualHost *:80>  </VirtualHost>

```
<Proxy “balancer://mycluster”>
            BalancerMember http://<webserver1-private-ip-address>:80 loadfactor=5 timeout=1
           BalancerMember http://<webserver2-private-ip-address>:80 loadfactor=5 timeout=1
           ProxySet lbmethod=bytraffic
           # ProxySet lbmethod=byrequests
</Proxy>


ProxyPreserveHost on
ProxyPass / balancer://mycluster/
ProxyPassReverse / balancer://mycluster/
```

Result:

<img width="852" alt="Screenshot 2024-06-13 at 17 49 01" src="https://github.com/sheezylion/load-balancer-solution-with-apache/assets/142250556/99288994-fb65-4d56-be21-9de4bd7c5746">

iii. Restart Apache

```
sudo systemctl restart apache2
```

Result:

<img width="557" alt="Screenshot 2024-06-20 at 22 03 06" src="https://github.com/sheezylion/load-balancer-solution-with-apache/assets/142250556/8164a11a-c5db-4ef2-a92d-04f0902b89bf">

**bytraffic** balancing method with distribute incoming load between the Web Servers according to currentraffic load. The proportion in which traffic must be distributed can be controlled bt **loadfactor** parameter.

Other methods such as **bybusyness**, **byrequests**, **heartbeat** can also be adopted.

### 4. Verify that the configuration works

i. Access the website using the LB's Public IP address or the Public DNS name from a browser

Results:

<img width="1665" alt="Screenshot 2024-06-20 at 22 05 35" src="https://github.com/sheezylion/load-balancer-solution-with-apache/assets/142250556/aa85b82d-83d8-412e-9edd-7b1bce271d85">

<img width="988" alt="Screenshot 2024-06-20 at 22 06 52" src="https://github.com/sheezylion/load-balancer-solution-with-apache/assets/142250556/338059fa-17ed-41de-97f3-636f45e80189">


<img width="1680" alt="Screenshot 2024-06-20 at 22 23 21" src="https://github.com/sheezylion/load-balancer-solution-with-apache/assets/142250556/24e50f34-c102-4329-b07f-d0870b8ca966">



Note: If in the previous project, /var/log/httpd was mounted from the Web Server to the NFS Server, unmount them and ensure that each Web Servers has its own log directory.

ii. Unmount the NFS directory
- Check if the Web Server's log directory is mounted to NSF

```
df -h
sudo umount -f /var/log/httpd
```

Note: In the image below it is not mounted so we can proceed.

Result:

<img width="560" alt="Screenshot 2024-06-20 at 22 13 03" src="https://github.com/sheezylion/load-balancer-solution-with-apache/assets/142250556/84f58ff9-ef00-4e15-8cf5-cba27e23676a">

iii. Open two ssh consoles for both Web Server and run the command:

```
sudo tail -f /var/log/httpd/access_log
```

iv. Refresh the browser page several times and ensure both Web Servers receive HTTP and GET requests. New records must apear in each web server log files. The number of request to each servers will be approximately the same since loadfactor is set to the same value for both servers. This means that traffic will be evenly distributed between them.

Result:

<img width="1680" alt="Screenshot 2024-06-20 at 22 19 21" src="https://github.com/sheezylion/load-balancer-solution-with-apache/assets/142250556/53153382-b2d8-4816-8049-b4ffb25f64f1">


### Optional Step - Configure Local DNS Names Resolution

Sometimes it is tedious to remember and switch between IP addresses, especially if there are lots of servers to manage. It is best to configure local domain name resolution. The easiest way is use /etc/hosts file, although this approach is not very scalable, but it is very easy to configure and shows the concept well.

### Configure the IP address to domain name mapping for our Load Balancer.

Step 1: Open the hosts file

```
sudo vi /etc/hosts
```

Step 2: Add two records into file with Local IP address and name for the Web Servers and their private ip-address respectively

Result:

<img width="849" alt="Screenshot 2024-06-20 at 22 27 48" src="https://github.com/sheezylion/load-balancer-solution-with-apache/assets/142250556/c7f5eb78-68da-4053-9391-fdfb7eb22319">

Step 3: Update the LB config file with those arbitrary names instead of IP addresses

```
sudo vi /etc/apache2/sites-available/000-default.conf
```

```
BalancerMember http://Web1:80 loadfactor=5 timeout=1
BalancerMember http://Web2:80 loadfactor=5 timeout=1
```

Result:

<img width="847" alt="Screenshot 2024-06-20 at 22 29 36" src="https://github.com/sheezylion/load-balancer-solution-with-apache/assets/142250556/e10691cd-97bf-483c-afff-b4de60d2bfce">

Step 4: Curl the Web Servers from LB locally

```
curl http://Web1
```

Result:

<img width="849" alt="Screenshot 2024-06-20 at 22 31 10" src="https://github.com/sheezylion/load-balancer-solution-with-apache/assets/142250556/4c43a9f8-c602-40a0-8be3-164307d97d28">

```
curl http://Web2
```

Result:

<img width="798" alt="Screenshot 2024-06-20 at 22 32 16" src="https://github.com/sheezylion/load-balancer-solution-with-apache/assets/142250556/3cb0628a-65ef-4dbc-aeef-4039223ce97c">

Remember, This is only internal configuration and also local to the LB server, these names will neither be 'resolvable' from other servers internally nor from the Internet.

### Conclusion
The load-balancer solution with apache offers robust features for load balancing, including support for health checks, and various load balancing algorithms. Properly configuring these options ensures high availability, scalability, and reliability for web applications.

