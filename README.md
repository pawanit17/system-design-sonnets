# Foreword
system-design-sonnets is a collection of system design concepts and principles that help in both progression towards an architect role in your organization as well as for cracking product based companies which stress not only on the coding ability, but also on the architectural part.

## Some good resources on which this content is researched upon are:
- Overall Topics: https://github.com/donnemartin/system-design-primer
- Multiple Load Balancers: https://www.flexera.com/blog/cloud/dns-load-balancing-and-using-multiple-load-balancers-in-the-cloud/
- Load Balancing Techniques: https://kemptechnologies.com/load-balancer/load-balancing-algorithms-techniques/
- CAP Theorem: https://www.youtube.com/watch?v=k-Yaq8AHlFA
- Diagrams authored via https://app.diagrams.net/
- Content Delivery Network: https://www.youtube.com/watch?v=Bsq5cKkS33I
- Forward Proxy vs Reverse Proxy vs Load Balancers: https://www.youtube.com/watch?v=MiqrArNSxSM
- Heartbeat Systems: https://medium.com/@adhorn/patterns-for-resilient-architecture-part-3-16e8601c488e
- API Gateways: https://www.nginx.com/blog/building-microservices-using-an-api-gateway/


# Centralized vs Distributed Systems

- Centralized systems have everything on the same machine. They offer single point of failures.
- Distributed systems have blocks which are running on different nodes ( machines ). This provides seperation and helps in extending the architecture per need in future by adding redundance/replication.

# System Design - Why is it needed, a short intro

As you build an application and the application becomes a success, first thing that you would notice is that the number of users visiting or using your application increasing. This increase could then expose faulty areas in your architecture that are previously dormant. This is, in most cases, the web servers, databases etc. Furthermore, if a database that you are so reliant on goes down, then your application is rendered useless.

[comment]: <> Add a picture for easier recollection

That is why a need for reducing the bottlenecks and having standby extras ( databases, servers etc ) is handy.

## Horizontal vs Vertical Scaling

If the numbers of requests that a server is serving are increasing, we have two options.

### Vertical scaling

- Simply add more hardware to the system. Increase its RAM, buy better harddrives or move to SSD etc. Add more L1, L2 caches.
  - L1 cache is directly on the microprocessor chip itself and so is the fastest cache.
  - L2 cache is typically on a seperate chip on the motherboard but very close to the chip. L2 is only consulter is L1 lookup is a **miss**.
  - ***L1 is about 14x faster than L2 & about 100x faster than a RAM query.***
- Sooner or later you would hit a limit for such increase on a server though.
- ***Not to mention, having a single server serve the user requests would result in a single point of failure, the server itself. If it goes down, the application goes down.***

### Horizontal scaling

- In this mode, we add multiple servers of the similar configuration and put a load balancer ahead of them.
- This means that the load balancer would now be hit for any requests to these servers.
- ***If the load on the application is bound to increase, simply increase the number of servers behind the load balancer.***
- ***Load balancer would need to have a public IP and all the other servers need not have a public IP => need not be accessible to the internet.***
- ***For the application domain name, we would now have the load balancer IP mapped in the DNS record. Furthermore, we can also have more than one load balancers for an application. In this case all the different load balancers would be listed in DNS in multiple records.***
- ***Take google for example: 
[comment]: <> Add a picture for easier recollection

## Load Balancers

When a request reaches a load balancer from the internet, there are some variety of ways in which this request could be forwarded to one among the different application servers available.

![Load Balancer & Distribution Algorithms](/images/load-balancer-and-balancing-algorithms.png)

#### Algorithm: Random distribution
- Requests are distributed to random webserver.
  - ***Same server could be fed with most of the load and yields improper usage of other resources.***
  - ***Applications that need to connect to the same server for all requests would be lost.***
  - Shopping card application where the checked out items are present in the session.

#### Algorithm: Least Busy Server
- Requests are distributed to the server that has the least amount of load/connections.
  - ***Judicious usage of servers.***
  - ***Applications that need to connect to the same server for all requests would be lost.***
  - Shopping card application where the checked out items are present in the session.

#### Algorithm: Round Robin
- Requests are distributed to each of the servers in turn, one after the other.
  - Judicious distribution of tasks.
  - Same heavily loaded server would still get the requests.
  - ***Applications that need to connect to the same server for all requests would be lost.***
  - Shopping card application where the checked out items are present in the session.

#### Algorithm: Sticky sessions / Source IP Hash
- Requests are analyzed by load balancer and is directed to the webserver, which is the HASH of say, userid / source ip.
  - All requests from the same client machine of the given user go to the same webserver.
  - This HASH could even be stored as a cookie on the client machine by the browser.
  - ***This approach would preserve the session caches that we have discussed about.***
  - ***Drawback of this problem is the unequal distribution of requests against the servers.***

#### Algorithm: Round Robin + Session Cache
- Requests go to the load balancer which then checks if the user is new to the site.
  - If so, a server that falls next in the round robin would be assigned to him.
  - An entry to the central cache holding the session information would be made.
- If the user is not new
  - A look up is made to see which server he was previously allocated to.
  - Request is redirected to that application server.
- ***This approach preserves the user historic session information and is also making judicious usage of servers.***
- ***Important point to note here is the storage for user sessions - it should be a persistant simple storage like REDIS.***

