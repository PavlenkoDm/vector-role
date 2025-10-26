# Ansible Role: Vector

[![License](https://img.shields.io/badge/license-MIT-brightgreen.svg)](LICENSE)

## Описание

Роль для установки и настройки Vector - высокопроизводительного сборщика логов на Ubuntu 22.04/20.04.

Vector собирает логи из различных источников и отправляет их в различные хранилища. Эта роль настраивает Vector для отправки логов в ClickHouse.

Роль выполняет следующие действия:

- Устанавливает необходимые зависимости (curl, wget)
- Загружает и распаковывает бинарный файл Vector
- Копирует бинарник в `/usr/local/bin`
- Создаёт конфигурационный файл для отправки логов в ClickHouse
- Настраивает systemd сервис для Vector и запускает его

## Требования

- Ansible >= 2.12
- Ubuntu 22.04 (Jammy) или Ubuntu 20.04 (Focal)
- Права sudo на целевом хосте
- Доступ к интернету для загрузки Vector
- Работающий экземпляр ClickHouse (для отправки логов)

## Переменные роли

### defaults/main.yml

| Переменная                   | Описание                           | Значение по умолчанию |
| ---------------------------- | ---------------------------------- | --------------------- |
| `vector_version`             | Версия Vector для установки        | `"0.50.0"`            |
| `vector_config_dir`          | Директория для конфигурации Vector | `/etc/vector`         |
| `vector_user`                | Пользователь для запуска Vector    | `vector`              |
| `vector_clickhouse_host`     | Адрес сервера ClickHouse           | `"127.0.0.1"`         |
| `vector_clickhouse_port`     | Порт ClickHouse                    | `9000`                |
| `vector_clickhouse_database` | Имя базы данных в ClickHouse       | `"logs"`              |
| `vector_clickhouse_table`    | Имя таблицы в ClickHouse           | `"events"`            |

### vars/main.yml

Технические переменные отсутствуют (роль использует только defaults).

## Зависимости

Роль не имеет зависимостей от других ролей.

**Важно:** Для корректной работы необходим работающий экземпляр ClickHouse на указанном хосте.

## Пример использования

### Базовое использование

```yaml
---
- name: Install Vector
  hosts: vector_servers
  become: yes

  roles:
    - role: vector
      vars:
        vector_clickhouse_host: "192.168.1.10"
```

### С кастомными переменными

```yaml
---
- name: Install Vector with custom settings
  hosts: vector_servers
  become: yes

  roles:
    - role: vector
      vars:
        vector_version: "0.51.0"
        vector_clickhouse_host: "clickhouse.example.com"
        vector_clickhouse_port: 9000
        vector_clickhouse_database: "my_logs"
        vector_clickhouse_table: "system_events"
```

### Использование с динамическим получением IP ClickHouse

```yaml
---
- name: Install Vector with dynamic ClickHouse host
  hosts: vector_servers
  become: yes

  roles:
    - role: vector
      vars:
        # Получаем IP из inventory другого хоста
        vector_clickhouse_host: "{{ hostvars['clickhouse-01']['ansible_host'] }}"
```

### Использование с tags

```yaml
# Установить только Vector без запуска сервиса
ansible-playbook site.yml --tags vector --skip-tags service
```

## Конфигурация

Роль создаёт конфигурационный файл Vector (`/etc/vector/vector.toml`) с следующими компонентами:

**Source (источник):**

- Тип: `file`
- Файл: `/var/log/syslog`

**Sink (получатель):**

- Тип: `clickhouse`
- База данных: настраивается через переменные
- Таблица: настраивается через переменные

## Handlers

Роль использует следующие handlers:

- `restart vector` - перезапускает сервис Vector
- `reload systemd` - перезагружает конфигурацию systemd

Handlers вызываются автоматически при изменении конфигурационных файлов.

## Теги

Роль использует следующие теги:

- `vector` - применяется ко всем задачам роли

## Структура файлов

```
vector-role/
├── tasks/
│   └── main.yml              # Основные задачи установки
├── handlers/
│   └── main.yml              # Обработчики событий
├── templates/
│   ├── vector.toml.j2       # Конфигурация Vector
│   └── vector.service.j2    # Systemd unit файл
├── defaults/
│   └── main.yml              # Переменные по умолчанию
├── vars/
│   └── main.yml              # Внутренние переменные
├── meta/
│   └── main.yml              # Метаданные роли
└── README.md                 # Документация
```

## Устранение неполадок

### Vector не запускается

Проверьте логи:

```bash
journalctl -u vector -n 50
```

### Логи не попадают в ClickHouse

1. Проверьте доступность ClickHouse:

```bash
   telnet <clickhouse_host> 9000
```

2. Проверьте конфигурацию Vector:

```bash
   /usr/local/bin/vector validate /etc/vector/vector.toml
```

3. Проверьте, что таблица существует в ClickHouse:

```sql
   SHOW TABLES FROM logs;
```

## Поддерживаемые платформы

- Ubuntu 22.04 (Jammy Jellyfish)
- Ubuntu 20.04 (Focal Fossa)

## Лицензия

MIT

## Автор

Dmitry Pavlenko  
GitHub: [@PavlenkoDm](https://github.com/PavlenkoDm)  
Создано в рамках курса DevOps от Netology

## Ссылки

- [Официальная документация Vector](https://vector.dev/docs/)
- [Vector GitHub](https://github.com/vectordotdev/vector)
- [Конфигурация ClickHouse sink](https://vector.dev/docs/reference/configuration/sinks/clickhouse/)
