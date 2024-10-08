# Инструкция по использованию утилиты `gsync`

## 1. Общее описание и назначение утилиты

`gsync` — это утилита для синхронизации файлов и каталогов между локальными и удалёнными серверами, построенная на основе `rsync`. Основная функция `gsync` — отслеживать изменения в указанной директории и автоматически копировать изменённые файлы в целевую директорию. Утилита поддерживает гибкую настройку через конфигурационный файл и может игнорировать определённые файлы и папки с помощью файлов `.nogsync`, аналогично `.gitignore` в Git.

## 2. Инструкция по установке зависимостей

Для работы `gsync` требуется `rsync`, `inotify-tools` и `awk`. Установите их в зависимости от используемой операционной системы.

### Ubuntu/Debian

```bash
sudo apt update
sudo apt install rsync inotify-tools gawk
```

### CentOS/RHEL

```bash
sudo yum install rsync inotify-tools gawk
```

### Fedora

```bash
sudo dnf install rsync inotify-tools gawk
```

### Arch Linux

```bash
sudo pacman -S rsync inotify-tools gawk
```

### macOS (через Homebrew)

```bash
brew install rsync inotify-tools gawk
```

## 3. Как скачать утилиту из репозитория

Скачайте `gsync` из официального репозитория GitHub и добавьте права на выполнение.

```bash
git clone https://github.com/e-bash/gsync.git
cd gsync
chmod +x gsync
```

Теперь утилита `gsync` готова к использованию.

## 4. Подробное описание конфигурационного файла

Конфигурационный файл содержит настройки для работы `gsync`. По умолчанию, утилита ищет файл конфигурации в домашней директории пользователя (`~/.gsync`) или принимает его в виде параметра командной строки.

### Пример конфигурационного файла

```bash
[OPTIONS]
source="/path/to/source/directory/" # Локальная директория для отслеживания. 
target="user@server:/path/to/target/directory/" # Целевая директория для копирования.
links=true # Сохранение символьных ссылок.
perms=false # Сохранение прав доступа.
times=true # Сохранение времени изменения.
group=false # Сохранение групповой принадлежности.
owner=false # Сохранение владельца.
```

### Инструкция по созданию конфигурационного файла

1. Создайте файл конфигурации в домашней директории:

```bash
nano ~/.gsync
```

2. Добавьте необходимые параметры в файл.

3. Сохраните файл и закройте редактор.

Параметры `source` и `target` обязательны. Остальные параметры являются опциональными и могут быть добавлены по необходимости.

### Дополнительная информация о целевой директории

Целевая директория (`target`), куда будут синхронизироваться файлы, может быть как локальной, так и удалённой. Если целевая директория указана в формате `user@server:/path/to/target/directory/`, то утилита `gsync` использует подключение по SSH. В этом случае `rsync` требует наличия доступа к серверу через SSH, который может быть установлен с использованием пароля или SSH-ключей.

- **Локальная целевая директория**: Если путь в `target` указывает на локальную директорию, никаких дополнительных настроек не требуется.
- **Удалённая целевая директория**: Требуется подключение по SSH. 

#### Подключение по паролю

Для разового запуска `gsync` можно использовать подключение по паролю. При каждом запуске скрипт запросит пароль пользователя удалённого сервера.

```bash
./gsync --config=/path/to/config
```

При использовании пароля вы не можете автоматизировать процесс, так как необходимо вручную вводить пароль при каждом запуске. Это не подходит для длительных или автоматизированных процессов.

#### Подключение с использованием SSH-ключей

Использование SSH-ключей позволяет автоматизировать процесс подключения к удалённым серверам без необходимости вводить пароль при каждом запуске.

##### Шаг 1: Генерация SSH-ключа

Чтобы создать SSH-ключ, выполните следующую команду в терминале:

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

Эта команда создаст SSH-ключи в директории `~/.ssh`. По умолчанию будут созданы два файла:

- `~/.ssh/id_rsa` — приватный ключ
- `~/.ssh/id_rsa.pub` — публичный ключ

##### Шаг 2: Добавление публичного ключа на удалённый сервер

Для добавления публичного ключа на удалённый сервер используйте следующую команду:

```bash
ssh-copy-id user@server
```

Эта команда добавит содержимое `~/.ssh/id_rsa.pub` в файл `~/.ssh/authorized_keys` на удалённом сервере для указанного пользователя. Теперь вы можете подключаться к серверу без пароля.

##### Шаг 3: Настройка SSH-конфигурации в `~/.ssh/config`

Файл конфигурации `~/.ssh/config` позволяет вам настроить параметры подключения для различных серверов, упрощая команды для SSH и `gsync`.

