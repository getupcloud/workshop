# Flux


## Installation

You can download the binary on the [GitHub release page](https://github.com/fluxcd/flux2/releases) or use the following script:

```
curl -s https://fluxcd.io/install.sh | sudo bash
```

Check the installation:

```
flux -v
```

## Examples

### 00 - Basic installation and components

- Create a basic cluster with [kind](../kind/README.md).

```
flux check --pre
```

- Install flux:

```
flux install
```

- Check the new objects created:

```
flux version
kubectl get pod -A | grep flux
kubectl api-resources | grep fluxcd
```

- Remove Flux components:

```console
flux uninstall
```

### 01 - Bootstrap with GitHub

- Create a basic cluster with [kind](../kind/README.md).

- With `flux bootstrap` command you can install flux and configure it to manage itself from a git repository. If Flux was previously installed, it will be upgrade if needed. The bootstrap is idempotent, it’s safe to run the command as many times as you want. 

```
flux bootstrap github \
  --owner=mmmarceleza \
  --repository=workshop \
  --branch=developer \
  --path=flux/examples/01 \
  --interval=1m \
  --personal
```
note: change the `--owner` parameter to match your user.

- Check all the changes in your repository on GitHub:
  - commits;
  - deploy keys on settings;
  - new files created on the path you specified before;

- Check all the changes in the cluster:
  - new objects of Flux;
  - 01-deploy on namespace default;

- Remove Flux components:

```console
flux uninstall
```  


### 02 - Using more git repositories

- Create a basic cluster with [kind](../kind/README.md).

- Bootstrap your main repository pointing to the second example:

```
flux bootstrap github \
  --owner=mmmarceleza \
  --repository=devops \
  --path=kubernetes/flux/examples/02 \
  --interval=1m \
  --personal
```

- Let's use podinfo as a second reference:


```console
flux create source git podinfo \
  --url=https://github.com/stefanprodan/podinfo \
  --branch=master \
  --interval=30s \
  --export > kubernetes/flux/examples/02/podinfo-source.yaml
```

- Commit the changes:

```console
git add kubernetes/flux/examples/02/podinfo-source.yaml
git commit -m "Add podinfo GitRepository"
git push
```

- Create a Flux kustomization:

```console
flux create kustomization podinfo \
  --target-namespace=default \
  --source=podinfo \
  --path="./kustomize" \
  --prune=true \
  --interval=5m \
  --export > kubernetes/flux/examples/02/podinfo-kustomization.yaml
```

- Commit the changes:

```
git add kubernetes/flux/examples/02/podinfo-kustomization.yaml
git commit -m "Add podinfo Kustomization"
git push
```

- Check all the changes in the cluster:

- Remove Flux components:

```console
flux uninstall
``` 

### 03 - Using Helmrelease - part 1

- Create a basic cluster with [kind](../kind/README.md).

- Bootstrap your main repository pointing to the third example:

```
flux bootstrap github \
  --owner=mmmarceleza \
  --repository=devops \
  --path=kubernetes/flux/examples/03 \
  --interval=1m \
  --personal
```

- Check all the changes in the cluster:

- Remove Flux components:

```console
flux uninstall
``` 

### 04 - Using Helmrelease - part 2

- Create a basic cluster with [kind](../kind/README.md).

- Bootstrap your main repository pointing to the third example:

```
flux bootstrap github \
  --owner=mmmarceleza \
  --repository=devops \
  --path=kubernetes/flux/examples/04 \
  --interval=1m \
  --personal
```

- Check all the changes in the cluster:

- Remove Flux components:

```console
flux uninstall
``` 

### 05 - Using image controlers

- Create a basic cluster with [kind](../kind/README.md).

- Bootstrap your main repository pointing to the third example:

```
flux bootstrap github \
  --components-extra=image-reflector-controller,image-automation-controller \
  --owner=mmmarceleza \
  --repository=devops \
  --path=kubernetes/flux/examples/05 \
  --interval=1m \
  --read-write-key=true \
  --personal
```

- Check all the changes in the cluster:

- Remove Flux components:

```console
flux uninstall
``` 
## Command to connect this repo with your cluster

- Create a Flux gitrepository:

```console
flux create source git devops \
  --url=https://github.com/mmmarceleza/devops.git \
  --branch=main \
  --interval=5m
```

- Create a Flux kustomization:

```console
flux create kustomization devops \
  --source=devops \
  --path="./kubernetes/flux/apps/" \
  --prune=true \
  --interval=5m
```
