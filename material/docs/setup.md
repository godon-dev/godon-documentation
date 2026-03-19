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

## Installation
---------------

!!! warning "Early Stage"

    Godon is in early development. Helm charts and APIs may change without notice.

### Prerequisites

- Kubernetes cluster (1.20+)
- Helm 3.8+

### Install

```bash
helm install godon oci://ghcr.io/godon-dev/charts/godon
```

For Helm 2.x or 3.0-3.7:

```bash
helm repo add godon https://godon-dev.github.io/godon-charts/charts
helm install godon godon/godon
```

### Uninstall

```bash
helm uninstall godon
```

### Resources

- [Chart Source](https://github.com/godon-dev/godon-charts)
