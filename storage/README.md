# Storage

Ways to provide both long term and temporary storage to Pods: volumes and their types, persistent volumes and claims, storage classes and the Container Storage Interface. Corresponds to the "Storage" chapter of the Kubernetes documentation Concepts.

## Notes in reading order

1. [Volumes](volumes.md): why volumes exist, volume types, `subPath`, resources, CSI, mount propagation, read-only mounts.

## Planned topics

- Persistent Volumes and PersistentVolumeClaims (access modes, reclaim policy, binding)
- Projected volumes
- Ephemeral volumes (generic and CSI ephemeral)
- Storage classes
- Volume attributes classes
- Dynamic volume provisioning
- Volume snapshots and volume snapshot classes
- CSI volume cloning
- Volume populators and data sources
- Storage capacity
- Node-specific volume limits
- Local ephemeral storage
- Volume health monitoring
- Windows storage
