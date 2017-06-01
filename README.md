# redisconf2017

Notes from redisconf 2017.
May 30–June 1, 2017
Marriott Marquis, San Francisco

## Day 1

### Redis Community Updates (Salvatore Sanfilippo)

#### redis 4.0

psync2 Bug (alibaba)

rdb module values

modules thread safe contexts

UNLINK: threaded non blocking DEL

modules:
  - redis as framework to build things.
  - extend redis

threaded contexts: non blocking O(N) operations

LRU + LFU

RDB + AOF

_RC now, AG in a few weeks_

#### redis 4.2

1. streams

2. more threading

3. better clienting (wg??)

4. client side caching

DISQUE

#### streams

time series (+ random access)

multiple consumers problem

#### sorted set

wasteful if not good Use

#### pub-sub

ADT: streams are logs

one entry per lineordered

entries are hashes

offsets are logical (entry unique id)

### Joshua McKenty:  Death to Conway's law

nasa, openstack, pivotal

1. share
2. clean up
3. say sorry

Kids today do not roll vcr.  
we need things instant  

### Redis Flash on K8s Demo (google)

taint nodes

kubectl port-forward

google assistant using redis cluster api (awesome!)

load test jobs. overflow ram, and end up in flash.

redis enterprise

### open source discussion with google PM

we don't like each other, but we respect each other.

dram database replacing disk database?

1 maintainer for ntp.  
lacking fund. infrastructure.

### Redis stream (Salvatore Sanfilippo)

radix tree  
faster than dict.c  
stream data strucutre in 4.2  
listpack  
redis event loop  
modules threading
  - key-locking
  - read-write locks
  - auto blocking of client accessing busy keys

### Redis at Scale (Square)

LXC contaner  

persistence not guaranteed  
reply on persistence of slaves and failovers  
RDB BGSAVE severe performance hit for large datasets  
never perform on active master ==> disable auto BGSAVE  

RDB and AOF disabled  
never restart redis automatically  
never auto join cluster  

`Success over`  

Snapshot only as warm ups. Failover as persistence.

ghostunnel  
https://github.com/square/ghostunnel  
authentication  
certificate hotswapping  
redis.sock accessible only by root  

redis cluster  
ha in the event of outage due to distributed design  
consistency at cost of performance  
can not enforce consistency in network partition  

global readonly  
redis lacks  
min-slave-to-write config  

square internal patches  
native ssl/tls support

### Rediseach module

### Redis at Roblox

general purpose LRU cache  
product feature data store  
rate limiting  
message relay  

stackexchange redis client  
signalir  

`sub-pub`

chat web server  
use storage events to set one tab as leader  
others follow from leader tab  
==> one session to rt web server

distributed cache invalidation  
- local cache on every machine
- every invalidation broad cast to all machines
- reaching a saturation point
- more traffic if network issue

redis pub-sub?

### nighthawk (twitter)

#### cahce as a service
redis at core
  * 10 M QPS
  * 10k nodes
  * 10TB data

#### Usage
  * Analytics 0 ads, video
  * ad serving
  * ad exchange
  * direct messaging
  * mobile app conversion tracking

#### architecture

client <--> proxy <--> backend redis <--> persistent storage  
topology and cluster manager  


move one partition a time (15 min or 2 hours. configurable.)

vs client managed partitioning
  - thin client
  - clients, proxy, backends scale independently.
  - custom lb
  - no cluster downtime during scaling out/up/backends

```
ha with replication
rf = 2, intra dc
no support for cross dc yet
idempotent operations only 0 get, put, remove etc.
copies of partition never on same host and rack
passive warming for failed/restarted replicas
```

```
diff pools of backends
writes are send to both pools
read from one pools
```
#### hot key mitigation

- sampling small percentage of requests and logging
- post processing the logs to identify high frequency keys
- client side key detection and caching

use rate-limited copy when warming replicas  
esp with active node - you dont want to give it more load during its replication

qa:

topology stored in zookeeper

memecached not good for replication

hashing happens at proxy  
partition status stored at zookeeper

wg:  
looks not open sourced?  
a ref:    
http://highscalability.com/blog/2014/9/8/how-twitter-uses-redis-to-scale-105tb-ram-39mm-qps-10000-ins.html

## Day 2
