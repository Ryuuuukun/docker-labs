
### Среда

---

OS: Windows 10
Терминал: Windows PowerShell
Версия клиента: 29.8.0
Версия сервера: 29.8.0

В качестве базового образа был выбран `ngnix:alpine`.

### Задания

---

#### 1. Версии и теги

Определение версии клиента и демона Docker:

```shell
docker version # client: 29.8.0, server: 29.8.0 
```

Запуск выбранного образа в конкретном теге (`python:3.10.20-alpine`):

```shell
docker run --name select-tag -d python:3.10.20-alpine
```

![[image-01.png]]

(ссылки на лекции: [запуск в конкретном теге](https://github.com/tiranousor/os-modern-docker-course/blob/main/4-course/docker-and-containers/lectures/02_commands.md#10-docker-attach-%D0%B8--d-%D0%BD%D0%B0-%D0%BF%D1%80%D0%B8%D0%BC%D0%B5%D1%80%D0%B5-%D0%B0%D0%BD%D0%B0%D0%BB%D0%B8%D0%B7%D0%B0-%D1%82%D0%B5%D0%BA%D1%81%D1%82%D0%B0))

#### 2. Первый запуск сервиса (detached)

Запуск контейнера в фоновом режиме, сначала на порту `8080`, затем перезапуск на порту `5050`:

```shell
# Запускаем контейнер на порту 8080
docker run --name lab-web-shouryuuko -d -p 8080:80 nginx:alpine

# Удаляем старый контейнер + (-f) останавливаем, если был запуще
docker rm -f lab-web-shouryuuko

# Запускаем контейнер на другом порту 5000
docker run --name lab-web-shouryuuko -d -p 5000:80 nginx:alpine
```

Запуск на порту `8080`:

![[image-02.png]]

Перезапуск на порту `5000`:

![[image-03.png]]

(ссылки на лекции: [проброс портов](https://github.com/tiranousor/os-modern-docker-course/blob/main/4-course/docker-and-containers/lectures/03_docker_run.md#3-%D0%BF%D1%80%D0%BE%D0%B1%D1%80%D0%BE%D1%81-%D0%BF%D0%BE%D1%80%D1%82%D0%BE%D0%B2--p), [именование контейнеров](https://github.com/tiranousor/os-modern-docker-course/blob/main/4-course/docker-and-containers/lectures/03_docker_run.md#4-%D0%B8%D0%BC%D0%B5%D0%BD%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5-%D0%BA%D0%BE%D0%BD%D1%82%D0%B5%D0%B9%D0%BD%D0%B5%D1%80%D0%BE%D0%B2---name))

#### 3. Том (bind-mount): сайт/данные из папки хоста

```shell
# Создаем каталог для монтирования
mkdir ~/docker/site
echo "<h1>Hello, World!</h1>" > ~/docker/site/index.html

docker rm -f lab-web-shouryuuko

# Запускаем контейнер с монтированием каталога в режиме read-only
docker run --name lab-web-shouryuuko -d \
	-v ~/docker/site:/usr/share/nginx/html:ro \
	-p 5000:80 nginx:alpine

# Меняем содержимое контента и проверяем 
echo "<h1>!dlroW, olleH</h1>" > ~/docker/site/index.html
```

До изменения содержимого файла `index.html`:

![[image-04.png]]

После изменения содержимого:

![[image-05.png]]

(ссылки на лекции: [монтирование томов](https://github.com/tiranousor/os-modern-docker-course/blob/main/4-course/docker-and-containers/lectures/03_docker_run.md#5-%D1%82%D0%BE%D0%BC-%D0%B4%D0%BB%D1%8F-%D1%85%D1%80%D0%B0%D0%BD%D0%B5%D0%BD%D0%B8%D1%8F-%D0%B4%D0%B0%D0%BD%D0%BD%D1%8B%D1%85--v))

#### 4. Интерактив / exec

```shell
# Подключаемся к контейнеру с командной оболочкой Shell
docker exec -it lab-web-shouryuuko sh

# -------------- in terminal --------------
ls -l /usr/share/nginx/html
touch /usr/share/nginx/html/try_create
# -----------------------------------------

# Удаляем контейнер
docker rm -f lab-web-shouryuuko

# Перезапускаем контейнер с монтированием каталога в режиме 
# read-write (значение по умолчанию)
docker run --name lab-web-shouryuuko -d \
	-v ~/docker/site:/usr/share/nginx/html \
	-p 5000:80 nginx:alpine

# Подключаемся к контейнеру
docker exec -it lab-web-shouryuuko sh

# -------------- in terminal --------------
touch /usr/share/nginx/html/try_create
ls -l /usr/share/nginx/html
# -----------------------------------------

# Проверяем, что файл появился на хосте
ls -l ~/docker/site
```

Листинг каталога `/usr/share/nginx/html` и попытка создания файла при `read-only`:

![[image-07.png]]

Создание файла при `read-write` и проверка на хосте:

![[image-08.png]]

(ссылки на лекции: [подключение к контейнеру](https://github.com/tiranousor/os-modern-docker-course/blob/main/4-course/docker-and-containers/lectures/02_commands.md#%D1%88%D0%B0%D0%B3-3-%D0%B2%D1%8B%D0%BF%D0%BE%D0%BB%D0%BD%D0%B5%D0%BD%D0%B8%D0%B5-%D0%BA%D0%BE%D0%BC%D0%B0%D0%BD%D0%B4%D1%8B-%D0%B2-%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D0%B0%D1%8E%D1%89%D0%B5%D0%BC-%D0%BA%D0%BE%D0%BD%D1%82%D0%B5%D0%B9%D0%BD%D0%B5%D1%80%D0%B5-docker-exec))

#### 5. Логи и attach

```shell
curl http://localhost:5000/
curl http://localhost:5000/nope
curl http://localhost:5000/try_another_nope

docker logs --tail 10 lab-web-shouryuuko

docker attach lab-web-shouryuuko
# Для корректной отстыковки нажимаем (Ctrl + P) + (Ctrl + Q)
# (Ctrl + C) отправит сигнал SIGINT и остановит основной процесс контейнера
```

Фрагмент журнала контейнера:

![[image-09.png]]

(ссылки на лекции: [получение логов](https://github.com/tiranousor/os-modern-docker-course/blob/main/4-course/docker-and-containers/lectures/03_docker_run.md#8-%D0%BB%D0%BE%D0%B3%D0%B8-%D0%BA%D0%BE%D0%BD%D1%82%D0%B5%D0%B9%D0%BD%D0%B5%D1%80%D0%B0-docker-logs), [прикрепление и корректная отстыковка](https://github.com/tiranousor/os-modern-docker-course/blob/main/4-course/docker-and-containers/lectures/02_commands.md#10-docker-attach-%D0%B8--d-%D0%BD%D0%B0-%D0%BF%D1%80%D0%B8%D0%BC%D0%B5%D1%80%D0%B5-%D0%B0%D0%BD%D0%B0%D0%BB%D0%B8%D0%B7%D0%B0-%D1%82%D0%B5%D0%BA%D1%81%D1%82%D0%B0))

#### 6. Краткоживущий процесс

```shell
# Запускаем контейнер в оболочке Shell и указываем, что выполнить
docker run --name short-lived alpine sh -c 'cat << "EOF"
                ⢀⠤⡀
 ⢠⠊⣉⠒⠤⢀⡀       ⡐⢁⠴⢜⢄
 ⡎⢸ ⠉⠐⠢⢌⠑⢄    ⡸ ⡆  ⠣⠱⡀
 ⡇⢸     ⣀⠗  ⠉⠉⠁  ⠙⠢⠤⡀⢃⢱
 ⡇⠘⣄⢀⠔⠉               ⠈⠁⠘⡄
 ⢇    ⠁                   ⠘⡄
 ⢸       ⢀⣀⣀⡀       ⢀⣀⣀⡀  ⢣
 ⡸     ⢴⣾⡿⠿⠽⠇      ⠘⠛⠛⠛  ⠈⢄
⠰⡁          ⢠⠒⠢⡀⠈⠒⠊      ⡠⢄⡘
 ⠱⣀       ⢀⠜    ⠇     ⢀⠔⠁  ⡏
    ⠑⠤⢄⣀⠔⠁    ⡜     ⠊⠁  ⢀⠜
EOF'

docker ps -a

```

Контейнер работает до тех пор, пока выполняется его главный процесс, указанная в директиве `CMD` или `ENTRYPOINT`. Команда `echo`, после вывода текста, сразу завершает свою работу.

Запуск контейнера и статус контейнера после выполнения:

![[image-12.png]]

(ссылка на лекции: [запуск краткоживущего процесса](https://github.com/tiranousor/os-modern-docker-course/blob/main/4-course/docker-and-containers/lectures/03_docker_run.md#6-%D0%B0%D0%B2%D1%82%D0%BE%D0%BC%D0%B0%D1%82%D0%B8%D1%87%D0%B5%D1%81%D0%BA%D0%BE%D0%B5-%D1%83%D0%B4%D0%B0%D0%BB%D0%B5%D0%BD%D0%B8%D0%B5---rm))

#### 7. Inspect

Вырезка блока `ports`: 

![[image-10.png]]

Вырезка блока `mounts`:

![[image-11.png]]

(ссылка на лекции: [инспекция контейнера](https://github.com/tiranousor/os-modern-docker-course/blob/main/4-course/docker-and-containers/lectures/03_docker_run.md#6-%D0%B0%D0%B2%D1%82%D0%BE%D0%BC%D0%B0%D1%82%D0%B8%D1%87%D0%B5%D1%81%D0%BA%D0%BE%D0%B5-%D1%83%D0%B4%D0%B0%D0%BB%D0%B5%D0%BD%D0%B8%D0%B5---rm))

#### 8. Чистка

```shell
docker rm -f select-tag lab-web-shouryuuko short-lived
docker rmi alpine:latest nginx:alpine python:3.10.20-alpine
```

![[image-13.png]]

(ссылка на лекции: [удаление контейнеров](https://github.com/tiranousor/os-modern-docker-course/blob/main/4-course/docker-and-containers/lectures/02_commands.md#4-docker-rm), [удаление образов](https://github.com/tiranousor/os-modern-docker-course/blob/main/4-course/docker-and-containers/lectures/02_commands.md#6-docker-rmi))


### Мини-квиз

---

1. Что произойдёт с данными, созданными **внутри контейнера**, если удалить контейнер без тома?
	>Все данные будут безвозвратно удалены, поскольку хранятся в [**writable layer**](https://docs.docker.com/engine/storage/drivers/#storage-drivers-versus-docker-volumes), который не сохраняется после завершения работы.
2. Чем отличается порт **хоста** от порта **в контейнере**?
	>Порт хоста является физическим портов на реальной машине, тогда как порт в контейнере является виртуальным.
3. Для чего нужна пара флагов интерактивного запуска, и когда одного из них достаточно?
	>Флаг `-i` оставляет открытым поток ввода, чтобы контейнер мог принимать команды. Флаг `-t` создает виртуальный терминал. Зачастую достаточно флага `i`, для передачи команд контейнеру.
4. Что показывает `Mounts` в `inspect` и как понять, что это именно bind‑mount?
	>Показывает все подключенные к контейнеру внешние директории и тома. В случае монтирования конкретного каталога (bind-mount) в типе указывается `bind`.
5. Почему образ может не удаляться, и что нужно сделать перед удалением?
	>Потому что он используется существующим контейнером. Безопасный вариант решения проблемы -  удалить все контейнеры использующие данный образ, менее безопасный - указать флаг `-f` для принудительного удаления образа (но это не удалит контейнеры, использующие этот образ).