# high-level-system-design-exercise-notes

## What should you consider for EVERY DIAGRA<

Here’s a comprehensive list of requirements every **architecture diagram** should include, incorporating industry-standard considerations for designing and communicating distributed systems effectively:

---

### **1. Functional Requirements**
   - **Use Cases**: Clearly define the primary use cases the system is designed to handle (e.g., e-commerce, messaging, data processing).
   - **User Interaction Flow**: Represent how users or services interact with the system at a high level (e.g., APIs, UI, integrations).

---

### **2. Non-Functional Requirements**
   - **Scalability**:
     - Expected system growth (e.g., projected DAU, monthly growth rate).
     - Load expectations: Peak requests per second (RPS), read/write ratio.
   - **Latency**:
     - Define expected response times for user-facing operations (e.g., ≤100ms for search results).
     - Set acceptable thresholds for backend processes (e.g., ≤500ms for batch processing or analytics).
     - HOW??? PUT MORE HOWS FOR THESES? CDN??? CACHE????
   - **Availability**:
     - Target uptime percentage (e.g., 99.9%, 99.99%, or higher for mission-critical systems).
     - Redundancy and failover strategies for system components.
   - **Partition Safety**:
     - Mitigation strategies for network partitions, natural disasters, or regional outages.
     - Multi-region failover and disaster recovery plans.
   - **Security**:
     - Authentication and authorization mechanisms.
     - Data encryption (at rest and in transit) and compliance with standards (e.g., GDPR, HIPAA).

---

### **3. CAP and PACELC**
   - **CAP Theorem**:
     - Clearly state trade-offs between **Consistency**, **Availability**, and **Partition Tolerance**.
     - Highlight the system's behavior during network partitions (e.g., eventual consistency vs. strong consistency).
   - **PACELC**:
     - Consider trade-offs when there is no partition: **Latency vs. Consistency** in normal operating conditions.

---

### **4. Data Management**
**TRY TO DEFINE the tables needed and the data inside those tables**
   - **Data Storage**:
     - Identify types of storage used (e.g., **SQL** for structured data, **NoSQL** for high scalability, **Object Storage** like S3 for unstructured data).
     - Define data partitioning and replication strategies.
   - **Sharding**:
      - how are we sharding, what is the key, are we sharding at all, sharding vs partition?
   - **Data Consistency**:
     - Outline consistency models (e.g., strong, eventual, causal).
   - **Data Retention and Archiving**:
     - Plan for long-term data retention, archiving, and retrieval policies.
   - **Backup and Recovery**:
     - Define backup frequency and recovery time objectives (RTO/RPO).

### **8. Diagram Specifics**
   - **Component Overview**:
     - Include core system components (e.g., APIs, databases, services, storage layers).
   - **Data Flow**:
     - Illustrate how data flows through the system, including inputs and outputs.
   - **Key Interactions**:
     - Highlight integrations with external systems or dependencies (e.g., payment gateways, third-party APIs).

By including these details, the architecture diagram provides a holistic view of the system, making it easier to evaluate, maintain, and improve.

## Random facts
* for interview purposes lets assume **the average image is 10MB**, which is **10 MILLION BYTES**
* 32GB of RAM is a lot for a personal computer, but for systems like Redis and Memcached, 256GB is entirely manageable. High-end servers can support up to 500GB or even 1TB of RAM, making them ideal for in-memory caching and real-time data processing at scale
* Summary: Why NoSQL Is Faster: Avoids schema validation and normalization overhead. Uses distributed, horizontal scaling to reduce node load. Skips or limits ACID compliance for performance. Optimized for specific access patterns and workloads. In-memory and append-only storage engines accelerate reads and writes.

## Security
If you wanted to secure a backend so only your frontend can access you can do the following:
* **FOR AWS** Security groups can isolate by resource (stateful, only have allow rules) and NACLs (Network Access Control List) can limit by VPC/subnet-level. NACLs are stateless meaning that incoming traffic is not automatically allowed to exit (unlike Security Groups which are stateful)
* Quick refresh, CIDR blocks can be divided into subnets. Subnets are smaller ranges of IP addresses, like 10.0.0.0/24 would be 255 ip addresses. from 10.0.0.1 -> 10.0.0.0.255
* use a secret/API key stored in the cloud, this would be put in the "Authorization" header of the api call
* use a JWT token

