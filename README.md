# Neo4j causal cluster + Kubernete + Data model sample

## Configure Kubernet

* Create the statefulset controllers for the core server. In this case we are creating 2 core servers.
* Create the headless service to allow the nodes to see each other.
* Create a loadbalancer to expose the ports needed to access from outside.

Image used neo4j:3.4.1-enterprise

```
$ kubectl apply -f cores
service "neo4j" created
service "neo4j-loadbalancer" created
statefulset "neo4j-core" created
```

-Create 1 replica using a Deployment type.

```
$ kubectl apply -f read-replicas/
deployment "neo4j-replica" created
```

Make sure that everything was created and running.

Check Pods out

```
$ kubectl get pods
NAME                            READY     STATUS    RESTARTS   AGE
neo4j-core-0                    1/1       Running   0          2m
neo4j-core-1                    1/1       Running   0          2m
neo4j-replica-c88f7bdb6-dsvjz   1/1       Running   0          46s
```

Check Services out

```
$ kubectl get services
NAME                 TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)                         AGE
kubernetes           ClusterIP      Y.Y.Y.Y         <none>          443/TCP                         3d
neo4j                ClusterIP      None            <none>          7474/TCP,6362/TCP               6m
neo4j-loadbalancer   LoadBalancer   Z.Z.Z.Z         X.X.X.X         7474:30826/TCP,7687:32650/TCP   6m
```

You can now access the browser
http://X.X.X.X:7474

If a login page comes up enter the following information.

server: bolt+routing://X.X.X.X:7687
user: neo4j
pass: neo4j

Bolt+routing is needed to let neo4j redirect the traffic to the leader node. Otherwise you can get an error message, 
saying that you cannot write on a FOLLOWER node.


## Chat Data model sample

Let's crete an instant messages model.

User --> SEND -- > Message --> TO --> User

```

CREATE (leo:User { name: 'Leo', age: 35 })
,(wolverine:User { name: 'Wolverine', age: 137 })

CREATE (msg1:Message {id:'1', contents:'Hi Wol', day_sent: 'Monday'})
,(msg2:Message {id:'2', contents:'Hi Leo', day_sent: 'Monday'})

CREATE (leo)-[:SENT]->(msg1)
,(wolverine)-[:SENT]->(msg2)
,(msg1)-[:TO]->(wolverine)
,(msg2)-[:TO]->(leo)
,(leo)-[:FRIEND]->(wolverine)
```

```
MATCH path=(User)-[:SENT]->(Message)
RETURN path
```

```
MATCH (us1:User)-[r:SENT]->(msg:Message)
WHERE us1.name="Leo"
RETURN us1
```

```
MATCH (msg:Message)-[r:TO]->(us1:User)
WHERE us1.name="Leo"
RETURN msg
```

```
MATCH (us1:User)-[r:SENT]->(msg:Message)-[t:TO]->(usrTo:User)
WHERE usrTo.name="Wolverine"
RETURN msg
```

#### Remove nodes and relationships

MATCH (n) OPTIONAL MATCH (n)-[r]-() DELETE n,r

MATCH (user:User)
RETURN user.age AS oldest
ORDER BY user.age DESC
LIMIT 1