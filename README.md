```bash
export digitalocean_token=***********

export driver_ops="--driver=digitalocean \
  --digitalocean-region=fra1 \
  --digitalocean-access-token=$digitalocean_token \
  --digitalocean-size=1gb"
```

```bash
docker-machine create $driver_ops demo-infra
eval $(docker-machine env demo-infra)
```

```bash
cd build-ship-run
# build
docker build -t angeldimitrov/build-ship-run .
# ship
docker push angeldimitrov/build-ship-run
# run
docker run angeldimitrov/build-ship-run
```

```bash
# install consul on the stand-alone node
docker-compose -f docker-compose-consul.yml up -d
```
```bash
# create the swarm with 3 nodes
docker-machine create $driver_ops \
    --swarm \
    --swarm-master \
    --swarm-discovery="consul://$(docker-machine ip demo-infra):8500" \
    --engine-opt="cluster-store=consul://$(docker-machine ip demo-infra):8500" \
    --engine-opt="cluster-advertise=eth0:2376" \
    demo-swarm-node-01
    
docker-machine create $driver_ops \
    --swarm \
    --swarm-master \
    --swarm-discovery="consul://$(docker-machine ip demo-infra):8500" \
    --engine-opt="cluster-store=consul://$(docker-machine ip demo-infra):8500" \
    --engine-opt="cluster-advertise=eth0:2376" \
    demo-swarm-node-02
    
docker-machine create $driver_ops \
    --swarm \
    --swarm-master \
    --swarm-discovery="consul://$(docker-machine ip demo-infra):8500" \
    --engine-opt="cluster-store=consul://$(docker-machine ip demo-infra):8500" \
    --engine-opt="cluster-advertise=eth0:2376" \
    demo-swarm-node-03
```

```bash
# add metrics aggregation
cd swarm-compose/monitoring
docker-compose up -d
# scale cadvisor over all the nodes
docker-compose scale cadvisor=3
```

```bash
# add logging aggregation
cd swarm-compose/logging
docker-compose up -d
docker-compose scale logspout=3
```
