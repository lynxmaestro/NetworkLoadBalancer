# What is a Network Load Balancer (NLB)?
Network Load Balancer (NLB) is a service provided by cloud computing platforms, such as Amazon Web Services (AWS), designed to efficiently distribute incoming network traffic across multiple targets, such as Amazon EC2 instances, in a way that enhances the availability and fault tolerance of applications. <br/>

A Network Load Balancer is designed to handle millions of requests per second while maintaining ultra-low latency. It operates at the Layer 4 (Transport Layer) of the OSI model, routing traffic based on IP protocol data (TCP, TLS, UDP).

NLB is best suited for use cases where performance and scalability are top priorities—like gaming apps, real-time communication, or financial systems.

Unlike Application Load Balancer no NATing with LoadBalancer IP is performed on NLB, the requests hitting on the NLB listener ports will be routed directly to the target groups assossiated. which makes it much more faster when compared with ALB, Also NLB is the costlier when compared with ALB.

# Key Features of AWS Network Load Balancer

Here are the core features that make NLB a go-to choice for high-performance workloads:
## 1. High Throughput & Low Latency

NLB can handle millions of requests per second with consistent low latency. It’s built to scale horizontally as demand grows.
## 2. Static IP Support

You can assign Elastic IPs to your NLB, allowing clients to whitelist a fixed IP range—especially useful for regulatory or firewall purposes.
## 3. Zonal Isolation

Each Availability Zone (AZ) is isolated, so failures in one AZ don’t affect others. NLB can distribute traffic across multiple AZs for fault tolerance.

## 4. Preserve Source IP

Unlike Application Load Balancer (ALB), NLB preserves the source IP address of the client. This is critical for apps that require IP-based security or analytics.

If we have a naked domain name say 'example.com' whose DNS is hosted on any 3rd party registar like GoDaddy, Hostinger etc. Also assume our domain is hosted on a load balancer endpoint. <br/>
Typically primary domain's cannot be pointed to a loadbalancer dns name, but in case of NLB we could deligate Elastic IP's purchased from amazon so that these IP's can be pointed as A record for the domain. <br/>
Amazon Route53 have a feature to alias the domain to the corresponding load balancer endpoint, in a way they are also pointing the domain to the public ip address of load balancer, but as the load balancer is managed by aws there could be chances where the public ip can be changed in case of any issues on the load balancer. <br/>

## Let’s Go for Demonstration on NLB

Setting up an Network Load Balancer with AWS EC2.

### Step 1: Launching Backend EC2 Instances
To ensure high availability, we launched two t3.micro instances across different Availability Zones: ap-south-1a and ap-south-1b.
Instance Details:
```
    webapp-ap-south-1a: Running in ap-south-1a.
    webapp-ap-south-1b: Running in ap-south-1b.
```

We used the following UserData script to install a web server and PHP, creating a page that identifies the specific instance:

```
#!/bin/bash
yum update -y
yum install httpd php -y
cat <<EOF > /var/www/html/index.php
<?php
\$output = shell_exec( 'echo \$HOSTNAME' );
echo "<h1><center><pre>\$output</pre></center></h1>";
echo "<h1><center>webapp-ap-south-1b</center></h1>"
?>
EOF
systemctl restart httpd.service && systemctl enable httpd.service
```

<img width="1903" height="408" alt="Pasted image" src="https://github.com/user-attachments/assets/ad0c6772-28af-4da2-af44-1b48f9b773f6" />

--------------------------------------------------------------------------------

### Step 2: Allocating Static Elastic IPs
For a predictable entry point, we allocated two Elastic IP addresses from Amazon’s pool. We tagged these as elp1 and elp2 to easily identify them during the load balancer configuration phase,.

<img width="1920" height="1080" alt="Pasted image (4)" src="https://github.com/user-attachments/assets/a18e75ee-df2c-4800-8314-aa7a66a4b418" />

<img width="1886" height="821" alt="Pasted image (5)" src="https://github.com/user-attachments/assets/f0bccb33-0b67-4dff-8a7b-646e7fcf3c30" />


--------------------------------------------------------------------------------
### Step 3: Creating the Target Group
We created a Target Group specifically for the NLB. Because NLBs operate at Layer 4, we configured the group to use the TCP protocol for the backend instances to ensure efficient traffic handling.

<img width="1919" height="935" alt="image" src="https://github.com/user-attachments/assets/47e709c8-51a0-4a69-8e9d-9ea09457f7d8" />



--------------------------------------------------------------------------------
### Step 4: Provisioning the Network Load Balancer
The NLB was created within the VPC vpc-0dc199b2204a7cc14. During the Network Mapping phase, we manually assigned our Elastic IPs to the corresponding subnets:

    In ap-south-1a, we mapped the subnet subnet-0e0c3baca24d97717 to the Elastic IP 13.205.55.116.

<img width="1904" height="899" alt="Pasted image (8)" src="https://github.com/user-attachments/assets/de75547a-3568-446e-af3e-18752cabcb46" />


--------------------------------------------------------------------------------
### Step 5: Route 53 and Functional Verification
After mapping our domain to the NLB via Route 53, we verified that the load balancer was correctly distributing traffic between the two AZs.

    Instance 1a Response: Displays the internal hostname ip-172-31-44-215.ap-south-1.compute.internal.
    Instance 1b Response: Displays the internal hostname ip-172-31-6-76.ap-south-1.compute.internal.

<img width="1904" height="899" alt="Pasted image (9)" src="https://github.com/user-attachments/assets/afcb84ac-fb07-4769-bfa8-56964c74e15a" />

--------------------------------------------------------------------------------
### The NLB Advantage: Source IP Preservation
A critical feature of the Network Load Balancer is that it does not perform Network Address Translation (NAT) for the source IP. By examining the Apache access logs on the backend instance, we can confirm that the user's actual public IP address is passed directly to the server.
Backend Log Analysis:

```
[root@ip-172-31-44-215 ~]# cat /var/log/httpd/access_log | grep -v "ELB-HealthChecker/2.0"
117.251.48.189 - - [02/Mar/2026:06:38:08 +0000] "GET / HTTP/1.1" 200 126 "-" "Mozilla/5.0..."
117.251.48.189 - - [02/Mar/2026:06:43:49 +0000] "GET / HTTP/1.1" 200 126 "-" "Mozilla/5.0..."
```

As seen in the logs, the IP 117.251.48.189 (the user's public IP) is recorded directly. This confirms that no NATing is happening at the load balancer level, allowing your backend applications to see the real client identity without needing complex headers like X-Forwarded-For.

<img width="1904" height="968" alt="Pasted image (10)" src="https://github.com/user-attachments/assets/7adf6f54-78fe-4081-bec0-1932c8193c6b" />
