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

### Параметры виртуальной машины (ВМ)
<img width="691" height="430" alt="image" src="https://github.com/user-attachments/assets/5f5455c0-eb31-4504-9b86-656950c1634f" />

### Запуск ВМ

Первым же делом ВМ захватила мою мышь!! И не отдавала! 

> Хост-кнопка - правый Ctrl, возвращает мышь. с помощью этого можно сделать скриншот ВМ с сохранением на Windows

Выбираю английский язык, интерактивную установку, скачиваю дополнительные драйвера для WiFi и медиа.

 <img width="486" height="378" alt="image" src="https://github.com/user-attachments/assets/a211c03d-378a-4c9b-82bf-39b59b674edd" />

### Установка Docker на ВМ

После установки Ubuntu построчно запускаю следующие команды в терминале (Ctrl + Alt + T):

```bash
sudo apt update && sudo apt upgrade -y # обновление установленных программ
sudo apt install -y git curl wget vim net-tools # установка базовых утилит для работы с Git, Docker, сетью
ping -c 3 google.com # проверка интернета
curl -fsSL https://get.docker.com | sh # официальный скрипт установки Docker
sudo usermod -aG docker $USER # добавляем текущего пользователя в группу docker (чтобы не писать sudo перед docker)
newgrp docker # изменение прав без перезагрузки
docker --version 
docker run hello-world # убедиться, что Docker жив
sudo reboot # перезагрузка, чтобы все изменения применились корректно
```

### Создание структуры проекта на ВМ в терминале

```bash
mkdir ~/dockerfiles
cd ~/dockerfiles # создаем и открываем нужную папку
mkdir bad good # создаем подпапки

# создаем файлы
touch bad/Dockerfile
touch bad/docker-compose.yml
touch bad/.env

touch good/Dockerfile
touch good/docker-compose.yml
touch good/.dockerignore

# проверяем структуру
ls -R
```

<img width="319" height="178" alt="image" src="https://github.com/user-attachments/assets/8acbf45b-c09f-4a94-9c1e-fb333f9a119d" />


### Заполнение файлов
Это можно сделать в редакторе nano / vim

1. **bad/Dockerfile**

```bash
nano bad/Dockerfile
```

Вставляем следующий скрипт (детальный разбор будет ниже):

```dockerfile
FROM ubuntu:latest

RUN apt-get update && apt-get install -y nginx

CMD ["nginx", "-g", "daemon off;"]
```

> Сохраняем: Ctrl + O + Enter

> Закрываем: Ctrl + X

> Для чтения и проверки, что файл не пустой, используем команду `cat`

<img width="603" height="161" alt="image" src="https://github.com/user-attachments/assets/5bf9128f-7fc1-43fc-86bf-bb56d29b3e44" />

По аналогии заполняем остальные файлы:

2. **good/Dockerfile**
   
```dockerfile
FROM ubuntu:22.04

RUN apt-get update && \
    apt-get install -y nginx && \
    apt-get clean

RUN useradd -m -u 1000 appuser

USER appuser

CMD ["nginx", "-g", "daemon off;"]
```

3. **good/.dockerignore**

```
.git
node_modules
*.log
.env
__pycache__
*.md
```

4. **bad/docker-compose.yml**

```yaml
version: '3.8'

services:
  web:
    image: nginx:1.21
    ports:
      - "8080:80"
    environment:
      - DB_HOST=192.168.1.100
      - DB_PASSWORD=secret123
```

5. **good/docker-compose.yml (с сетевой изоляцией)**
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
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 30s
      timeout: 10s
      retries: 3

networks:
  public:
    driver: bridge
  private:
    driver: bridge
    internal: true

volumes:
  db-data:
    name: myapp-database-data
```

6. **bad/.env**

```
DB_PASSWORD=secret123
DB_HOST=192.168.1.100
```

### Тестирование сборки
```bash
cd ~/dockerfiles

# собираем плохой образ
cd bad
docker build -t bad-app .



# собираем хороший образ
cd ../good
docker build -t good-app .
```

```bash
# сравниваем размеры
docker images | grep -E "bad-app|good-app"
```

<img width="599" height="175" alt="image" src="https://github.com/user-attachments/assets/57611495-94af-4eb9-b5a2-75ae5407835e" />

```bash
# история слоев (покажет разницу в подходах)
docker history bad-app
docker history good-app
```

<img width="594" height="437" alt="image" src="https://github.com/user-attachments/assets/77707cf7-c58d-42ce-9a65-90ed1604c1c3" />

<img width="598" height="524" alt="image" src="https://github.com/user-attachments/assets/d8894636-fb08-4e57-a8d9-8ee6ab39fb88" />




### Тестирование Docker Compose
```bash
sudo apt install docker-compose

docker compose up -d

# проверяем статус
docker compose ps
```

<img width="1200" height="110" alt="image" src="https://github.com/user-attachments/assets/bb8db888-65c7-4846-8a24-8fdba67eb9b1" />

```bash
# смотрим логи
docker compose logs
```

<img width="1204" height="635" alt="image" src="https://github.com/user-attachments/assets/64180ad4-8b6c-4a18-8040-21b5d1af20e4" />

```bash
# останавливаем
docker compose down
```

<img width="693" height="201" alt="image" src="https://github.com/user-attachments/assets/2935cec7-f7fe-4e44-a34e-416de539e798" />


### Пушим в репозитория на Git
```bash
cd ~/dockerfiles

# создаем пустой репозиторий
git init

# добавляем все файлы
git add .

# проверка, что все добавлено в stage area
git status
```

<img width="558" height="288" alt="image" src="https://github.com/user-attachments/assets/bd483dd1-d0f4-4a72-bf1c-94f2086cac83" />

```bash
# коммит сообщение
git commit -m "Initial commit: bad and good Docker practices"

# добавляем удаленный репозиторий
git remote add origin https://github.com/emopudge/dockerfiles.git

# пушим
git branch -M main
git push -u origin main
```

## Часть 1: Обзор Dockerfile

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

# Ресурсы
- [Docker Documentation](https://docs.docker.com/?spm=a2ty_o01.29997173.0.0.4b935171ktu8tJ)
- [Dockerfile Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/?spm=a2ty_o01.29997173.0.0.4b935171ktu8tJ)
- [Compose File Reference](https://docs.docker.com/compose/compose-file/?spm=a2ty_o01.29997173.0.0.4b935171ktu8tJ)
