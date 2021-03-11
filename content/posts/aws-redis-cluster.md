---
title: AWS Implementation of Redis Cluster
date: 2020-05-04
categories:
  - Redis
tags:
  - redis
  - aws
---

## AWS redis cluster topology

Refer to [2]

## Number of connections to master and slave nodes

For each node there are two connection related metrics: NewConnections and CurrConnections.

- NewConnections, AWS ElastiCache derives this by subtracting two consecutive samples of the total_connections_received stats of a redis node.
- CurrConnections, this metric is derived directly from the redis stats connected_clients which contains the number of client connections(excluding the connections from replicas)

## Redirect

\> Normally slave nodes will redirect clients to the authoritative master for the hash slot involved in a given command, however clients can use slaves in order to scale reads using the[ READONLY](https://redis.io/commands/readonly) command.

[Learn Why Redis Client Read Requests Are Read From or Redirected to the Master Node of a Shard](https://aws.amazon.com/premiumsupport/knowledge-center/elasticache-redis-client-readonly/)

## Go-redis source code

Code path: github.com/go-redis/redis/cluster.go

**Entities**

- ClusterClient
  - opt        *ClusterOptions
  - nodes       *clusterNodes
  - state       *clusterStateHolder
  - process        func(Cmder) error

- ClusterOptions
  - Addrs      []string
  - MaxRedirects    int
  - ReadOnly    bool
  - RouteByLatency  bool
  - PoolSize    int

- clusterNodes
  - allAddrs   []string
  - allNodes   map[string]*clusterNode

- clusterNode
  - Client    Client (Including connPool)

- clusterStateHolder
  - load     func() (*clusterState, error)
  - State    atomic.Value (-> *clusterState)

- clusterState


![img](topo.png)


**Procedure**

- Cluster client initialization

- - ClusterOptions is passed to initialization the ClusterClient, in our case, the `Addrs` option is the AWS elasticache upfront domain address
  - The elasticache upfront address is stored in `clusterNodes.allAddrs` as the only address

- State construction - during the first time a redis cmd is called

- - -> (*clusterStateHolder).Get() -> (*clusterStateHolder).Reload() -> (*clusterStateHolder).load()

  - -> traverse (*clusterNodes).allAddrs, in our case only the elasticache upfront address, pick this addr to create the clusterNode object and connect to that node

  - - Note elasticache upfront address could resolve to anyone of the ip addresses behind it, that doesn’t matter. Go-redis will use whatever it returns to get the full snapshot of the redis cluster

  - -> a new clusterNode accompanied with a new client: -> NewClient() -> newConnPool() (means there will be multiple connections but up to poolSize established to each cluster node)

  - -> retrieve all slots info through the new connected clusterNode by using the `cluster slots` cmd(this is the recommended way by redis official[3])

  - Creates clusterNodes and connects to each node returned in the slots. Also maintains the relationships of slot -> clusterNodes

- Connection Pool

- - Each clusterNode object represents a client to the corresponding cluster node
  - Each client in go-redis maintains a connection pool to the corresponding cluster node

- Cmd flow after state construction:

- - -> (*ClusterClient).process(Cmder)

  - -> (*ClusterClient).cmdSlotAndNode(Cmder) calculate slot and choose the corresponding node

  - - If readonly enabled, prioritarily choose the slave node under that slot
    - Otherwise choose master node (this is why master nodes usually get more connections than slave nodes)

  - Find a clusterNode by calling (*ClusterClient).cmdSlotAndNode(Cmder). This method ensures the correct node of the slot is returned.

## References

1. [Redis Cluster Specification](https://redis.io/topics/cluster-spec)
2. Scaling Your Redis Workloads with Redis Cluster: https://youtu.be/3Ovx5vJ17ws
3. https://redis.io/topics/cluster-spec#clients-first-connection-and-handling-of-redirections
