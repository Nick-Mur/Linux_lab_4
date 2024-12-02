# Отчёт по лабораторной работе

## Задание

1. **Запустить в контейнере приложение `aafire`. Обратите внимание, что оно бесконечное и контейнер не будет автоматически отключаться. Приложить скриншот в процессе работы контейнера.**

2. **Настроить сеть между двумя контейнерами, как в предыдущей работе с двумя виртуальными машинами.**

3. **Установить утилиту `ping` в образ, помимо `aafire`, для проверки сети между контейнерами.**

4. **Запустить два контейнера с `aafire` и оставить их в работающем состоянии.**

5. **Создать сеть `myNetwork` и подключить к ней оба контейнера.**

6. **Проверить соединение между контейнерами с помощью `ping` и приложить скриншот.**

7. **Очистить Docker после работы, отключить/включить его.**

---

## Шаг 1: Создание Dockerfile

Создадим файл `Dockerfile` со следующим содержимым:

```dockerfile
FROM ubuntu:latest

# Установка необходимых пакетов
RUN apt-get update && \
    apt-get install -y libaa-bin iputils-ping && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Запуск приложения aafire
CMD ["aafire"]
```

**Пояснения:**

- Используем официальный образ Ubuntu.
- Устанавливаем `aafire` (пакет `libaa-bin`) и `ping` (пакет `iputils-ping`).
- Очищаем кэш пакетов для уменьшения размера образа.
- Указываем команду по умолчанию для запуска `aafire`.

---

## Шаг 2: Сборка Docker-образа

В терминале, находясь в директории с `Dockerfile`, выполним команду:

```bash
docker build -t aafire_image:latest .
```

**Пояснения:**

- `-t aafire_image:latest` задаёт имя и тег образа.
- Точка `.` указывает на текущую директорию как контекст сборки.

---

## Шаг 3: Запуск двух контейнеров с `aafire`

Запустим два контейнера и назовём их `mycontainer1` и `mycontainer2`.

### Запуск первого контейнера

```bash
docker run -d --name mycontainer1 aafire_image:latest
```

### Запуск второго контейнера

```bash
docker run -d --name mycontainer2 aafire_image:latest
```

**Пояснения:**

- `-d` запускает контейнеры в фоновом режиме (detached mode).
- Контейнеры будут выполнять `aafire` бесконечно.

---

## Шаг 4: Создание сети `myNetwork`

Создадим пользовательскую сеть Docker:

```bash
docker network create myNetwork
```

---

## Шаг 5: Подключение контейнеров к сети `myNetwork`

Подключим оба контейнера к созданной сети.

### Подключение первого контейнера

```bash
docker network connect myNetwork mycontainer1
```

### Подключение второго контейнера

```bash
docker network connect myNetwork mycontainer2
```

---

## Шаг 6: Проверка настроек сети

Проверим настройки сети `myNetwork`:

```bash
docker network inspect myNetwork
```

**Пример вывода:**

```json
[
    {
        "Name": "myNetwork",
        "Id": "...",
        "Created": "...",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": { ... },
        "Containers": {
            "mycontainer1": { ... },
            "mycontainer2": { ... }
        },
        "Options": {},
        "Labels": {}
    }
]
```

---

## Шаг 7: Проверка соединения между контейнерами с помощью `ping`

### Выполнение `ping` из первого контейнера во второй

```bash
docker exec -it mycontainer1 ping mycontainer2
```

**Пояснения:**

- `docker exec` позволяет выполнять команды внутри запущенного контейнера.
- `-it` делает сессию интерактивной.
- Мы пингуем `mycontainer2` по имени контейнера.

**Ожидаемый результат:**

```
PING mycontainer2 (172.18.0.3): 56 data bytes
64 bytes from 172.18.0.3: icmp_seq=1 ttl=64 time=0.123 ms
64 bytes from 172.18.0.3: icmp_seq=2 ttl=64 time=0.111 ms
...
```

![image](https://github.com/user-attachments/assets/2cb2cdd2-9f96-4d6a-b2d7-d1e281b8bdff)


---

## Шаг 8: Очистка Docker после работы

Чтобы очистить Docker от созданных контейнеров, образов и сети, выполните следующие команды.

### Остановка и удаление контейнеров

```bash
docker stop mycontainer1 mycontainer2
docker rm mycontainer1 mycontainer2
```

### Удаление образа

```bash
docker rmi aafire_image:latest
```

### Удаление сети

```bash
docker network rm myNetwork
```

### Удаление неиспользуемых объектов Docker

```bash
docker system prune -a --volumes
```

**Пояснения:**

- `docker system prune -a --volumes` удаляет все неиспользуемые объекты: остановленные контейнеры, неиспользуемые образы, тома и сети.
- **Внимание:** Убедитесь, что вам не нужны эти данные перед выполнением команды.

---

## Шаг 9: Отключение и включение Docker

### Отключение Docker

Чтобы остановить сервис Docker, выполните:

```bash
sudo systemctl stop docker
```

### Включение Docker

Чтобы запустить сервис Docker, выполните:

```bash
sudo systemctl start docker
```

### Проверка статуса Docker

```bash
sudo systemctl status docker
```

---

## Заключение

В ходе выполнения данной лабораторной работы мы:

- Создали Dockerfile для образа с `aafire` и `ping`.
- Собрали Docker-образ и запустили два контейнера с `aafire`.
- Создали пользовательскую сеть `myNetwork` и подключили к ней контейнеры.
- Проверили соединение между контейнерами с помощью `ping`.
- Очистили Docker после работы.
- Научились отключать и включать сервис Docker.

---
