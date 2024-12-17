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

* 
