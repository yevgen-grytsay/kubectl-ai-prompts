# Приклади запитів і відповідей для `kubectl-ai`

| NAME | PROMPT | DESCRIPTION | EXAMPLE |
| --- | --- | --- | --- |
| Nginx | deployment: latest nginx, open both web ports |  | [app.yaml](yaml/app.yaml) |
| Nginx + PHP-FPM | deployment of two containers: php-fpm version 8.3, nginx latest version. open ports for all containers. |  | [app-multicontainer.yaml](yaml/app-multicontainer.yaml) |
| Nginx + liveness probe | deployment latest nginx. add liveness probe |  | [app-livenessProbe.yaml](yaml/app-livenessProbe.yaml) |
| Nginx + readiness probe | deployment: latest nginx, open both web ports, add readiness check |  | [app-readinessProbe.yaml](yaml/app-readinessProbe.yaml) |
| Nginx 2 replicas + volume mounts | deployment: latest nginx, two replicas, open both ports, add persistent volume claim and mount it to nginx html directory |  | [app-volumeMounts.yaml](yaml/app-volumeMounts.yaml) |
| Cronjob | cronjob: every minute print current date and time to stdout |  | [app-cronjob.yaml](yaml/app-cronjob.yaml) |
| Job | job example: append date and time to a file in mounted volume; mount pvc volume |  | [app-job.yaml](yaml/app-job.yaml) |
| Resource Management | deployment: latest nginx. add resource limit |  | [app-resources.yaml](yaml/app-resources.yaml) |
| MySQL + env variables from a secret | deployment: latest mysql; create all necessary env variables one by one, take values from a secret |  | [app-secret-env.yaml](yaml/app-secret-env.yaml) |

## Setup
```sh
k3d cluster create ai-prompts-cluster \
    --registry-config /home/yevhen/.k3d/registries.yaml \
    --volume /home/yevhen/.k3d/ai-prompts-cluster-volume:/tmp/k3dvol \
    -p "8088:80"

kubectl create ns kube-ai

kubectl config set-context --current --namespace=kube-ai
```

## Liveness Probe for Nginx
```sh
$ kubectl logs -f --tail=3 pod/nginx-deployment-79db6c8d55-t7phb
10.42.0.1 - - [16/Apr/2024:18:36:25 +0000] "GET / HTTP/1.1" 200 615 "-" "kube-probe/1.28" "-"
10.42.0.1 - - [16/Apr/2024:18:36:35 +0000] "GET / HTTP/1.1" 200 615 "-" "kube-probe/1.28" "-"
10.42.0.1 - - [16/Apr/2024:18:36:45 +0000] "GET / HTTP/1.1" 200 615 "-" "kube-probe/1.28" "-"
```


## Volume mounts
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nginx-pvc-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/tmp/k3d-volume"

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nginx-pvc
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

```sh
$ kubectl apply -f _app-persistentVolume.yaml 
persistentvolume/nginx-pvc-volume created

$ kubectl get pv
NAME               CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
nginx-pvc-volume   1Gi        RWO            Retain           Available           manual                  6s

```

```sh
$ kubectl apply -f _app-persistentVolumeClaim.yaml

$ kubectl get pvc
NAME        STATUS   VOLUME             CAPACITY   ACCESS MODES   STORAGECLASS   AGE
nginx-pvc   Bound    nginx-pvc-volume   1Gi        RWO            manual         12s
```

## Check job results

```sh
$ kubectl exec pod/nginx-deployment-57f978445f-x57qv -- /bin/cat /usr/share/nginx/html/date-time.html
```

## Create Secret for MySQL

```sh
kubectl create secret generic mysql-secret \
    --from-literal=mysql-root-password='root' \
    --from-literal=mysql-user=u-kube \
    --from-literal=mysql-database=db-kube \
    --from-literal=mysql-password='qwe123'

kubectl apply -f app-secret-env.yaml

kubectl exec --stdin --tty pod/mysql-7f85f68549-sqfbr -- /usr/bin/mysql -u u-kube -D db-kube -p
```

## Resources
- [Managing Secrets using kubectl | Use raw data](https://kubernetes.io/docs/tasks/configmap-secret/managing-secret-using-kubectl/#use-raw-data)
- [Resource Management for Pods and Containers](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)
- [Running Automated Tasks with a CronJob](https://kubernetes.io/docs/tasks/job/automated-tasks-with-cron-jobs/)
