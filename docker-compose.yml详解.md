docker-compose.yml详解



### `build`

**指定 Dockerfile 所在文件夹的路径**（可以是绝对路径，或者相对 docker-compose.yml 文件的路径）。在使用 up 启动之时执行构建任务，这个构建标签就是 build

### `depends_on`

解决容器的依赖、启动先后的问题。以下例子中会先启动 `redis` `db` 再启动 `web`

```yaml
version: '3'
services:
  web:
    build: .
    depends_on:
      - db
      - redis
  redis:
    image: redis
  db:
    image: postgres
```

## networks

加入指定网络，格式如下：

```
services:
  some-service:
    networks:
     - some-network
     - other-networ
```

**entrypoint**

```
在 Dockerfile 中有一个指令叫做 ENTRYPOINT 指令，用于指定接入点。在 docker-compose.yml 中可以定义接入点，覆盖 Dockerfile 中的定义：

entrypoint: /code/entrypoint.sh
entrypoint 也可以是一个列表，方法类似于 dockerfile

entrypoint:
- php
- -d
- zend_extension=/usr/local/lib/php/extensions/no-debug-non-zts-20100525/xdebug.so
- -d
- memory_limit=-1
- vendor/bin/phpunit
```

