Kustomization file supports patching:

- patchesStrategicMerge: A list of patch files where each file is parsed as a [Strategic Merge Patch](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-api-machinery/strategic-merge-patch.md).
- patchesJSON6902: A list of patches and associated targets, where each file is parsed as a [JSON Patch](https://tools.ietf.org/html/rfc6902) and can only be applied to one target resource.

`patchesStrategicMerge` we cannot add one item to the list, it will overwrite all the items with the new one. We using `patchesJSON6902` instead.

Run the example in this repo:

```bash
$ kustomize build .
```

The result:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx-deployment
spec:
  replicas: 300
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - env:
        - name: var2
          value: VALUE_2
        - name: var1
          value: VALUE_1
        image: nginx:1.14.2
        name: nginx
```

Please take a look at the [base](base) and [prod](prod).

- [./base/deployment.yaml](./bash/deployment.yaml)

  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: nginx-deployment
    labels:
      app: nginx
  spec:
    replicas: 3
    selector:
      matchLabels:
        app: nginx
    template:
      metadata:
        labels:
          app: nginx
      spec:
        containers:
        - name: nginx
          image: nginx:1.14.2
          env:
            - name: var1
              value: PATCH_ME
  ```

- [./prod/patch-deployment.yaml](./prod/patch-deployment.yaml)

  ```yaml
  - op: replace
    path: /spec/template/spec/containers/0/env/0
    value:
      name: var1
      value: VALUE_1
  - op: add
    path: /spec/template/spec/containers/0/env/0
    value:
      name: var2
      value: VALUE_2
  ```

- [./prod/kustomization.yaml](./prod/kustomization.yaml)

  ```yaml
  resources:
    - ../base

  patchesJson6902:
    - path: ./patch-deployment.yaml
      target:
        group: apps
        version: v1
        kind: Deployment
        name: nginx-deployment

  patchesStrategicMerge:
    - |-
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: nginx-deployment
      spec:
        replicas: 300
  ```
