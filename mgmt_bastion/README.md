## Management bastionhost deployment

This deploys a swiss-knife pod with many tools preinstalled.

If you need to "jump" on one of the nodes, add the ssh key required to ssh. Here, the key is the default one, change the path to add any other key.

You can add the key with an imperative approach:

```
kubectl create secret generic nodes-ssh-key --from-file=/root/.ssh/id_rsa.nodes
```

or with a declarative approach, note how the `ssh-privatekey` is set to `-` while a new key is then deployed, that's to set the key name to `id_rsa`

```
cat << EOF > /etc/systemd/system/docker.service.d/http-proxy.conf
apiVersion: v1
kind: Secret
metadata:
  name: nodes-ssh-key
type: kubernetes.io/ssh-auth
stringData:
  ssh-privatekey: |
          -
data:
  id_rsa: |
    <paste the output of `cat /root/.ssh/id_rsa.nodes | base64 -w0` here>

EOF
```

Next, create the bastionhost/jump server with

```
kubectl apply -f /root/git/Dockerfiles/mgmt_bastion/jump_pod.yaml
```

The manifest is also setting some AWS variables, to set those variables while applying it use `envsubst`:

```
export AWS_ACCESS_KEY_ID=AKIAQKXXXXXXXXXX
export AWS_SECRET_ACCESS_KEY=FakeFakeFakeFakeFakeFakeFakeFake
export AWS_DEFAULT_REGION=us-west-1

envsubst < deployment.yaml | kubectl apply -f -

unset AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY AWS_DEFAULT_REGION

```

To jump on a server or a node, provided access is allowed at network level, run

```
kubectl exec -it bastionhost -- ssh 172.20.1.20 -l ubuntu -i /etc/id_rsa.nodes/id_rsa
``` 

NOTE how the file is mounted into a folder `id_rsa.nodes` while the file is just `id_rsa`.

## Deployments

There are many bastionhost deployments, the pod deployed is always the same, it's just customised for different platform/requirements.

The `bh_eks.yaml` manifest is deploying the pod on AWS EKS using the EBS storage

The `bh_microk8s.yaml` manifest is deploying the pod on microk8s, using a custom StorageClass which allows to define any directory as local storage

The `bh_microk8s-default_pvc.yaml` is the same as above, but it's using the default microk8s StorageClass, which is by default creating a local directory in `/var/snap/microk8s/common/default-storage/`.
