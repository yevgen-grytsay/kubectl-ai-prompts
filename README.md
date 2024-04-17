# Приклади запитів і відповідей для `kubectl-ai`

| NAME | PROMPT | DESCRIPTION | EXAMPLE |
| --- | --- | --- | --- |
| Nginx | deployment: latest nginx, open both web ports |  | [app.yaml](app.yaml) |
| Nginx + PHP-FPM | deployment of two containers: php-fpm version 8.3, nginx latest version. open ports for all containers. |  | [app-multicontainer.yaml](app-multicontainer.yaml) |
| Nginx + liveness probe | deployment latest nginx. add liveness probe |  | [app-livenessProbe.yaml](app-livenessProbe.yaml) |
|  |  |  |  |
|  |  |  |  |
|  |  |  |  |
|  |  |  |  |





## Liveness Probe for Nginx
```sh
$ kubectl logs -f --tail=3 pod/nginx-deployment-79db6c8d55-t7phb
10.42.0.1 - - [16/Apr/2024:18:36:25 +0000] "GET / HTTP/1.1" 200 615 "-" "kube-probe/1.28" "-"
10.42.0.1 - - [16/Apr/2024:18:36:35 +0000] "GET / HTTP/1.1" 200 615 "-" "kube-probe/1.28" "-"
10.42.0.1 - - [16/Apr/2024:18:36:45 +0000] "GET / HTTP/1.1" 200 615 "-" "kube-probe/1.28" "-"
```
