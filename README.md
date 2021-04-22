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
- Zookeeper: https://lucidworks.com/post/how-to-use-apache-zookeeper-to-build-distributed-apps-and-why/
- Bloom Filters: https://www.youtube.com/watch?v=bgzUdBVr5tE https://www.youtube.com/watch?v=heEDL9usFgs
- Cassandra: https://blog.discord.com/how-discord-stores-billions-of-messages-7fa6ec7ee4c7
- UUID: https://blog.twitter.com/engineering/en_us/a/2010/announcing-snowflake.html

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

## Apache Zookeeper
 
A tried and tested Apache project for co-oridnation between distributed applications. Zookeeper is used by other Apache projects like Hadoop, Hbase, HDFS, Solr. It is also used by Kafka & Pulsar as well. 

***In the CAP philosophy, Zookeeper comes in the CP category.*** That means Zookeeper is oriented towards greater consistency. This is extremely crucial for banking related applications. ***Cassandra would come in the AP categoy which gives greater availability though.***

Zookeeper helps in

 - Distributed configuration management
 - Select election / consensus building
 - Coordination and locks
 - Able to store Key value stores used for configurations.
 - Used by Hadoop, Kafka, Pulsar, Solr

## CAP Theorem
All distributed systems sufffer from partitions - network failures for example. So the extent to which Availability of a distributed system could be studied is using CAP Theorem. 

- **C**onsistency
- **A**vailability
- **P**artition Tolerance

In the event of failures, a Distributed system is broken into Partitions. In such case, the design of the distributed system can either favor Consistency or Availability but not both.

**Working Example** - Assume that there is an ATM machine that is not connected to the central server. So when a user walks up and requests for withdrawal,
 - ***Consistent design*** would let the request by the user to be rejected. Ex: Financial transactions.
 - ***Availability design*** would let the request to be processed. Ex: Book reviews systems.

