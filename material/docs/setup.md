<!--
Copyright (c) 2019 Matthias Tafelmeier.

This file is part of godon

godon is free software: you can redistribute it and/or modify
it under the terms of the GNU Affero General Public License as
published by the Free Software Foundation, either version 3 of the
License, or (at your option) any later version.

godon is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU Affero General Public License for more details.

You should have received a copy of the GNU Affero General Public License
along with this godon. If not, see <http://www.gnu.org/licenses/>.
-->

## Setup
---------------

!!! warning "Early Stage"

    Godon is in early development. Helm charts and APIs may change without notice.

Godon deploys as a Helm chart into Kubernetes. Two deployment modes are supported:

| | **Local Node** | **Existing Cluster** |
|---|---|---|
| Cluster | kind on a single machine | Your existing cluster (1.20+) |
| Storage | Host bind mount (overlay bypass) | Standard PVCs / storage classes |
| Helm command | Identical | Identical |

Both modes use the same Helm chart from [godon-charts](https://github.com/godon-dev/godon-charts) --- the difference is how the target cluster is provisioned and whether storage overrides are needed.

### Local Node Deployment

A local node deployment uses [kind](https://kind.sigs.k8s.io/) to run a Kubernetes cluster on a single machine. A working configuration --- including the storage bypass YugabyteDB requires --- is maintained in the [godon-charts](https://github.com/godon-dev/godon-charts) repository.

#### Prerequisites

- [Docker](https://docs.docker.com/get-docker/)
- [kind](https://kind.sigs.k8s.io/docs/user/quick-start/#installation) (v0.20+)
- Helm 3.8+

#### Step 1 --- Prepare the charts

```bash
git clone https://github.com/godon-dev/godon-charts.git
```

#### Step 2 --- Create a local cluster

Create the host storage directory and spin up a kind cluster using the provided configuration (1 control plane + 2 worker nodes):

```bash
sudo mkdir -p /srv/storage
kind create cluster --config godon-charts/kind_config.yaml
kubectl create namespace godon
```

`/srv/storage` must exist **before** launching kind. The `kind_config.yaml` bind-mounts this path into every node, which is essential for bypassing kind's virtual overlay filesystem (explained below).

#### Step 3 --- Install the godon stack

For modern Helm (3.8+):

```bash
helm install godon oci://ghcr.io/godon-dev/charts/godon \
  --namespace godon
```

For older Helm (2.x, 3.0--3.7):

```bash
helm repo add godon https://godon-dev.github.io/godon-charts/charts
helm install godon godon/godon --namespace godon
```

#### Step 4 --- Verify

Check that all pods are running:

```bash
kubectl get pods --namespace godon
```

The godon API is available at `localhost:7080` (forwarded through the kind control plane node).

#### Applying environment overrides

The `stacks/` directory in godon-charts contains environment-specific value overrides. For example, to apply the test stack profile:

```bash
helm install godon oci://ghcr.io/godon-dev/charts/godon \
  --namespace godon \
  -f godon-charts/stacks/osuosl/test-stack.yaml
```

!!! info "Why the storage bypass matters"

    kind nodes run inside Docker containers and use an overlay filesystem for their virtual disks. This overlay adds significant I/O overhead, which causes the archive database (YugabyteDB) to fail its startup health probes due to slow disk sync operations --- timing out and crash-looping.

    On an existing cluster with real disks this is not a concern. The local deployment avoids it with three coordinated mechanisms:

    1. **Host bind mount** --- `extraMounts` in `kind_config.yaml` maps `/srv/storage` from the host machine into every kind node at the same path. Data written there lands directly on the host filesystem, bypassing the overlay.
    2. **hostPath volumes** --- the archive-db override in `test-stack.yaml` sets `ephemeral: true` to skip PVC creation and instead mounts `hostPath` volumes pointing to `/srv/storage/yugabytedb/{master,tserver}`. YugabyteDB writes its data directories through these hostPath mounts.
    3. **Health check skip** --- `skipHealthChecks: true` disables the YugabyteDB disk sync probes that would otherwise fail in the kind environment, while keeping all other monitoring active.

    Without this bypass, YugabyteDB will not stabilize in kind. The metadata database (PostgreSQL) is unaffected because it has less demanding I/O requirements and tolerates the overlay.

### Existing Kubernetes Cluster

On an existing cluster, no storage workarounds are needed --- the Helm chart uses standard PVCs and YugabyteDB health probes work as intended.

#### Prerequisites

- Kubernetes cluster (1.20+)
- Helm 3.8+

#### Install

```bash
helm install godon oci://ghcr.io/godon-dev/charts/godon
```

For Helm 2.x or 3.0--3.7:

```bash
helm repo add godon https://godon-dev.github.io/godon-charts/charts
helm install godon godon/godon
```

### Uninstall

```bash
helm uninstall godon --namespace godon
```

To tear down a local kind cluster:

```bash
kind delete cluster --name godon-1-cluster
```

### Resources

- [Chart Source](https://github.com/godon-dev/godon-charts)
