```
$ docker pull fireflyiii/core:latest
$ docker run -d --name firefly \
-v firefly_iii_upload:/var/www/html/storage/upload \
-p 8010:8080 \
-e APP_KEY=app_key \
-e DB_HOST=192.168.2.20 \
-e DB_PORT=33306 \
-e DB_CONNECTION=mysql \
-e DB_DATABASE=fireflyDB \
-e DB_USERNAME=root \
-e DB_PASSWORD=password \
fireflyiii/core:latest
```

> <https://github.com/firefly-iii/firefly-iii>
