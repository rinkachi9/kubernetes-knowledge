# Container technologies

Container engines and runtimes outside the Kubernetes API: what they are, how they relate to the CRI runtimes on Kubernetes nodes, and when they matter. This section is not a page of the Kubernetes documentation; it holds the technologies that the Containers chapter assumes.

## Notes in reading order

1. [Docker](docker.md): the Docker stack, Docker and Kubernetes after the dockershim removal, building images for clusters.
2. [LXC](lxc.md): Linux system containers and how they differ from application containers.

## Planned topics

- containerd
- CRI-O
- Podman and Buildah
- OCI image, runtime and distribution specifications
- Sandboxed runtimes: gVisor and Kata Containers
