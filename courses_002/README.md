```shell
kind create cluster --config management.yaml
kind create cluster --config staging.yaml
kind create cluster --config production.yaml
```

```shell
kctx kind-management
```

```
export GIT_TOKEN=<TOKEN>
export GIT_REPO=https://github.com/Mathod95/courses/courses_002/autopilot
```

```shell
argocd-autopilot repo bootstrap
```

```shell
INFO argocd initialized. password: <PASSWORD>
INFO run:

    kubectl port-forward -n argocd svc/argocd-server 8080:80
```

```shell
kns argocd
```

```shell
❯ argocd login --core
Context 'kubernetes' updated

❯ argocd cluster list
SERVER                          NAME        VERSION  STATUS      MESSAGE  PROJECT
https://kubernetes.default.svc  in-cluster  1.32     Successful

❯ argocd cluster add
CURRENT  NAME             CLUSTER          SERVER
*        kind-management  kind-management  https://127.0.0.1:41469
         kind-production  kind-production  https://127.0.0.1:44821
         kind-staging     kind-staging     https://127.0.0.1:40971

❯ argocd cluster add kind-production
WARNING: This will create a service account `argocd-manager` on the cluster referenced by context `kind-production` with full cluster level privileges. Do you want to continue [y/N]? y
{"level":"info","msg":"ServiceAccount \"argocd-manager\" created in namespace \"kube-system\"","time":"2025-05-11T17:49:02+02:00"}
{"level":"info","msg":"ClusterRole \"argocd-manager-role\" created","time":"2025-05-11T17:49:02+02:00"}
{"level":"info","msg":"ClusterRoleBinding \"argocd-manager-role-binding\" created","time":"2025-05-11T17:49:02+02:00"}
{"level":"info","msg":"Created bearer token secret for ServiceAccount \"argocd-manager\"","time":"2025-05-11T17:49:02+02:00"}
Cluster 'https://127.0.0.1:44821' added

❯ argocd cluster add kind-staging
WARNING: This will create a service account `argocd-manager` on the cluster referenced by context `kind-staging` with full cluster level privileges. Do you want to continue [y/N]? y
{"level":"info","msg":"ServiceAccount \"argocd-manager\" created in namespace \"kube-system\"","time":"2025-05-11T17:49:10+02:00"}
{"level":"info","msg":"ClusterRole \"argocd-manager-role\" created","time":"2025-05-11T17:49:10+02:00"}
{"level":"info","msg":"ClusterRoleBinding \"argocd-manager-role-binding\" created","time":"2025-05-11T17:49:10+02:00"}
{"level":"info","msg":"Created bearer token secret for ServiceAccount \"argocd-manager\"","time":"2025-05-11T17:49:10+02:00"}
Cluster 'https://127.0.0.1:40971' added
```

```shell
argocd-autopilot project create production
argocd-autopilot project create staging
argocd-autopilot project create management
```

```
argocd-autopilot app create <hello-world> --app github.com/Mathod95/courses/courses_002/apps/examples -p <PROJECT> --wait-timeout 2m
```