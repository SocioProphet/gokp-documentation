# Azure Quickstart

The following guide will install a GOKP cluster on your Azure Account.
Please read the entire guide before attempting and install.

# Prerequisites

The following are preqs. Since this is centered around [KIND](https://kind.sigs.k8s.io/) and [CAPI](https://cluster-api.sigs.k8s.io/), there will be a lot of similar prereqs. I recommend reading over them.

At a minimum:
* Azure Account ([Prereqs For Azure](https://capz.sigs.k8s.io/topics/getting-started.html#prerequisites))
* [GitHub Token](github-token.md)
* Docker on your workstation

Podman may or maynot work. It's [considered experemental by KIND](https://kind.sigs.k8s.io/docs/user/rootless/#creating-a-kind-cluster-with-rootless-podman) so YMVM.

Please make sure these are met before proceeding.

# Installing the Cluster

After you have [gotten the binary](../README.md#getting-the-binary), you can run the `create-cluster` command. You will need the following from Azure beforehand:

* Azure App ID
* Azure App Secret
* Azure Subscription ID
* Azure Tenant ID

```shell
gokp create-cluster azure --cluster-name=$MYCLUSTER \
--github-token=$GH_TOKEN \
--azure-app-id=$AZURE_APP_ID \
--azure-app-secret=$AZURE_APP_SECRET \
--azure-subscription-id=$AZURE_SUB_ID \
--azure-tenant-id=$AZURE_TENNANT_ID
```

Other options to note:

* `--gitops-controller` - By default this is set to `argocd`, but can also be set to `fluxcd`.
* `--private-repo` - By default this is set to `true`. Set it to `false` to create a public repo.
* `--azure-region` - Set this to the region you'd like to install to. By defualt this is set to `westus2`.

After about 40 min you should have a cluster ready to go. You'll have some information.

```shell
INFO[1037] Cluster Successfully installed! Everything you need is under: ~/.gokp/$MYCLUSTER
```

The Kubeconfig of this cluster is in this directory

```shell
export KUBECONFIG=~/.gokp/$MYCLUSTER/$MYCLUSTER.kubeconfig
```

Run `kubectl get pods -A` and you should see a Kuard sample application.

```
$ k get pods -A | grep 'kuard'
NAMESPACE     NAME                                                  READY   STATUS    RESTARTS       AGE
kuard         kuard-857f95f9df-99x87                                1/1     Running   0              4m43s
```

Run `kubectl get nodes` and you should have 3 controlplane and 3 worker nodes.

```
NAME                                      STATUS   ROLES                         AGE     VERSION
my-cluster-3192150395-control-plane-h4c2q   Ready    control-plane,master          12m     v1.22.2
my-cluster-3192150395-control-plane-l28qr   Ready    control-plane,master          10m     v1.22.2
my-cluster-3192150395-control-plane-z4h7j   Ready    control-plane,master          7m39s   v1.22.2
my-cluster-3192150395-md-0-gkq5n            Ready    worker                        10m     v1.22.2
my-cluster-3192150395-md-0-gvjdv            Ready    worker                        10m     v1.22.2
my-cluster-3192150395-md-0-wql7z            Ready    worker                        10m     v1.22.2
```

# Verifying Installation

Verifying installation will depend on which GitOps controller you installed. Select the one that's applicable to you.

* [Argo CD](argo)
* [Flux CD](flux)

# Adding Workloads

To add a workload, you can use the repo saved under `~/.gokp/$MYCLUSTER/$MYCLUSTER`. 

```shell
$ cd ~/.gokp/$MYCLUSTER/$MYCLUSTER
```

> :exclamation: The SSH key used to make commits/pushes is under `~/.gokp/$MYCLUSTER` and it ends with an `_rsa`
> you will need to add this via `ssh-add` or set the [GIT_SSH_COMMAND](https://git-scm.com/docs/git) environment variable.

There is a sample of what you can do under `cluster/tenants`.

```shell
$ ls -1 cluster/tenants/
kuard
```

Create your workload in this directory with all the needed artifacts.
For example, to deploy NGINX sample workload. You would first create
the directory.

```shell
mkdir cluster/tenants/nginx
```

Then I would create the YAML files for NGINX in this new directory. First the namespace.

```shell
kubectl create ns nginx \
--dry-run=client -o yaml > cluster/tenants/nginx/nginx-ns.yaml
```

Next the deployment.

```shell
kubectl create deployment nginx -n nginx --image=nginx \
--dry-run=client -o yaml > cluster/tenants/nginx/nginx-deploy.yaml
```

Then the service.

```shell
kubectl create service clusterip nginx --tcp=80:8080 -n nginx \
-o yaml --dry-run=client > cluster/tenants/nginx/nginx-svc.yaml
```

> Optionally, you can also create your `Kustomization` files here.

Commit these to your repo.

```shell
git add .
git commit -am "added new application"
git push
```

In a few minutes you should see that your workload is on the cluster now.

```shell
kubectl get deploy,svc,pods -n nginx
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   1/1     1            1           60s

NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/nginx   ClusterIP   10.131.140.13   <none>        80/TCP    60s

NAME                         READY   STATUS    RESTARTS   AGE
pod/nginx-6799fc88d8-77tmk   1/1     Running   0          59s
```

# Cleaning Up

To delete your cluster just run the following.

```shell
gokp delete-cluster azure \
--cluster-name=$MYCLUSTER \
--kubeconfig=/home/chernand/.gokp/$MYCLUSTER/$MYCLUSTER.kubeconfig
```

This will delete your cluster running in Azure. It does not delete the repo that was created, so you will have to do that yourself.
