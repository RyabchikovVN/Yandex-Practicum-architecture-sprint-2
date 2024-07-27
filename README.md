# sharding-repl-cache

## Как запустить

Перейти в директорию sharding-repl-cache

Запускаем mongodb и приложение

```shell
docker compose up -d
```

Конфигурируем шарды и реплики и заполняем mongodb данными

```shell
./scripts/sharding-init.sh
```

## Как проверить

Проверка JSON ответа сервера:
Откройте в браузере http://localhost:8080

Проверка shard-1 на количество документов

```shell
./scripts/shard-1-check-count.sh
```

Проверка shard-2 на количество документов

```shell
./scripts/shard-2-check-count.sh
```


## Ссылка на диаграмму
```shell
https://drive.google.com/file/d/1kO2wXX8BBlgbeNrHXNbKailQfWbSqqde/view?usp=sharing
```
