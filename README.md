# Hypershift POC

All the following assumes `KUBECONFIG` points to the management cluster.

Build binaries and push an image for the control plane operator:

```
$ make
$ docker build -t quay.io/my/hypershift:latest .
$ docker push quay.io/my/hypershift .
```

Install the operator's supporting resources into the management cluster:
```
$ oc apply --filename manifests/
```

Define release image info to be referenced by clusters:

```
hack/generate-release-images.rb | oc apply --filename -
```

Run the operator:
```
$ bin/hypershift-operator run --control-plane-operator-image quay.io/my/hypershift:latest
```

Create a cluster, referencing a release image present in the `release-images` configmap
previously created:

```yaml
apiVersion: hypershift.openshift.io/v1alpha1
kind: OpenShiftCluster
metadata:
  namespace: hypershift
  name: guest-hello
spec:
  releaseImage: quay.io/openshift-release-dev/ocp-release@sha256:d78292e9730dd387ff6198197c8b0598da340be7678e8e1e4810b557a926c2b9
  baseDomain: guest-hello.devcluster.openshift.com
  pullSecret: '{"auths": { ... }}'
  serviceCIDR: 172.30.0.0/16
  podCIDR: 10.128.0.0/14
  sshKey: 'ssh-rsa ...'
  cloudProvider: AWS
  computeReplicas: 1
```

Get the cluster kubeconfig using:
```
$ oc get secret --namespace guest-hello --template={{.data.kubeconfig}} | base64 -D
```

And delete the cluster using:

```
$ oc delete openshiftclusters/guest-hello
```
