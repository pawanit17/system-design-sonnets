system-design-sonnets is a collection of system design concepts and principles that help in both progression towards an architect role in your organization as well as for cracking product based companies which stress not only on the coding ability, but also on the architectural part.

Some good resources on which this content is researched upon are:
https://github.com/donnemartin/system-design-primer#system-design-topics-start-here


# System Design - Why is it needed, a short intro

As you build an application and the application becomes a success, first thing that you would notice is that the number of users visiting or using your application increase. This increase could then expose faulty areas in your architecture that are previously dormant. This is, in most cases, the web servers, databases etc. Furthermore, if a database that you are so reliant on goes down, then your application is rendered useless.

[comment]: <> Add a picture for easier recollection

That is why a need for reducing the bottlenecks and having standby extras ( databases, servers etc ) is handy.

## Horizontal vs Vertical Scaling

If the numbers of requests that a server is serving are increasing, we have two options.

### Vertical scaling

- Simply add more hardware to the system. Increase its RAM, buy better harddrives or move to SSD etc.
- Sooner or later you would hit a limit for such increase on a server though.
- ***Not to mention, having a single server serve the user requests would result in a single point of failure, the server itself. If it goes down, the application goes down.***

### Horizontal scaling

- In this mode, we add multiple servers of the similar configuration and put a load balancer ahead of them.
- This means that the load balancer would now be hit for any requests to these servers.
- ***If the load on the application is bound to increase, simply increase the number of servers behind the load balancer.***
- ***Load balancer would need to have a public IP and all the other servers need not have a public IP => need not be accessible to the internet.***

[comment]: <> Add a picture for easier recollection

## Load Balancers

## Message Queues

