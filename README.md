# RedisConf 2017

Notes from RedisConf 2017.
May 31–June 1, 2017
Marriott Marquis, San Francisco

Note I did not attend all the sessions, because they are parallel sessions..

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

### Cofounder Redis labs

primary db for enterprise  

redis flash dbaas  
redis cloud private  

redis flash  
dram: keys  and hot values  
ssd: cold values  
rdb/aof: persistent storage  
flash ratios  

NVMe  

modules  
c/c++, go, python  
multi-threaded  

redis-graph  

redis-ML  
  serving the model  

ReJSON  
any database store json  
native json  

RediSearch  

IoT  

multi-master geo-distributed  
geo-distributed latency 70ms  
conflict-free replicated data type  
consensus free with strong eventual consistency  
write locally, converge asyn  

CRDB  
bidirectional replication  

### Redis-ML and Spark (databrics)

big data is missing link  
hardest part of ai is big data  

data --> spark training --> fs --> customer server <-- client app  
data loaded in spark  
model saved in files  
model loaded to custom app  
serving client  

spark missing last mile: serving the model  
==> Redis-ML  

spark summit

### pattern language for microservices (eventuate.io)

chris richardson

microservices patterns  

functional decomposition  
api gateway  

easily try other technologies  
and fail safely...  

cache

no silver bullet  
complexity  
development: ipc, partial failure, distributed data  
testing: integration, end to end  
deployment  

always ask good fit for my application?  

pattern: reusable solution  

microservices.io  

distributed data mgmt  
loose coupling = encapsulated data  
queries across multiple db  
can not use acid transactions  
2 phase commit is not an option  
instead use event driven sagas  
saga-based eventually consistent order processsing  

### observability and glorious future: future of observability in complex systems

data reliability engineering  
dba ops  
hates monitoring  
honeycomb facebook  
not monitoring company  

monitoring --> observability  

unreliable symptoms or reports  
as soon as we know the question, we usually know the answer too.  

full stack instrumentation  
you need strace for systems

symptoms not problems  

no more easy problems in the future, only hard problems

if there is a schema or index, it is not future proof  

context is everything, preserve it  

debugging with data, not eyeballs  
debugging is a social act.  

event-driven, not pre-aggregated  
hight cardinality  

### REdis in Kubernetes (amadeus)

reids operator  
thrid party resources  
specifity hidden in a new k8s object  

#### next
opensource redis-manager  
this month  
migrate redis-manager logic in an operator  
package helm chart  
integrate with k8s service catalogue

easy deployment using k8s
automation using redis-manager

using redis as session store, no persistent storage

### Redis-ML

performance  
simplicity through data structures  
extensibility through modules

any c/c++ program run on redis  

gini impurity

majority vote  

docker pull shaynativredis-ml

### Redis Graph

node  
relations  

store  
attributes hash  

graph.addnode  
graph.addedge  
edge store  

Hexastore  
subject predicate object  

```
match <graph_name> <graph_pattern>
where <filters>
return <entities>
```

query --> parser --> ast --> exec plan  

tokenizer - lex  

graph.explain graph_name query  
my queries are running slow.  
create index on it.

### multi-master

CRDT  
conflict free replicated data types  
academic research  
consensus free protocol  
strong eventual consistency

vector clocks  
maintain global ordering  
if need to do any reconciliation

demo at github  

support different cluster topology

counter  
multi increase --> add together   
add wins / observed remove  

strings: last write wins  

local clock synced  

### Image View count at imgur

hbase --> redis  

tcp syslog stream (fastly cdn)  
--> node ingest service  
--> redis 3.2 cluster   
--> API service  
--> imgur / internet

ingest service:  
parse syslog lines, report metrics via statsd  

hbase --> node backfill service --> redis 3.2 cluster  

faster: 50 ms
cheaper: reduce 75%
simpler: incr get

devops and platform team  

php7 redis client

key redirection adds latency
