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

3. better clienting

4. client side cacheing

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

### nighthawk twitter

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

partition

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

rate-limited copy

topology stored in zookeeper

memecached not good for replication

hashing happens at proxy
partition stored status at zookeeper

## Day 2