## Content Delivery Network

When an application gets deployed at a certain geography, say in Netherlands, people from the Netherlands would notice that the site is faster to access. But people from other parts of the world, say US, would notice that it is slower. US to Netherlands has a delay of about 140ms. Furthermore customers residing in australia could experience the worst traffice where the ping delay could be around 200ms. Greater this delay worser would be the performance of the application. The delay that we just referred to here is the ***ping delay***, the amount of time it takes for a network packet to move from US/AU to Netherlands and travel back. 

For most applications, the part that always remain the same can be cached - this could be the images, the HTML pages etc. If we are able to cache them closer to say, US/AU in this case, the delay would radically come down and gives the application a very good performance. Note that the dynamic parts that have to go to Netherlands central server for execution would still continue to exist, but the application would still offer a far better performance than without the caches.

These caches are called as Content Delivery Networks - CDNs.

### What can be cached
  - Text, graphics, scripts
  - media files, software, documents
  - these days dynamic content too is getting cached

### CDN Types

Basically there can be two types of CDNs.

#### Pull CDN
The first request to load the web page would cache the content at the CDN. Subsequent readers would then read it from the CDN instead of the central server. Note that this caching is only done in the region that user is requesting from.

- Easier to implement.
- If the content on the central site changes, it may take some time for it to reflect on the CDN.

#### Push CDN
Proactively push the application cacheable content onto the CDN. 

- Always up to date.
- Effort involved to push content to CDN.
  
### Advantages
  - Reduction in traffic that hits the main central server
  - The infra detail at the central server could be trimmed down.
  - Increase in uptime at the central server, increase in security.

### Disadvantages
  - Procurement of the CDN servers
  - Maintaining the content on the CDN servers, especially if the cached content changed on central server.

## Proxy Server vs Reverse Proxy vs Load Balancer

### Proxy Server
Typically in organizations, all the user laptops are connected in VLAN ( User lan ) and the requests from this LAN to the internet are achieved via a Proxy Server. That means all the requests are received to the proxy server, which then consults the internet.

TODO: Add an image

#### Benefits:
- Safety of the clients
- The destination servers in the internet also do not know about the client ip addresses.

### Reverse Proxy
A reverse proxy masks one or more servers by serving as the CNAME for the application hostname. This means that the internet does not know the ip address of the backed servers. A reverse proxy can still be used even if only one web server is in play. Also, the presence of a reverse proxy introduces a single point of failure, so redundancy has to be introduced ( multiple DNS records ) to ensure high availability. 

Example: Nginx

***Reverse proxy can do a lot of things, one such thing is load balancing. A load balancer on the otherhand, is strictly for balancing the load.***

#### Benefits
 - Since only the reverse proxy is exposed, the backend can be changed without any effect to the end users.
 - Will be able to do the decrypting of HTTPS traffic which is costly to do at the web servers.
 - Compression and caching features are also provided by reverse proxies.

## API Gateway
API Gateway, is a specialization of reverse proxy. API gateway is mainly used to facilitate the ease of access of the APIs to the different clients.

![A simple product information page at Amazon](/images/amazon-product-info.png)
As we can see, a typical product information page at Amazon contains lots of information - product description, images, product reviews, recommendations etc. In a monolithic application, all the functionalities described above would be catered by a single large service. Problems arise when new aspects are to be added, existing be removed or if there is a need to either merge or split the services.

In a Microservices world, this would be taken up differently. Each of the features would be taken up by a correspnding Microservice. Here, we have two options.

### Exposing all the services to all the clients
In this approach, all the different clients - tablets, pcs, mobiles etc, would have access to each of the individual services and they can call them individually. Disadvantage of this is that the client would then have to call multiple REST API calls to get the job done.

### Using an API Gateway
In this approach, we add an API Gateway to the system and with this all the different clients now connecting to it, which inturn connect to the Microservices ( or to other reverse proxy ). Advantage of this scheme is that the clients do not need to know how the internal architecture of the system is designed. This allows removing of adding new services easy. This scheme also helps us in providing an API to a client that is specifically tailored for it ( tablet, Pc, mobile etc ). 

Disadvantage would be that we are introducing a single point of failure and so redundancies have to be built into the system.

### Netflix example
Netflix streaming service orginally had a single sized API for all types of clients. They then noticed that itwas not doing well because of the diverse range of devices that Netflix caters.

## Heartbeat Servers

Load balancers always do a Heartbeat check on the web servers that it is managing. Typically what happens is if a server does not respond to the hearbeat message, then the load balancer would no longer forward requests to that server until the service is back again. This helps in preventing loss of messages from end users.

- ***Shallow Heartbeat services*** just employ a **ping** or a **telnet** command. This is fast and easier to maintain but the problem is the machine may be reachable but the application may be stuck.
- ***Deep Heartbeat services*** do application level checks to see if the application is responding.

Most implementations also route these messages in the form of logs to Splunk or to ElasticSearch/LogStash so that the visualization could happen in Kibana to understand in more depth as to what cause the interruption of service.

Database Indexes
Consistent Hashing
Distributed Hashing
Dynamic Sharding
Storing/Sorting large data
Zookeeper - elections
NoSQL databases
CAP Theorem
Distributed File System Design
OAuth & JWT
Multi Region Clusters
Streaming Data
Message Queues

