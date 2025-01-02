# high-level-system-design-exercise-notes

## Random facts
* 32GB of RAM is a lot for a personal computer, but for systems like Redis and Memcached, 256GB is entirely manageable. High-end servers can support up to 500GB or even 1TB of RAM, making them ideal for in-memory caching and real-time data processing at scale

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
  
* How are we doing **cache eviction** ( I like always using LRU (least recently used) unless there's a reason not to)
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
* Read through: On a cache miss, the cache **automatically** fetches the data from the origin server, stores it, and serves it to the application.
* Lazy-Loading: On a cache miss, the cache ** conditionally based on the application's stipultation** fetches the data from the origin server, stores it, and serves it to the application.
![image](https://github.com/user-attachments/assets/18223096-3947-4fcd-bc2f-1bff0a4ad4f5)

* Write through: Data is written to both the cache and the origin server simultaneously.
* Write around: Data is written directly to the origin server, bypassing the cache. The cache only gets updated when a subsequent read request causes a cache miss.
* Write back/behind: Data is written to the cache first and later asynchronously written to the origin server.

#### Cache invalidation
* LRU (Least Recently Used): best option imo, popular things will be sustained
* LFU (Least Frequently Used): Based on total visits, OLD viral things will be kept
* FIFO: queue, regardless of access patterns
* TTL: expires based on time


define fault tolerance

define redundance

discord or chat apps in general rely on low latency; how do we achieve this

practice rough calculations for iops/s gb/s , how many people in a server

per channel matters

maxes on msg for bytes/chars

