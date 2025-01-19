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
TRY TO DEFINE THE TABLES!!! WHATS IN THEM???/
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

---

### **5. Observability and Monitoring**
   - **Logging and Metrics**:
     - Define key performance indicators (KPIs) for system health (e.g., CPU usage, error rates, latency).
   - **Tracing**:
     - Represent end-to-end request tracing for debugging and performance analysis.
   - **Alerts**:
     - Define critical alerts (e.g., high latency, node failure, partition issues).

---

### **6. Deployment and Infrastructure**
   - **Cloud/On-Prem**:
     - Specify where the system is hosted (e.g., AWS, Azure, GCP, or on-premise data centers).
   - **Compute Resources**:
     - Highlight the compute layer: VMs, Kubernetes clusters, serverless functions, etc.
   - **Load Balancing**:
     - Describe traffic distribution strategies (e.g., DNS-based, L7 load balancers).
   - **Network Architecture**:
     - Illustrate regional distribution, edge nodes, and CDN integration.
   - **CI/CD Pipeline**:
     - Briefly indicate deployment strategies (e.g., blue-green, canary deployments).

---

### **7. Fault Tolerance and Reliability**
   - **Redundancy**:
     - Describe replication mechanisms for fault tolerance (e.g., primary-secondary, sharded replication).
   - **Error Handling**:
     - Plan for dead letter queues, retries, and circuit breakers for failures.
   - **Disaster Recovery**:
     - Define recovery strategies (e.g., warm/cold standby, multi-region replication).

---

### **8. Diagram Specifics**
   - **Component Overview**:
     - Include core system components (e.g., APIs, databases, services, storage layers).
   - **Data Flow**:
     - Illustrate how data flows through the system, including inputs and outputs.
   - **Key Interactions**:
     - Highlight integrations with external systems or dependencies (e.g., payment gateways, third-party APIs).

---

### **9. Future Considerations**
   - **Scalability Potential**:
     - Note how the system is designed to scale (horizontal vs. vertical scaling).
   - **Extensibility**:
     - Show areas where the architecture allows for feature additions or modular upgrades.

---

### **10. Documentation**
   - Ensure the diagram is accompanied by a glossary of terms, detailed descriptions of components, and a legend for symbols used.

By including these details, the architecture diagram provides a holistic view of the system, making it easier to evaluate, maintain, and improve.

## Random facts
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
* fjiles need ot be private
* block level storage is cheaper, must be reassembled, needs KV store for it maybe, we can do "deduplication" which doens't store du;licatte blocks, could be done on a global level or EVERY user, multi-user!! also allows for CHUNKING (save partial upload)
* paste my overviews to notes
* FILE SYSTEM VS OBJECT STORE (edit capabilities)
* look up content addressable sotrage
* folders in a kv store? whyu?
* file also has reference to folder, why not just do it in name like AWS?
* maybe we could implement a garbage colleciton sserverice that goes and tracks how manuy users are using the file in the kv sotre, and perdodically goes through and delted duplicates
* can setup backup loadbalancers iwth a heartbeat ping, can use **Zookeeper** to coordiante heartbeat and other distibuted system stuff

## Design Google Maps
* make a list of requirements that EVERY ARCHITECHTURE DIAGRAM NEEDS, DAU/scale reads and writes , latency (how fast should a user expect a response), availability (will the system always be available to use), partition safety (i tie this in with availbility, so natural disasters, network disconnects/partitions), CAP and PACELC, data storage (s3, nosql, sql) IM ADDING THIS TO THE TOP, CLEAN THIS UP LATER
* review dkstras and some graph algorithsm
* read about spacial indexing and determing its most general uses, he says its upported by most databases
* look up kafka vs kinesis, when do we need these?
* 