Пример файла `~/.ssh/config`:

```bash
Host myserver
    HostName server.example.com
    User user
    Port 22
    IdentityFile ~/.ssh/id_rsa
    Compression yes
    ServerAliveInterval 60
```

- **Host**: Имя, под которым вы будете обращаться к серверу.
- **HostName**: Фактическое доменное имя или IP-адрес сервера.
- **User**: Пользователь, под которым вы будете подключаться.
- **Port**: Порт SSH (по умолчанию 22).
- **IdentityFile**: Путь к приватному ключу.
- **Compression**: Включение сжатия для SSH-сессий.
- **ServerAliveInterval**: Интервал отправки пакетов для проверки активности сессии.

С такой конфигурацией вы можете использовать `myserver` в качестве псевдонима при запуске `gsync`:

```bash
./gsync --config=/path/to/config
```

В этом случае целевая директория в конфигурационном файле `gsync` может быть указана как `myserver:/path/to/target/directory/`.

## 5. Описание возможностей использования `.nogsync`

Файлы `.nogsync` позволяют исключить из синхронизации определённые файлы и папки. Эти файлы могут находиться в любом месте внутри отслеживаемой директории.

### Пример файла `.nogsync`

```bash
# Игнорировать папку logs и всё её содержимое
logs/

# Игнорировать все файлы с расширением .tmp
*.tmp

# Игнорировать конкретный файл
secret.txt
```

Файл `.nogsync` игнорирует строки, начинающиеся с `#`, позволяя добавлять комментарии. Исключённые пути могут быть как относительными, так и с использованием масок, например `*.log`.

## 6. Порядок использования

### Разовый запуск из командной строки

Вы можете запустить `gsync` разово из командной строки:

```bash
./gsync --config=/path/to/config
```

**Преимущества**: Простой запуск, подходит для разовых операций.

**Недостатки**: Не подходит для постоянного отслеживания изменений.

### Запуск из командной строки в фоновом режиме

Запуск в фоновом режиме с использованием `&`:

```bash
./gsync --config=/path/to/config &
```

**Преимущества**: Утилита продолжает работать после закрытия терминала.

**Недостатки**: Неудобство управления процессом, возможные проблемы с отслеживанием состояния.

### Запуск в качестве службы

Создание системной службы для постоянного мониторинга:

1. Создайте файл службы:

```bash
sudo nano /etc/systemd/system/gsync.service
```

2. Добавьте следующее содержимое:

```bash
[Unit]
Description=GSync Service
After=network.target

[Service]
ExecStart=/path/to/gsync --config=/path/to/config
Restart=always

[Install]
WantedBy=multi-user.target
```

3. Активируйте и запустите службу:

```bash
sudo systemctl enable gsync
sudo systemctl start gsync
```

**Преимущества**: Автоматический запуск при старте системы, надёжность.

**Недостатки**: Требует прав суперпользователя для настройки.

### Запуск с помощью Supervisor

Установка и настройка Supervisor для управления `gsync`:

1. Установите Supervisor:

**Ubuntu/Debian:**

```bash
sudo apt install supervisor
```

**CentOS/RHEL:**

```bash
sudo yum install supervisor
```

2. Создайте конфигурационный файл для `gsync`:

```bash
sudo nano /etc/supervisor/conf.d/gsync.conf
```

3. Добавьте следующее содержимое:

```bash
[program:gsync]
command=/path/to/gsync --config=/path/to/config
autostart=true
autorestart=true
stderr_logfile=/var/log/gsync.err.log
stdout_logfile=/var/log/gsync.out.log
```

4. Перезапустите Supervisor:

```bash
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start gsync
```

**Преимущества**: Удобное управление процессами, логирование.

**Недостатки**: Требует установки дополнительного ПО.

### Достоинства и недостатки каждого способа

| Способ                              | Достоинства                                                                 | Недостатки                                              |
|--------------------------------------|------------------------------------------------------------------------------|---------------------------------------------------------|
| Разовый запуск из командной строки   | Легко запустить, подходит для одноразовых задач                              | Не подходит для долгосрочного использования             |
| Запуск в фоновом режиме              | Удобен для длительных операций в одном сеансе                                | Сложно управлять и отслеживать процесс                  |
| Запуск в качестве службы             | Надёжно, автоматически стартует, управляется системными средствами           | Требует настройки и прав суперпользователя              |
| Запуск с помощью Supervisor          | Удобное управление, логирование, автоматический перезапуск                   | Требует установки и настройки Supervisor                |

Используйте `gsync` в зависимости от ваших задач и сценариев использования, чтобы максимально эффективно управлять синхронизацией файлов.
