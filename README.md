# Skillfactory-D3-4-Kubernetes-Practice

## Задание

* [x] - :one: **Создать Deployment со свойствами ниже:**
      - *образ — ~~nginx:1.21.1-alpine~~ morsh92/nginx_auth:1.21.1-alpine;*
      - *имя — nginx-sf;*
      - *количество реплик — 3.*
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
    

