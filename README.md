# Foreword
system-design-sonnets is a collection of system design concepts and principles that help in both progression towards an architect role in your organization as well as for cracking product based companies which stress not only on the coding ability, but also on the architectural part.

Some good resources on which this content is researched upon are:
https://github.com/donnemartin/system-design-primer#system-design-topics-start-here
https://www.flexera.com/blog/cloud/dns-load-balancing-and-using-multiple-load-balancers-in-the-cloud/
https://kemptechnologies.com/load-balancer/load-balancing-algorithms-techniques/

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

### Load Balancers

When a request reaches a load balancer from the internet, there are some variety of ways in which this request could be forwarded to one among the different application servers available.

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


Database Indexes
Consistent Hashing
Distributed Hashing
Scalable Backends
Dynamic Sharding
Storing/Sorting large data
Heartbear Servers
Zookeeper - elections
NoSQL databases
CAP Theorem
Distributed File System Design
OAuth
JWT
Multi Region Clusters
Streaming Data

## Message Queues

