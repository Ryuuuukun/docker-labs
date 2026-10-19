Ссылка на образ: https://hub.docker.com/r/shouryuuko/image-lab/tags

### Преимущества данного образа

---

- Многоэтапная сборка, минимизация размера образа: в образ попадают только необходимые зависимости, без временных файлов, кеша пакетного менеджера.
- Запуск от имени непривилегированного пользователя, с ограниченными правами: права выделнны только на директорию приложения.
- Поддержка режима `--read-only`, за счет ограничения кеширования и перенаправления логов приложения на прямую в консоль контейнера.
- Корректная обработка сигнала `SIGTERM`: `gunicorn` по умолчанию корректно обрабатывает принудительное завершение работы контейнера 

Исключение: передача аргументов `LOGIN` и `TOKEN` через аргументы сборки небезопасно, поскольку их можно просмотреть выполнив команду `docker history...`

### Проверка сборки

---

#### История сборки

<img src="https://github.com/Ryuuuukun/docker-labs/blob/main/lab-02/proof/history.png"></img>

#### Проверка эндпоинтов

<img src="https://github.com/Ryuuuukun/docker-labs/blob/main/lab-02/proof/endpoints.png"></img>

#### Проверка поддержки работы с `--read-only`

<img src="https://github.com/Ryuuuukun/docker-labs/blob/main/lab-02/proof/read_only.png"></img>

#### Проверка `SIGTERM`

<img src="https://github.com/Ryuuuukun/docker-labs/blob/main/lab-02/proof/sigterm.png"></img>
