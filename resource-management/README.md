# Resource management

How the kubelet manages node level resources beyond CPU and memory requests: CPU, memory and topology managers that align resources for latency sensitive workloads, and dynamic resource allocation (DRA) for devices such as GPUs. Corresponds to the "Resource Management" chapter of the Kubernetes documentation Concepts. Requests and limits of containers live in [Configuration](../configuration/resource-management-for-pods-and-containers.md).

TODO: Write the notes of this area; there is no material for it yet.

## Planned topics

- Resource managers (CPU manager, memory manager, topology manager)
- Dynamic resource allocation (DRA API objects, how DRA works, DRA features, device taints and tolerations, observability)
- Pod-level resource managers
