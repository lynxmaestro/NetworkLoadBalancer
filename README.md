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

Setting up an Network Load Balancer with AWS EC2
