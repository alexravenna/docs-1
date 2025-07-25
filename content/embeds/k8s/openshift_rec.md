```yaml
apiVersion: app.redislabs.com/v1
kind: RedisEnterpriseCluster
metadata:
  name: rec
  labels:
    app: redis-enterprise
spec:
  # The number of Redis Enterprise nodes in the clusters.
  nodes: 3

  persistentSpec:
    # Whether to enable persistent storage for the Redis Enterprise nodes.
    enabled: true

    # The size of the persistent volume for each Redis Enterprise node.
    volumeSize: 20Gi

  # The resources allocated to each Redis Enterprise node.
  redisEnterpriseNodeResources:
    requests:
      cpu: 2
      memory: 4Gi
    limits:
      cpu: 2
      memory: 4Gi
  redisEnterpriseImageSpec:
    repository: registry.connect.redhat.com/redislabs/redis-enterprise
    versionTag: 7.22.0-216
  redisEnterpriseServicesRiggerImageSpec:
    repository: registry.connect.redhat.com/redislabs/services-manager
  bootstrapperImageSpec:
    repository: registry.connect.redhat.com/redislabs/redis-enterprise-operator
```
