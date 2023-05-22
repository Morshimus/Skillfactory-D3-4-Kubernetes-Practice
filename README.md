# Skillfactory-D3-4-Kubernetes-Practice

## Задание

* [x] - :one: **Создать Deployment со свойствами ниже:**
     - *образ  ~~nginx:1.21.1-alpine~~ morsh92/nginx_auth:1.21.1-alpine;*
     - *имя  nginx-sf;*
     - *количество реплик  3.*

> Немного улучшил исходный докер образ изменив entrypoint.


```Dockerfile
      FROM nginx:1.21.1-alpine

      ENV SECRET_USERNAME=''

      ENV SECRET_PASSWD=''

      COPY ./docker-entrypoint.sh /docker-entrypoint.sh

      ENTRYPOINT [ "/docker-entrypoint.sh"]

      EXPOSE 80

      STOPSIGNAL SIGQUIT

      CMD ["nginx", "-g", "daemon off;"]
```

```sh      
      #!/bin/sh
      # vim:sw=4:ts=4:et

       set -e

      if [ -z "${NGINX_ENTRYPOINT_QUIET_LOGS:-}" ]; then
         exec 3>&1
      else
         exec 3>/dev/null
        fi

      if [ "$1" = "nginx" -o "$1" = "nginx-debug" ]; then
      if /usr/bin/find "/docker-entrypoint.d/" -mindepth 1 -maxdepth 1 -type f -print -quit 2>/dev/null | read v; then
         echo >&3 "$0: /docker-entrypoint.d/ is not empty, will attempt to perform configuration"

         echo >&3 "$0: Looking for shell scripts in /docker-entrypoint.d/"
         find "/docker-entrypoint.d/" -follow -type f -print | sort -V | while read -r f; do
             case "$f" in
                 *.sh)
                     if [ -x "$f" ]; then
                         echo >&3 "$0: Launching $f";
                         "$f"
                     else
                         # warn on shell scripts without exec bit
                         echo >&3 "$0: Ignoring $f, not executable";
                     fi
                     ;;
             *) echo >&3 "$0: Ignoring $f";;
             esac
         done

         echo >&3 "$0: Configuration complete; ready for start up"
       else
         echo >&3 "$0: No files found in /docker-entrypoint.d/, skipping configuration"
       fi
      fi

      echo "$SECRET_USERNAME:$SECRET_PASSWD" > /etc/nginx/.htpasswd

      exec "$@"
```

* [x] - :two: **Создать конфигурационный файл для нашего приложения и поместить его в наш Pod со следующими свойствами:**
     - *путь до файла в Pod’е — /etc/nginx/nginx.conf;*
     - *содержимое файла:*
      
```conf
      user nginx;
      worker_processes  1;
      events {
          worker_connections  10240;
      }
      http {
        server {
          listen       80;
          server_name  localhost;
          location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
        }
       }
      }
```

* [x] - :three: **Создать service для того, чтобы можно было обращаться к любому из Pod’ов по единому имени:**
     - *имя сервиса sf-webserver;*
     - *внешний порт — 80.*

* [x] - :four: **Создать секрет со следующими данными:**
     - *имя секрета — auth_basic;*
     - *~~ключ объекта в секрете — ~~user1~~;~~ ключ username равный значению username1*
     - *~~значение объекта в секрете user1 — password1;~~ password1 в base64 который в openssl -apr1 для .htpasswd*

* [x] - :five: **Подключить в наш контейнер эти секреты.Обновить конфиг nginx таким образом, чтобы подключенные секреты использовались для авторизации для доступа к странице по умолчанию в nginx.**

> Добавляем в nginx 2 строчки auth_basic и auth_basic_user_file
    

```conf
NAME                        READY   STATUS    RESTARTS   AGE   IP            NODE             NOMINATED NODE   READINESS GATES
nginx-sf-5c489dc78d-fpz25   1/1     Running   0          34m   10.244.3.22   sf-d-2-4-1-m04   <none>           <none>
nginx-sf-5c489dc78d-hgtk2   1/1     Running   0          34m   10.244.4.20   sf-d-2-4-1-m05   <none>           <none>
nginx-sf-5c489dc78d-sp62d   1/1     Running   0          34m   10.244.1.21   sf-d-2-4-1-m02   <none>           <none>
```

```conf
sf-webserver   ClusterIP   10.110.70.108   <none>        80/TCP    36m
```

> Запускаем отдельный под с alpine для проверки команды curl http://username1:password1@sf-webserver

```conf
kubectl run -ti  --rm debug  --image alpine  -- ash
If you don't see a command prompt, try pressing enter.
/ # apk add curl
fetch https://dl-cdn.alpinelinux.org/alpine/v3.18/main/x86_64/APKINDEX.tar.gz
fetch https://dl-cdn.alpinelinux.org/alpine/v3.18/community/x86_64/APKINDEX.tar.gz
(1/7) Installing ca-certificates (20230506-r0)
(2/7) Installing brotli-libs (1.0.9-r14)
(3/7) Installing libunistring (1.1-r1)
(4/7) Installing libidn2 (2.3.4-r1)
(5/7) Installing nghttp2-libs (1.53.0-r0)
(6/7) Installing libcurl (8.1.0-r2)
(7/7) Installing curl (8.1.0-r2)
Executing busybox-1.36.0-r9.trigger
Executing ca-certificates-20230506-r0.trigger
OK: 12 MiB in 22 packages
/ # curl http://username1:password1@sf-webserver
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```
