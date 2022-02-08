# RabbitMQ cluster formation using single Docker Swarm service

## Prerequisites

## Cluster configuration summary

RabbitMQ cluster is configured using `classic_config`.

See details [here](https://www.rabbitmq.com/configure.html#config-file)

```
cluster_formation.classic_config.nodes.1 = rabbit@rabbitmq-1
cluster_formation.classic_config.nodes.2 = rabbit@rabbitmq-2
cluster_formation.classic_config.nodes.3 = rabbit@rabbitmq-3
```

in `rabbitmq.conf`.

- `rabbitmq-*` is container name thanks to service template `"rabbitmq-{{.Task.Slot}}"`. See details about temlates [here](https://docs.docker.com/engine/swarm/services/#create-services-using-templates)

- `rabbit@rabbitmq-1` is the expected node name from the point of view of RabbitMQ cluster.

## Demo

- Create docker config: 

`docker config create rabbitmq-config rabbitmq.conf`

- Create swarm network:

`docker network create --driver overlay prod-net`

- Deploy rabbitmq cluster as docker swarm service:

`docker stack deploy -c docker-compose.yml cluster`

- Find virtual ip of each container

`docker exec -i [container_ip] hostname -i`

- Modify `/etc/hosts` of each container to contain entries of other 2 containers:

For example for `cluster_rabbitmq.1...` container run:

`docker exec -it [rabbitmq1_container_ip] bash`
`bash-5.0# echo "10.0.1.9    rabbitmq-2" >> /etc/hosts`
`bash-5.0# echo "10.0.1.10    rabbitmq-3" >> /etc/hosts`

- Verify cluster is configured by running in any container:

`docker exec -it [rabbitmq1_container_ip] bash`

`bash-5.0# rabbitmqctl cluster_status`

You should see 3 nodes in cluster:

```
...
Running Nodes

rabbit@rabbitmq-1
rabbit@rabbitmq-2
rabbit@rabbitmq-3
...
```

Read more at [RabbitMQ cluster as a single Docker Swarm service](https://www.rokpoto.com/rabbitmq-cluster-single-docker-swarm-service/)