## Design a rate-limiter
* fail open: when it fails, the system still allows traffic
* fail closed: when down, all traffic is blocked

* rate limiter acts like a reverse proxy, forwarding some requests, and blocking people who are being limited
* we want to use a in memory key-value store like redis or memcached to track upploads in the past 24 hours, also to write additions when users upload new videos

* if we weren't basing reqeusts on logged in individuals, we'd have to rate limit based on IP, there are issues with this however ISP change interal ip addresses so rate limiting wont affect a whole school for exmpale

* "Fixed window" rate limit allows 100 requests per minute, and resets at the end, so you could potnetially send 200 requests in 2 seconds if you did it at the end and beginning of a threshold
* "Sliding Winow" will continually test the last 60 seconds on each request to see if we've gone over

## TinyURL 
<img width="670" alt="Screenshot 2025-01-20 at 3 07 42 PM" src="https://github.com/user-attachments/assets/8dad85b2-9ee5-4dd9-b736-899ba900a9a7" />

#### Good questions
* What is the ratio of reads to writes (maybe 100:1, or 1000:1)
* How many reads per day? (100,000 reads, so maybe 1000 writes as well)
* How many reads per second? ( 100,000 / 24 * 60 * 60 ) -> (100,000 / 25 * 3600) -> 4000/3600  -> about 2 reads per second
* How much storage do we need? (1000 writes per day means 30,000 writes per month. so we need to ask more questions)
* How much data is in each entry? well a shorturl we agreed is 8 chars, which is 8 bytes. The longurl could average 100 chars which is 100 bytes. A date is 4 bytes, so that's about 112 bytes per entry, we'll call it 100 for simplicity. So we need 3,000,000 bytes per month, which is 3MB so x12 is 36MB per year.
* How is hashing going to work? (if we use database sharding/partitioning, we have to use consitent hashing in case a shard is removed)
* **Sharding is a form of paritioning called Horizontal Paritioning**
  
