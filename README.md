# Kubevious Helm Charts
**Kubevious** brings clarity and safety to Kubernetes. Kubevious renders all configurations relevant to the application in one place. That saves a lot of time from operators, enforcing best practices, eliminating the need for looking up settings and digging within selectors and labels.

For more information refer to the root repository: https://github.com/kubevious/kubevious

## Notable Changes
Unfortunately, versions 0.7.25 or lower are not upgradable to the newer versions of Kubevious (0.7.26+). That was the reason of the delay between versions 0.6 and 0.7, as we wanted to pack all non-upgradable changes into version 0.7. The following changes were made that prevents seamless upgrades. 

- Helm charts were rewritten, allowing significantly more freedom of deployment configuration choices 
- The database schema was redesigned to improve query and old data purge performance
- MySQL now deployed with autogenerated root and user passwords

Significant changes to the Helm chart values were also made. We no longer configure public cloud specific annotations and resources. With such a variaty of environment options, it was impossible to maintain. Please see complete table of [configurations](#configuration) below.

A two-way feedback mechanism was added to Kubevious. It includes version checks, news updates, useful hints and tips,  and reporting of errors, cluster size metrics, and internal time counters. Participants would also see their clusters on a https://worldvious.io leaderboard map. Location is anonymized to the nearest city/zip. No IP address is stored or logged. We calculate the SHA256 hash of the IP address and use it as a key in the backend. If, for some reason, you do not want to participate, please see details of reporting [configurations](#configuration) parameters and instructions to opt-out (it's super easy).

## Prerequisites

- Kubernetes v1.13 or higher
- Helm v3.2 or higher

## Installation 

First create a namespace:

```sh
kubectl create namespace kubevious
```

Add Kubevious repository and install the Helm chart:
```sh
helm repo add kubevious https://helm.kubevious.io
helm upgrade --atomic -i kubevious kubevious/kubevious --version 0.7.26 -n kubevious 
```

## Accessing Kubevious
Kubevious runs within your cluster. Upon successful completion of helm chart installation, you will see commands to access kubevious UI. There are two ways to access Kubevious UI. 

### Option 1. Access using port forwarding
The easiest but not most convenient method. Wait few seconds before pods are up and running. Setup port forwarding:

```sh
kubectl port-forward $(kubectl get pods -n kubevious -l "app.kubernetes.io/component=kubevious-ui" -o jsonpath="{.items[0].metadata.name}") 8080:80 -n kubevious  
```
Access from browser: http://localhost:8080

### Option 2. Expose using Ingress
Enable Ingress deployment using dedicated value parameters. See full list of [helm chart values](#configuration) to cofigure Ingress parameters.

```sh
helm upgrade --atomic -i -n kubevious \
    --version 0.7.26 \
    --set ingress.enabled=true \
    kubevious kubevious/kubevious
```

## Uninstalling the Chart
Undeploy from cluster:

```sh
helm uninstall kubevious -n kubevious
```

**IMPORTANT:** As requested by the community, now Kubevious Helm charts generate random MySQL root and user passwords. The Helm uninstall leaves behind the MySQL persistent volume. The same volume will be mounted if Kubevious is reinstalled into the same namespace using the same release name. That creates a big problem because Helm chart will generate new passwords for the backend to connect to MySQL, but the connection would fail because the mounted volume is initialized using the password generated using the initial installation. There are few solutions to this:

1. Delete the PersistentVolumeClain after *helm uninstall*:
```sh
$ kubectl delete pvc data-kubevious-mysql-0 -n kubevious
```

2. Install Kubevious providing your own root and user passwords. See *mysql.root_password* and *mysql.db_password* configuration values below.

3. Bit more complicated way is to update passwords in *kubevious-mysql-secret* and *kubevious-mysql-secret-root* Kubernetes secrets. Though wouldn't recommend going that route.


## Configuration

The following table lists the configurable parameters of the kubevious chart and their default values.

| Value                               | Description                                                  | Default                                      |
| ----------------------------------- | ------------------------------------------------------------ | -------------------------------------------- |
| nameOverride                        | Overrides the *app.kubernetes.io/name* label value           |                                              |
| fullnameOverride                    | Overrides name of the app                                    |                                              |
| cluster.domain                      | Overrides the default Kubernetes cluster domain name.        | cluster.local                                |
| ingress.enabled                     | Whether to expose Kubevious using Ingress gateway.           | false                                        |
| ingress.annotations                 | Dictionary of Ingress annodations.                           | `{kubernetes.io/ingress.allow-http: "true"}` |
| ingress.hosts                       | Array of hosts and paths for ingress                         | `[{host: "", paths: [ "" ] }`]               |
| ingress.tls                         | Array of ingress tls configurations. Fields are *hosts* array and *secretName* |                                              |
| kubevious.podAnnotations            | Kubevious backend pod annotations                            |                                              |
| kubevious.image.pullPolicy          | Kubevious backend PodSpec pullPolicy                         | IfNotPresent                                 |
| kubevious.image.imagePullSecrets    | Kubevious backend PodSpec imagePullSecrets                   |                                              |
| kubevious.service.type              | Kubevious backend type of service                            | ClusterIP                                    |
| kubevious.service.port              | Kubevious backend port of service                            | 4002                                         |
| kubevious.resources.requests.cpu    | Kubevious backend request CPU                                | 100m                                         |
| kubevious.resources.requests.memory | Kubevious backend request Memory                             | 200Mi                                        |
| kubevious.resources.limits.cpu      | Kubevious backend limit CPU                                  |                                              |
| kubevious.resources.limits.memory   | Kubevious backend limit Memory                               |                                              |
| kubevious.podSecurityContext        | Kubevious backend PodSpec securityContext                    |                                              |
| kubevious.nodeSelector              | Kubevious backend PodSpec nodeSelector                       |                                              |
| kubevious.tolerations               | Kubevious backend PodSpec tolerations                        |                                              |
| kubevious.affinity                  | Kubevious backend PodSpec affinity                           |                                              |
| parser.podAnnotations               | Kubevious parser pod annotations                            |                                              |
| parser.image.pullPolicy          | Kubevious parser PodSpec pullPolicy                         | IfNotPresent                                 |
| parser.image.imagePullSecrets    | Kubevious parser PodSpec imagePullSecrets                   |                                              |
| parser.service.type              | Kubevious parser type of service                            | ClusterIP                                    |
| parser.service.port              | Kubevious parser port of service                            | 4002                                         |
| parser.resources.requests.cpu    | Kubevious parser request CPU                                | 100m                                         |
| parser.resources.requests.memory | Kubevious parser request Memory                             | 200Mi                                        |
| parser.resources.limits.cpu      | Kubevious parser limit CPU                                  |                                              |
| parser.resources.limits.memory   | Kubevious parser limit Memory                               |                                              |
| parser.podSecurityContext        | Kubevious parser PodSpec securityContext                    |                                              |
| parser.nodeSelector              | Kubevious parser PodSpec nodeSelector                       |                                              |
| parser.tolerations               | Kubevious parser PodSpec tolerations                        |                                              |
| parser.affinity                  | Kubevious parser PodSpec affinity                           |                                              |
| parser.serviceAccount.create | Indicates wheter a service account should be created for Kubevious parser | true |
| parser.serviceAccount.annotations | Annotations to add to Kubevious parser service account |                                              |
| parser.serviceAccount.name | The name of the service account to use. If not and create is tru, a name is generated |                                              |
|                                     |                                                              |                                              |
| ui.podAnnotations            | Kubevious ui pod annotations                            |                                              |
| ui.image.pullPolicy          | Kubevious ui PodSpec pullPolicy                         | IfNotPresent                                 |
| ui.image.imagePullSecrets    | Kubevious ui PodSpec imagePullSecrets                   |                                              |
| ui.service.type              | Kubevious ui type of service                            | ClusterIP                                    |
| ui.service.port              | Kubevious ui port of service                            | 80                                        |
| ui.resources.requests.cpu    | Kubevious ui request CPU                                | 25m                                        |
| ui.resources.requests.memory | Kubevious ui request Memory                             | 50Mi                                        |
| ui.resources.limits.cpu      | Kubevious ui limit CPU                                  |                                              |
| ui.resources.limits.memory   | Kubevious ui limit Memory                               |                                              |
| ui.podSecurityContext        | Kubevious ui PodSpec securityContext                    |                                              |
| ui.nodeSelector              | Kubevious ui PodSpec nodeSelector                       |                                              |
| ui.tolerations               | Kubevious ui PodSpec tolerations                        |                                              |
| ui.affinity                  | Kubevious ui PodSpec affinity                           |                                              |
| mysql.image.pullPolicy          | Kubevious mysql PodSpec pullPolicy                         | IfNotPresent                                 |
| mysql.db_name | MySQL database name | kubevious |
| mysql.db_user | MySQL database user | kubevious |
| mysql.db_password | MySQL database password | autogenerated if not specified |
| mysql.root_password | MySQL root user password | autogenerated if not specified |
| mysql.persistence.enabled | Allows disabling of persistence | true |
| mysql.persistence.accessMode | MySQL persistent volume access mode | ReadWriteOnce |
| mysql.persistence.size | MySQL persistent volume size | 20Gi |
| mysql.persistence.storageClass | MySQL persistent volume storage class name |  |
| mysql.image.imagePullSecrets    | Kubevious mysql PodSpec imagePullSecrets                   |                                              |
| mysql.service.type              | Kubevious mysql type of service                            | ClusterIP                                    |
| mysql.service.port              | Kubevious mysql port of service                            | 3306                                    |
| mysql.resources.requests.cpu    | Kubevious mysql request CPU                                | 250m                                       |
| mysql.resources.requests.memory | Kubevious mysql request Memory                             | 1000Mi                                      |
| mysql.resources.limits.cpu      | Kubevious mysql limit CPU                                  |                                              |
| mysql.resources.limits.memory   | Kubevious mysql limit Memory                               |                                              |
| mysql.podAnnotations            | Kubevious mysql pod annotations                            |                                              |
| mysql.podSecurityContext        | Kubevious mysql PodSpec securityContext                    |                                              |
| mysql.nodeSelector              | Kubevious mysql PodSpec nodeSelector                       |                                              |
| mysql.tolerations               | Kubevious mysql PodSpec tolerations                        |                                              |
| mysql.affinity                  | Kubevious mysql PodSpec affinity                           ||
| worldvious.opt_out_version_check | Disables version check. As a part of the version check, Kubevious deployments are added to the leaderboard at https://worldvious.io. Reporting is anonymized to the nearest city/zip. No IP address is stored or logged. We calculate the SHA256 hash of the IP address and use it as a key. As a part of this request, we also added news notification and a feedback request mechanism. | false |
| worldvious.opt_out_error_report | Disables automatic exception and error reporting.            | false |
| worldvious.opt_out_counters_report | Disables periodic reporting of cluster metrics, such as: number of nodes, pods, ingresses, configmaps, etc. The number of pods and nodes would appear on the https://worldvious.io leaderboard. Those are the same counters you would see in the console log of kubevious and parser pods. | false |
| worldvious.opt_out_metrics_report | Disables periodic reporting of internal time metrics. In conjunction with counters reporting, this would help identify internal bottlenecks and improve overall performance.  Those are the same metrics you would see in the console log of kubevious and parser pods. | false |
| worldvious.opt_out_all | Opt out from all of the above reportings. | false |

