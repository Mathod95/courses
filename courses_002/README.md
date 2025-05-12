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
or
argocd-autopilot repo bootstrap --recover
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

❯ argocd cluster add kind-production --project production
WARNING: This will create a service account `argocd-manager` on the cluster referenced by context `kind-production` with full cluster level privileges. Do you want to continue [y/N]? y
{"level":"info","msg":"ServiceAccount \"argocd-manager\" created in namespace \"kube-system\"","time":"2025-05-11T17:49:02+02:00"}
{"level":"info","msg":"ClusterRole \"argocd-manager-role\" created","time":"2025-05-11T17:49:02+02:00"}
{"level":"info","msg":"ClusterRoleBinding \"argocd-manager-role-binding\" created","time":"2025-05-11T17:49:02+02:00"}
{"level":"info","msg":"Created bearer token secret for ServiceAccount \"argocd-manager\"","time":"2025-05-11T17:49:02+02:00"}
Cluster 'https://127.0.0.1:44821' added

❯ argocd cluster add kind-staging --project staging
WARNING: This will create a service account `argocd-manager` on the cluster referenced by context `kind-staging` with full cluster level privileges. Do you want to continue [y/N]? y
{"level":"info","msg":"ServiceAccount \"argocd-manager\" created in namespace \"kube-system\"","time":"2025-05-11T17:49:10+02:00"}
{"level":"info","msg":"ClusterRole \"argocd-manager-role\" created","time":"2025-05-11T17:49:10+02:00"}
{"level":"info","msg":"ClusterRoleBinding \"argocd-manager-role-binding\" created","time":"2025-05-11T17:49:10+02:00"}
{"level":"info","msg":"Created bearer token secret for ServiceAccount \"argocd-manager\"","time":"2025-05-11T17:49:10+02:00"}
Cluster 'https://127.0.0.1:40971' added
```

```shell
argocd-autopilot project create production --dest-server 
argocd-autopilot project create staging --dest-server 
argocd-autopilot project create management --dest-server in-cluster --project management --yes 
```

```
argocd-autopilot app create hello-world --app github.com/Mathod95/courses/courses_002/apps/examples -p production --wait-timeout 2m
argocd-autopilot app create hello-world --app github.com/Mathod95/courses/courses_002/apps/examples -p staging --wait-timeout 2m

argocd-autopilot app create uptime-kuma --app github.com/Mathod95/courses/courses_002/apps/uptimekuma -p production --wait-timeout 2m
argocd-autopilot app create uptime-kuma --app github.com/Mathod95/courses/courses_002/apps/uptimekuma -p staging --wait-timeout 2m

argocd-autopilot app create uptime-kuma --app github.com/Mathod95/courses/courses_002/apps/uptimekuma --project staging --namespace uptimekuma --wait-timeout 2m
```


❯ argocd cluster list
aSERVER                          NAME             VERSION  STATUS      MESSAGE  PROJECT
https://192.168.1.23:6445       kind-staging     1.32     Successful           staging
https://192.168.1.23:6444       kind-production  1.32     Successful           production
https://kubernetes.default.svc  in-cluster       1.32     Successful

❯ argocd proj list
NAME        DESCRIPTION         DESTINATIONS  SOURCES  CLUSTER-RESOURCE-WHITELIST  NAMESPACE-RESOURCE-BLACKLIST  SIGNATURE-KEYS  ORPHANED-RESOURCES  DESTINATION-SERVICE-ACCOUNTS
default                         *,*           *        */*                         <none>                        <none>          disabled            <none>
production  production project  *,*           *        */*                         <none>                        <none>          disabled            <none>
staging     staging project     *,*           *        */*                         <none>                        <none>          disabled            <none>

❯ argocd cluster list
SERVER                          NAME             VERSION  STATUS      MESSAGE  PROJECT
https://192.168.1.23:6445       kind-staging     1.32     Successful           staging
https://192.168.1.23:6444       kind-production  1.32     Successful           production
https://kubernetes.default.svc  in-cluster       1.32     Successful

❯ argocd proj list
NAME        DESCRIPTION         DESTINATIONS                 SOURCES  CLUSTER-RESOURCE-WHITELIST  NAMESPACE-RESOURCE-BLACKLIST  SIGNATURE-KEYS  ORPHANED-RESOURCES  DESTINATION-SERVICE-ACCOUNTS
default                         *,*                          *        */*                         <none>                        <none>          disabled            <none>
production  production project  https://192.168.1.23:6444,*  *        */*                         <none>                        <none>          disabled            <none>
staging     staging project     https://192.168.1.23:6445,*  *        */*                         <none>                        <none>          disabled            <none>

❯ argocd-autopilot app create hello-world --app github.com/Mathod95/courses/courses_002/apps/examples -p production --wait-timeout 2m
INFO cloning git repository: https://github.com/Mathod95/courses.git
Enumerating objects: 41, done.
Counting objects: 100% (41/41), done.
Compressing objects: 100% (33/33), done.
Total 41 (delta 6), reused 34 (delta 4), pack-reused 0 (from 0)
INFO using revision: "", installation path: "/courses_002/autopilot"
INFO cloning repo: 'https://github.com/Mathod95/courses.git', to infer app type from path 'courses_002/apps/examples'
Enumerating objects: 41, done.
Counting objects: 100% (41/41), done.
Compressing objects: 100% (33/33), done.
Total 41 (delta 6), reused 34 (delta 4), pack-reused 0 (from 0)
INFO inferred application type: kustomize
FATAL cluster 'https://192.168.1.23:6444' is not configured yet, you need to create a project that uses this cluster first