![image](https://user-images.githubusercontent.com/42272776/111916555-86bf8c00-8aa1-11eb-91b5-e82fef1f672d.png)

## Bloom Filters

- They are a probabilistic datastructure that tells if an entry is in a collection.
- Correct at giving TRUE POSITIVES all the time - case where the datastruture says that an entry is present in it when it really is present.
- Cannot give you a a FALSE NEGATIVE - case where the datastructure says that an entry is present when infact it is not.
- Can give you a FALSE POSITIVE - case where the datastructure says that an entry is present when infact it is not.


Consistent Hashing
Database Indexes
Distributed Hashing
Dynamic Sharding
Storing/Sorting large data
NoSQL databases
Distributed File System Design
OAuth & JWT
Multi Region Clusters
Streaming Data
Message Queues
Database design
1,2,3 Normal Forms
Database partition
When to use which NOSQL Database?.
https://www.youtube.com/watch?v=v5e_PasMdXc
https://medium.com/swlh/log-structured-merge-trees-9c8e2bea89e8


# AlgoExpert
- Understand what all things you want to build
- Pick the most important components and prepare estimates
  - Memory, Bandwidth etc

## Design Netflix
- Questions
  - Which sub-functions should be focussed on?.
  - Do we want to focus on authentication / payments / profile?.
  - How many users are we expecting to have?.
    - 200M
  - Is our goal high availability and low latency?.
  - Are we targetting global audience?.
  - Can the Recommendation system be a background process. 
  
## Approach
- Data
  - Videos
  - User metadata
  - Static content
  - Logs
- Estimates
  - Videos
    - Assume 10000 movies
    - Different video resolutions - Standard and HD
    - Average video length 1 hour - Standard 10GB and HD 20GB
      - 10000 movies * 30GB = 300k GB = 300TB
    - Use S3 storage for this.
  - User Metadata
    -  What all the user has watched, where he left off for a video etc.
    -  Assuming 200M users and that each user watches 2 movies/shows per week, in one year, that would be 100 shows.
    -  If the average time a user is on Netflix is 10 years, it would be 1000 * lets say, 100bytes per video = 100KB.
    -  For all users 200M * 100KB = 20TB of metadata information.
    -  Can be stored in a PostGRE database which is sharded based on User ID.
    -  RDBMS because an admin might want to run queries to compare watch histories of two different people and draw some reports.
  - Static content
    - Includes subtitles, titles, cast, ratings etc.
    - RDBMS or MongoDB can be used and part of this could be cached at our API servers.
  - Bandwidth
    - Total 200M users. 5% concurrent users - 10M at peak hours in HD.
    - 10M * 20GB/hour = 10M * 20GB/4k seconds = 50TBps
    - Need to be spread out to multiple locations as the frequency ask is too high.
  - Video Content Delivery
    - As seen above, if a new movie comes up, it is difficult to stream it out of a single datacenter itself.
    - We can use CDN. CDN should have a cache of videos.
    - CDNs can have thousands of Points of Presence.
    - Since CDN cannot have all the content, we need to have a service that will be populating the cache with updates.
  - API Servers
    - A round robin scheme can be employed to distribute end user network requests across our API servers.
    - These are hit when a user tries to access the system.
    - So a cache layer can be put to include user metadata.
  - Logs - Recommendation Engine
    - Store logs in HDFS
    - Async Map Reduce jobs that process data from HDFS
    - |UserID:"uid1"|event:"Type"|VideoID:"videoID1"|
    - Map: uid1: { (video1,Pause), (video2, Play) }
    - Reduce: Data pipeline + Machine learning model
      - uid1: { v1,v2,v3 }
- System Design
![image](https://user-images.githubusercontent.com/42272776/115126791-71317980-9fef-11eb-867b-91a67843cbd6.png)

- Video Process and Serving
  - To convert videos into different formats, Netflix uses a cron job that operates on chunks of Movies/Shows and converts them into the appropriate codec.
  - When you are watching a video, sometimes there will be lags. This is mainly because there is an API call fired to get the next part of the video.
  - This is because the entire video is broken into multiple chunks and during an API call, the corresponding chunk will be returned.
  - To reduce this sort of delays, instead of breaking video into multiple chunks, the video is broken into multiple scenes of different lengths.
  - Another optimization that is done is that Netflix places a cache called Open Connect at the ISPs. This is because all the content is in central US region
    and fetching them to India is not a trivial task. 90% of the requests are served from these devices.

## LinkedIn Search Architecture
- Based on Galene, which is build on top of Lucene.
- Needs faceting, relevance, updates without having to create a new document.

- Datasets in LinkedIn are either
  - ETL'ed to Hadoop
    - This has the complete dataset
  - Real time change notification team
    - Delta dataset
    - Person X is following Person Y
    - Change to only part of the document, not update to entire document.
 
- Galene Anatomy 
  - Base Index
    - Built by Hadoop periodically
    - Immutable, on disk
  - Live Index
    - Inverted index
    - In memory data structure
    - Contains incremental updates to the documents

## Twitter Search
- Source: https://www.infoq.com/presentations/Twitter-Timeline-Scalability/
- What does Twitter stand for: https://www.youtube.com/watch?v=VttXHNveuwI
- Queries per second ~ 300K
- Writing ~ 6000
- Active users - 150M
- 400M tweets per day

![image](https://user-images.githubusercontent.com/42272776/115769469-36518c00-a3c9-11eb-9f05-b8cd46a958e4.png)
- When a Tweet is posted, it goes to the WRITE API, which starts a process called FAN OUT.
- This process places the tweets to massive REDIS clusters.
- Every tweet is stored in 3 machines, 3 times.
- FAN OUT queries Social Graph Service ( who follows who ) and iterate through all the timelines that are stored in REDIS
- So if you have 20000 followers, once you post your tweet, the FAN OUT process would query the Social Graph Service to understand where they are stored and updated all the
entries with the new tweet id. Literally 20000 inserts are happening with just one tweet.
![image](https://user-images.githubusercontent.com/42272776/115769769-87618000-a3c9-11eb-827f-2496e7e06943.png)
- Cap of 800 tweets as you scroll through the timeline because it is meaningless to show up an old tweet in your timelines.
- Every user's timeline is stored in one of these servers.
- If the user is not logged in for the last 30 days, we skip timeline update.
- Index of all the home timeline are stored in REDIS. User timeline are stored in DISK - MySQL.
- If there is a miss, then the big query on the database is done and the data is loaded onto REDIS as soon as possible.
- So Timeline Service, just does a cache look up ( possible 3 servers ) and returns the one that got returned the fastest.
![image](https://user-images.githubusercontent.com/42272776/115770548-7bc28900-a3ca-11eb-9131-fb181992a2d9.png)

- Search
  - Every single EarlyBird shard is asked about a match.
  - EarlyBird is a Lucene Index.
  ![image](https://user-images.githubusercontent.com/42272776/115771457-91847e00-a3cb-11eb-917d-d23b62e3d11a.png)

![image](https://user-images.githubusercontent.com/42272776/115771626-cf81a200-a3cb-11eb-95fe-a2cd84cdb35f.png)
![image](https://user-images.githubusercontent.com/42272776/115778165-dd3b2580-a3d3-11eb-884b-0792280f232c.png)

- Tweet Post vs Tweet Search
![image](https://user-images.githubusercontent.com/42272776/115772110-5df62380-a3cc-11eb-9890-443f67341c86.png)

- This architecture can introuced Race conditions. Ex: Tweet from Lady Gaga may appear to one of her followers X before Y. And if Y also follows X and X replies or retweets Lady Gaga's tweet, then Y could potentially see X's reply to Lady Gaga's tweet first followed by the original tweet.
- These are resolved by sorting the tweets based on tweet ids.
- To handle the celebrities, there are different ways that Twitter is looking at like merging just before content is distributed.
- If two people are talking, their mutual friends alone would receive notifications.

![image](https://user-images.githubusercontent.com/42272776/115773822-60597d00-a3ce-11eb-8d38-b69cf087587e.png)


![image](https://user-images.githubusercontent.com/42272776/115759468-e4efcf80-a3bd-11eb-928c-9948b182e24a.png)

![image](https://user-images.githubusercontent.com/42272776/115759516-f46f1880-a3bd-11eb-9556-98527cd75662.png)


 


## My questions
- When to use a reverse proxy?.
- How do you ensure that the right videos are cached at CDN?.
- IXP vs Public points of CDN partnered with AT&T / Verizon.
- Authentication fror data requests to CDN?.
- Write through cache policy?.
- How do you design a system like Gitlab?.
- How should we handle cases like this:
  - I started following a new user X. My timeline should be updated with updates also from X starting now. Apart from DB storage, how are these handled - Kafka, Redis whats the best practise.
  - How are timelines generated?.
  - How are searches handled?.
- https://www.educba.com/redis-vs-kafka/
- https://logz.io/blog/kafka-vs-redis/





## System Design Tips
- Systems like Facebook, Twitter, LinkedIn, Reddit, Instagram have the following main features
  - CRUD
  - Search
  - Timeline view
  - Ability to follow others
  - Suggestions on who to follow
  - High availability + eventual consistency
- Most systems are READ heavy 
  - See if you can prove if the system is READ heavy.
- LRU seems to be a decent Cache replacement strategy at first sight
  - But for scenario where there are hot users, replacing their tweets/post may not be wise as they are READ by lot of other people.
- Load balancers are typically placed at multiple locations
  - Between users and API servers.
  - Between API servers and databases.
  - Probably between API servers and caches as well.
- Talk about replication and fault tolerance.








----------------------------------------------------------------------------------------------------------------------------------------------------------------
