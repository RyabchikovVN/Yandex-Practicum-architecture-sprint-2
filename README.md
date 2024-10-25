## Как запустить

Шаги запуска проекта:
1. Если проект запускается на локальной машине, то определяем папку проекта "mongo-sharding"
{YOUR FULL FOLDER PATH} - полный путь к папке (Например: "C:\Users\....\mongo-sharding")

```shell
Set-Location -Path {YOUR FULL FOLDER PATH}\mongo-sharding -PassThru
```

2. Запускаем инициализацию проекта
```shell
docker compose up -d
```
  
3. Проверяем успешную инициализацию проекта
```shell
docker ps  -a
```

4. Подключитесь к серверу конфигурации и сделайте инициализацию

```shell
docker exec -it configSrv mongosh --port 27017

> rs.initiate(
  {
    _id : "config_server",
       configsvr: true,
    members: [
      { _id : 0, host : "configSrv:27017" }
    ]
  }
);
> exit(); 
```

5. Инициализируйте шарды

```shell
docker exec -it shard1 mongosh --port 27018

> rs.initiate(
    {
      _id : "shard1",
      members: [
        { _id : 0, host : "shard1:27018" },
      ]
    }
);
> exit();

docker exec -it shard2 mongosh --port 27019

> rs.initiate(
    {
      _id : "shard2",
      members: [
        { _id : 1, host : "shard2:27019" }
      ]
    }
  );
> exit();
```

6. Инцициализируйте роутер и наполните его тестовыми данными

```shell
docker exec -it mongos_router mongosh --port 27020

> sh.addShard( "shard1/shard1:27018");
> sh.addShard( "shard2/shard2:27019");

> sh.enableSharding("somedb");
> sh.shardCollection("somedb.helloDoc", { "name" : "hashed" } )

> use somedb

> for(var i = 0; i < 1000; i++) db.helloDoc.insert({age:i, name:"ly"+i})

> db.helloDoc.countDocuments() 
> exit(); 
```

## Как проверить
1. Swagger API должен открываться по URL http://localhost:8080/docs
2. Проверка общего числа документов в mongoDB (результат 1000)

```shell
docker exec -it mongos_router mongosh --port 27020
> use somedb
> db.helloDoc.countDocuments() 
> exit(); 
```

4. Проверка количества документов в шарде 1 (результат 492)

```shell
 docker exec -it shard1 mongosh --port 27018
 > use somedb;
 > db.helloDoc.countDocuments();
 > exit(); 
```
5. Проверка количества документов в шарде 2 (результат 508)

```shell
 docker exec -it shard2 mongosh --port 27019
 > use somedb;
 > db.helloDoc.countDocuments();
 > exit(); 
```
