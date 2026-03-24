# Docker Best Practices

## Описание проекта

В данном репозитории представлены примеры **плохих** и **хороших** практик написания Dockerfile и Docker Compose файлов. 

Цель работы — понять разницу между подходами и научиться создавать оптимизированные образы.

<img width="960" height="760" alt="image" src="https://github.com/user-attachments/assets/799b5dd1-21dc-4c70-aafc-777fc97dd0e1" />

## Структура репозитория

├── bad/
│ ├── Dockerfile
│ ├── docker-compose.yml
│ └── .env
├── good/
│ ├── Dockerfile
│ ├── docker-compose.yml
│ └── .dockerignore
└── README.md


## Часть 0: Подготовка окружения

Работать с Docker на Windows неудобно, так что я установила виртуальную машину с Linux Ubuntu. Параметры модели согласовала с мощностью своего ноутбука, чтобы ничего не висло:
### Параметры ноутбука
Память
  16,0 ГБ
  
ЦП
	Ядра:	10

### Параметры виртуальной машины
<img width="691" height="430" alt="image" src="https://github.com/user-attachments/assets/5f5455c0-eb31-4504-9b86-656950c1634f" />

### Запуск ВМ

Первым же делом ВМ захватила мою мышь!! И не отдавала! 

> Хост-кнопка - правый Ctrl, возвращает мышь. с помощью этого можно сделать скриншот ВМ с сохранением на Windows

Выбираю английский язык, интерактивную установку, скачиваю дополнительные драйвера для WiFi и медиа.

 <img width="486" height="378" alt="image" src="https://github.com/user-attachments/assets/a211c03d-378a-4c9b-82bf-39b59b674edd" />

После установки Ubuntu запускаю следующие программы в терминале:

## Часть 1: Dockerfile

### Плохие практики в Dockerfile

#### 1. Использование тега `latest`

**Проблема:**  
В плохом Dockerfile используется базовый образ с тегом `latest`:

```dockerfile
FROM ubuntu:latest
```

Это плохая практика, потому что:

- Тег latest может указывать на разные версии образа в разное время,
- Невозможно воспроизвести сборку через месяц или год,
- Обновление базового образа может сломать ваше приложение.

**Решение в хорошем Dockerfile**:
```dockerfile
FROM ubuntu:22.04
```

**Фиксированная версия** обеспечивает воспроизводимость и стабильность сборки.

#### 2. Запуск от имени root

**Проблема:**  
В плохом Dockerfile используется базовый образ с тегом `latest`:

```dockerfile
RUN apt-get update && apt-get install -y nginx
CMD ["nginx", "-g", "daemon off;"]
```

Это плохая практика, потому что:

- Контейнер запускается от root, что создает уязвимость безопасности,
- Если злоумышленник получит доступ к контейнеру, у него будут максимальные привилегии.


**Решение в хорошем Dockerfile**:
```dockerfile
RUN useradd -m -u 1000 appuser
USER appuser
```

Создание **обычного пользователя** ограничивает права внутри контейнера.

#### 3. Отсутствие .dockerignore

**Проблема:**  

Это плохая практика, потому что:

- Без файла .dockerignore в образ попадают:
 - Файлы .git
 - Временные файлы
 - Локальные зависимости
 - Файлы с паролями и ключами
    
Это увеличивает размер образа и создает риски безопасности.


**Решение в хорошем Dockerfile**:

Создан файл .dockerignore:

```dockerfile
.git
node_modules
*.log
.env
__pycache__
```

В итоге **размер уменьшился** с до .

### Плохие практики с Docker Container даже при хорошем Dockerfile
#### 1. Запуск с флагом `--privileged`

Пример: 

```bash
docker run --privileged my-app
```

Это плохая практика, потому что:

- Контейнер получает полный доступ к хост-системе,
- Обходит все ограничения безопасности Docker,
- Эквивалентно запуску приложения напрямую на сервере.

Правильным подходом будет использовать **минимально необходимые права** и capabilities.

#### 2. Хранение данных внутри контейнера

Пример: 

```bash
docker run -d my-database
```

Это плохая практика, потому что:

- База данных хранится внутри контейнера. При удалении контейнера данные будут **потеряны**.

Правильным подходом будет **сохранение данных на хосте** с помощью volumes или bind mounts.

Пример:

```bash
docker run -d -v /host/data:/var/lib/mysql my-database
```

## Часть 2: Docker Compose

### Плохие практики в Docker Compose

#### 1. Захардкоженные значения

Пример:

```yaml
services:
  web:
    image: nginx:1.21
    ports:
      - "8080:80"
    environment:
      - DB_HOST=192.168.1.100
      - DB_PASSWORD=secret123
```

Это плохая практика, потому что:

- Любой мошенник и не только может украсть пароли,
- Нельзя быстро изменить конфигурацию,
- В разных окружениях (dev/prod) может не работать.

**Решение в хорошем Docker Compose**:
```yaml
services:
  web:
    image: nginx:${NGINX_VERSION:-1.21}
    ports:
      - "${WEB_PORT:-8080}:80"
    env_file:
      - .env
```

Использование **переменных окружения** и **env_file**

#### 2. Отсутствие healthcheck

Пример:

```yaml
services:
  api:
    image: my-api:latest
    # нет проверки здоровья
```

Это плохая практика, потому что:

- Оркестратор не знает, в каком состоянии система и готова ли она принимать запросы.

**Решение в хорошем Docker Compose**:
```yaml
services:
  api:
    image: my-api:1.0
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
```

Важно следить за **здоровьем** не только тела, но и системы:)

<img width="150" height="227" alt="image" src="https://github.com/user-attachments/assets/0505a46f-884a-47ee-b6fa-d26b7e775a51" />

#### 3. Безымянные volumes

Пример:

```yaml
services:
  db:
    volumes:
      - ./data:/var/lib/mysql
```

Это плохая практика, потому что:

- Безымянными volumes сложно управлять и переносить.

**Решение в хорошем Docker Compose**:
```yaml
services:
  db:
    volumes:
      - db-data:/var/lib/mysql

volumes:
  db-data:
    name: myapp-database-data
```

### Сетевая изоляция контейнеров
#### Задача
Настроить сервисы так, чтобы контейнеры поднимались вместе, но не видели друг друга по сети.
#### Решение
Используем разделение сетей и явное указание соединений:
```yaml
version: '3.8'

services:
  frontend:
    image: nginx:alpine
    networks:
      - public
      
  backend:
    image: node:18
    networks:
      - public
      - private
      
  database:
    image: postgres:15
    networks:
      - private

networks:
  public:
    driver: bridge
  private:
    driver: bridge
    internal: true
```
#### Принцип работы
- Frontend находится только в сети public и доступен извне,
- Backend находится в обеих сетях, может общаться с frontend и database,
- Database находится только в сети private с флагом internal: true,
- Frontend не может напрямую подключиться к database,
- Флаг internal: true создает изолированную сеть без доступа извне и без выхода в интернет.

#### Сравнение плохого и хорошего Docker Compose
| Метрика | Плохой | Хороший |
|-------------|-------------|-------------|
| Размер образа    | Ячейка 2    | Ячейка 3    |
| Воспроизводимость    | Ячейка 5    | Ячейка 6    |
| Безопасность | | |
| Время сборки | | |

#### Как запустить
1. Плохой вариант (не рекомендуется)
```bash
cd bad
docker build -t bad-app .
docker run -p 8080:80 bad-app
```
2. Хороший вариант
```bash
cd good
docker-compose up --build
```
