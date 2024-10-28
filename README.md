## Как запустить

1. Запускаем инициализацию проекта
```shell
docker compose up -d
```

2. Подключитесь к серверу конфигурации и сделайте инициализацию

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

3. Инициализируйте шарды и реплики

```shell
docker exec -it shard1-1 mongosh --port 27018

> rs.initiate(
    {
      _id : "shard1-1",
      members: [
        {_id: 0, host: "shard1-1:27018"},
        {_id: 1, host: "shard1-2:27019"},
        {_id: 2, host: "shard1-3:27020"}
      ]
    }
);
> exit();

docker exec -it shard2-1 mongosh --port 27019

> rs.initiate(
    {
      _id : "shard2-1",
      members: [
        {_id: 0, host: "shard2-1:27021"},
        {_id: 1, host: "shard2-2:27022"},
        {_id: 2, host: "shard2-3:27023"}
      ]
    }
  );
> exit();
```

4. Инцициализируйте роутер и наполните его тестовыми данными

```shell
docker exec -it mongos_router mongosh --port 27020

> sh.addShard( "shard1-1/shard1-1:27018");
> sh.addShard( "shard2-1/shard2-1:27021");

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

3. Проверка количества документов в шарде 1 (результат 492)

```shell
 docker exec -it shard1-1 mongosh --port 27018
 > use somedb;
 > db.helloDoc.countDocuments();
 > exit(); 
```

4. Проверка количества документов в шарде 2 (результат 508)

```shell
 docker exec -it shard2-1 mongosh --port 27021
 > use somedb;
 > db.helloDoc.countDocuments();
 > exit(); 
```

## Схема проекта draw.io - https://drive.google.com/file/d/11sACr9oyy8sLJCBHSoKGx0V8Egpksw9U/view?usp=sharing
Даниил, добрый день!
Спасибо за потраченное время на ревью и обратную связь!
Что сделано, с учетом ваших замечаний:
1. Скорректирована схема "Шаг-4"
2. Скорректирована схема "Шаг-5"