* How are we doing **cache eviction** ( I like always using LRU (least recently used, as opposed to LFU) unless there's a reason not to)
* How are we doing **cache invalidation** (we can set a TTL)
* How are we doing **DynamoDB invalidation** (we can use DynamoDB's TTL feature on our "expiration" field)
* how many possible urls are there?
<img width="411" alt="Screenshot 2025-01-02 at 12 00 15 PM" src="https://github.com/user-attachments/assets/41b59db0-d52f-458c-a412-4deaf1fcc777" />

#### How many reads is enough to consider NoSQL
* generally greater than 10,000 read/writes per second is enough to start consider nosql
* however, its important to consider that caching can reduce reads by 80-90%
* so it's more like if WRITES are above 10,000 and if reads are above 100,000 (with caching)
  
## Twitter

#### Application Server v.s. Webserver

* webserver serves static content, applications server calcuclates and serves dynamic content
* we have reverse proxy webservers: Traefik and NGINX, that can serve static and then relay dynamic requests to things like **Flask**, **Django**, or **Node.js**
### Reverse Proxy Webserver Examples: 
* Traefik,
* NGINX,
* Apache.

* **Static Content:** Can directly serve static assets from a filesystem or cache for speed.
* **Dynamic Content:** Forward requests to application servers (e.g., Flask, Django, Node.js).
* **Additional Features:** Load balancing, SSL termination, caching, and security (e.g., request filtering).

#### CDN (Cloud Delivery Network)
when you hear CDN, think static files: text, images, videos
* Pull CDN: you only update edge nodes when you make a request and you 'cache miss'
* Push CDN: Origin server updates and is responsible for pushing that content to edge nodes

#### Caching
#### Read strategies
* Read through: On a cache miss, the cache **automatically** fetches the data from the origin server, stores it, and serves it to the application.
* Lazy-Loading (cache aside): On a cache miss, the APPLICATION ** conditionally based on the application's stipultation** fetches the data from the origin server, stores it, and serves it to the application.
![image](https://github.com/user-attachments/assets/18223096-3947-4fcd-bc2f-1bff0a4ad4f5)
#### Write strategies
* Write through: Data is written to both the cache and the origin server simultaneously.
* Write around: Data is written directly to the origin server, bypassing the cache. The cache only gets updated when a subsequent read request causes a cache miss.
* Write back/behind: Data is written to the cache first and later asynchronously written to the origin server.

#### Cache Evication
* LRU (Least Recently Used): best option imo, popular things will be sustained
* LFU (Least Frequently Used): Based on total visits, OLD viral things will be kept
* FIFO: queue, regardless of access patterns
#### Cache Invalidation
* TTL: expires based on time (this is more cache eviction rather than invalidation)

## Design Discord

<img width="663" alt="Screenshot 2025-01-20 at 3 07 27 PM" src="https://github.com/user-attachments/assets/ae49aa4e-c058-46b1-9d50-aa23c4c919a5" />

#### Fault tolerance
* Load Balancers can prevent issues when servers go down (AWS has NLB, ALB, and DNS based LB)
* Kubernetes and EKS can help prevent issues when containers go down
* Database replication (both local and in different availability zones) can help when database shards or entire databases go down
* We have failover methods when things crash (active-active means they both do work, active-passive means the backup is only used when the original crashes)
* AWS Auto-scaling solutions like **Auto Scaling Groups** for EC2, **ECS Autoscaling** for both services (host and loadbalance groupings of pods) and clusters (host services), **Fargate** is serverless and handles the scaling for you (can integrate with EKS). **AWS EKS** can use HPA (horizontal pod autoscaling). 

Imagine a global e-commerce platform:
1. **Redundancy**:
   - Multiple servers in different regions handle user requests.
2. **Load Balancing**:
   - Requests are distributed across these servers using AWS Elastic Load Balancer.
3. **Failover**:
   - If a region goes offline, Route 53 reroutes traffic to a backup region.
4. **Replication**:
   - Databases replicate across regions to ensure data consistency and disaster r
     
#### Latency

* 5M active users per day, they'll probably do around 10 messages a day
* 50M messages per day / 24 / 60 / 60 -> 50M / 25 / 3600 -> 2M / 4000 ->  500,000 / 1000 -> 500 wps
* max limit on a message is how much, 2000 characters, so 2000 bytes, but maybe we average at 50 bytes?
* so it would be 25kilobytes / second for writes 

* 20,000 people can be in a server max
* 

#### Websocket
* we want websocket for live updates, but http for pagination and pulling of old messages
* API Gateway has a websocket api
* When a user connects, store their connection ID (provided by API Gateway) in a DynamoDB table
* When a user sends a message, the Lambda function for sendMessage can:
1. Retrieve all connection IDs for the channel from DynamoDB.
2. Use the API Gateway Management API to push the message to all active connections:
* Use the $disconnect route to remove connection IDs from DynamoDB when a user disconnects.
* websockets can and will need load balancers cause it cannot support inifite connections

#### Data storage
<img width="495" alt="Screenshot 2025-01-05 at 6 01 30 PM" src="https://github.com/user-attachments/assets/11aa34ad-ec5d-4bbd-9702-d99122026367" />
<img width="625" alt="Screenshot 2025-01-05 at 6 21 40 PM" src="https://github.com/user-attachments/assets/9155c7a6-9252-4e0d-9f01-56e31ea06c25" />


###### DynamoDB
* **Sort keys** order data **within a parition**, so we can do things like search for items < value
* **Global Secondary Indexes (GSI)** allow you to query data **across all paritions**
* Adds storage and write costs because DynamoDB maintains a copy of the indexed attributes in the GSI.

* maxes on msg for bytes/chars will help with the math for writes and reads
  



## Youtube
<img width="607" alt="Screenshot 2025-01-20 at 3 07 09 PM" src="https://github.com/user-attachments/assets/febc9bc4-6efa-4706-a8fd-7e7cb9c5a2b4" />

1,000,000,000
5,000,000,000 watches a day
50,000,000 uploads a day
50,000,000 x 5GB = 250,000,000GB a day
250,000,000 / 100,000
25,000 GB / second for WRITES
**not quite 2,500,000 GB / second for READS cause we do streaming and chunking**

**there are approximately 100,000 seconds in a day (actually 86k)**

when you talk about average numbers, you need to aknowledge that there is peak traffic time, and especially less traffic at night if you're just in the US

15 min videos around at 60fps at 1080 is around 1GB of data

#### Scaling youtube writes
* first we have an api (API Gateway) and it's scaled with a loadbalancer (NGINX or AWS ALB) 
* next we have an upload service (AWS ECS ((ecs task)), EKS both scale) , which chunks the video (AWS S3 multipart upload)
* videos are chunked when sending, so network interruptions don't restart the upload from scratch
* we have an asyncronous message so it decouples the upload to object storage and then processing (can use an AWS SQS queue or use a pub/sub model with AWS SNS and have the transcoding service subscribe to it) this way it's faster (user gets feedback after s3 upload, not transacoding) its more fault tolerant and it can handle spike traffic better
* the queue will trigger various services, data replication, transcoding, and meta-data storage (services on ECS or EKS)
* split the encoding service into multiple workers, which process in batches, and the encoded data is also stored in S3
* the metadata goes into Google Big Table (aws equivalent??) because it's good for writes (a wide table database)
* we also use sharding cause the data is so vast
* sharding is HORIZONTAL PARTITIONING (hashign and decreasing rows) vs VERTICAL PARTITIONING (splitting columns into two tables)

#### Scaling youtube reads
<img width="535" alt="Screenshot 2025-01-12 at 12 03 41 PM" src="https://github.com/user-attachments/assets/933e2ffb-b371-4353-855a-12748fc529dd" />

* user makes a request to API Gateway which is scaled by NGINX or ALB
* this would handle rate limiting and authentication
* ALB can rate limit based on IP and API Gateway can rate limit based on user, API key, or resource (nonspecific, whole system),  built into both of them
* API Gateway queries for metadata from dynamoDB (this step can also be cached with elasticache/redis/memcached) 
* The request would likely default to a resolution, but can be changed by the user or automatically
* After a user is authenticated and the api recieves the metadata as well as a **cloudfront URL** if it's cached in the CDN or a pre-signed S3 URL for Origin direct access in S3 (which afterward, it would likely be added to the CDN)
* Adaptive Bitrate Streaming the videos are divided into 10 second chunks at various resolutions and bitrates, the video is then streamed based on network speed/bandwith at different qualities
* CDNs autoscale, also we can add read replicas to DynamoDB


## Design Google Drive
<img width="935" alt="Screenshot 2025-01-20 at 4 29 27 PM" src="https://github.com/user-attachments/assets/d84f58fd-de09-45d0-866d-a9af4a9fca22" />

* HDFS (Hadoop Distributed File System) is a distributed file system designed to store large datasets across multiple nodes in a fault-tolerant and scalable manner. HDFS is a distributed file system that allows for append operations and file modifications.
* **File systems use block level storage by default!!!** This means you can add new data to an existing file or update chunks of a file as part of a distributed computation process.
* HDFS users block storage. When editing, HDFS ensures only the affected blocks are updated or re-replicated.
* block level storage is cheaper, data must be reassembled,, we can do "deduplication" which doens't store duplicatte blocks, could be done on a global level or EVERY user.
* also allows for CHUNKING (save partial upload)
* ALso imporantantly you need a **metadata table** to keep track of which blocks belong to which file, for large files, you could even grab blocks in parallel.
* for instance, we can use the file metadata table to keep track of who is allowed to access the file, and we can use this in conjunction with the congnito token to add security
* Content Addressable Storage is a storage paradigm where data is identified and retrieved based on its content (e.g., hash) rather than a traditional file path or name. So we use hashes to keep track of blocks, and duplicated data is not stored
* we COULD implement a garbage colleciton sserverice that goes and tracks how manuy users own the file in the kv sotre, and perdodically goes through and delted duplicates
* to ensure safety when using a loadbalancer, you can setup backup loadbalancers wtih a heartbeat ping, can use **Zookeeper** to coordiante heartbeat and other distibuted system stuff

## Design Google Maps
* **Kafka** is a distributed event streaming platform designed for high-throughput, fault-tolerant, and scalable message/event handling.
* It acts as a pub-sub system (publish/subscribe) where producers publish messages (events) to topics, and consumers subscribe to topics to process messages.
![image](https://github.com/user-attachments/assets/0f97ac0f-8b36-467a-9823-97bda4f129bc)
* its all about **real time** and **need for scalability**
* Kafka Decouples Producers and Consumers, Without Kafka: Microservices must do things like push logs directly to S3 or Elasticsearch. Each service must be aware of the destination and handle failures/retries. With Kafka: Microservices (producers) send logs to Kafka topics. Kafka stores the logs durably and decouples producers from consumers. Consumers (like S3, Elasticsearch, or analytics systems) can independently process logs at their own pace.
<img width="719" alt="Screenshot 2025-01-22 at 7 02 49 PM" src="https://github.com/user-attachments/assets/e7bb2361-596a-44f4-bfe2-299bacf937b6" />

Industry Use Cases:
Uber: For collecting and processing GPS/location data in real-time.
Netflix: For tracking user activities and feeding them into analytics and recommendation systems.
LinkedIn: Initially developed Kafka for activity stream data.


High Event Volume:
* Example: Handling millions of events per second (e.g., user registrations or transaction logs).
* great for MANY MANY consumers

## Design a Key-Value Store
* Leader-Leader replication is **better for writes** and Leader-Follower, because multiple db copies can accept writes
<img width="970" alt="Screenshot 2025-01-23 at 5 21 18 PM" src="https://github.com/user-attachments/assets/8de42f66-15ea-44f0-82b0-eb16fd00a3cc" />
### PACELC theorem implications 
* try and describe pac vs elc, if you can't look it up, think about an example where a node goes down and the latency it takes to update a leader-follower even when things aren't down. consistence takes time. 
* we can use a "quorum" to decide how much latency vs consistency we should have. AKA should we wait for 1, some, or ALL of the nodes
* Databases like **Cassandra and MongoDB** can have custom quorum values or "tunable consistency"
#### Circular/Consistent Hashing for database sharding
* When using consistent hashing in distributed systems, virtual nodes (vNodes) are often employed to improve load balancing and fault tolerance.
* What They Are: Instead of assigning one contiguous range of the hash space to each physical node, multiple smaller virtual nodes are created per physical node.
* How they work: A physical node is responsible for multiple vNodes spread across the hash ring.
* When a node joins or leaves, only its vNodes are redistributed, making the rebalancing more granular and efficient.
#### Detecting node failures
* **zookeeper** is a tool you can use, all nodes could send heartbeats to zookeeper
* a more decentralized way is to use **gossip protocol** where db nodes keep track of one anoteher
* HOWEVER you can't rely on one node to determine, since THAT NODE might be the one taht disconnected, so we need MANY nodes to report the disconnect
#### Concurrent writes
* you take the latest write and use that, easy as that


## Design a Distributed Message Queue
<img width="757" alt="Screenshot 2025-01-24 at 12 39 03 PM" src="https://github.com/user-attachments/assets/26de6f29-5c3e-4e86-be19-63555f8b1209" />

* its all about **producers and consumers** and **decoupling services**
* can be used for **batch processing** or **job queues**
* Kafka is a little **more** than this, it's a **pub/sub event streaming platform**
#### Pub/Sub
* consumers subscribe to **Topics**, producers post to Topics
* Different delivery types **as least once**, it's definitely getting there, **at MOST once**, we don't want duplicates and **exactly once**, self explanatory but pretty complicated and has it's tradeoffs
* 

