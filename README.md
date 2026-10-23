# ansible-nginx-role

Ansible-роль `nginx-role`: устанавливает Nginx из репозитория дистрибутива, настраивает виртуальный хост по шаблону и публикует статическую страницу с названием сайта.

## Требования

- Control node: Linux или WSL, Ansible (ansible-core 2.14+).
- Управляемые хосты: Ubuntu 22.04 / 24.04 или Debian 12, SSH-доступ пользователем с правами sudo.

## Структура

```
ansible-nginx-role/
├── inventory.ini              # пример inventory
├── site.yml                   # плейбук применения роли
└── roles/nginx-role/
    ├── defaults/main.yml      # переменные по умолчанию
    ├── vars/main.yml          # переменные роли
    ├── tasks/main.yml         # задачи
    ├── handlers/main.yml      # проверка конфига и перезапуск Nginx
    ├── templates/
    │   ├── nginx-site.conf.j2 # виртуальный хост
    │   └── index.html.j2      # страница сайта
    ├── meta/main.yml          # метаданные роли
    ├── files/
    └── tests/
```

## Переменные

| Переменная | По умолчанию | Файл | Назначение |
|---|---|---|---|
| `nginx_port` | `80` | defaults | порт виртуального хоста, проверяется на диапазон 1–65535 |
| `site_name` | `My Cool Website` | defaults | название сайта на странице |
| `site_dir` | `my-cool-website` | defaults | имя каталога `/var/www/<site_dir>` и файла конфигурации; по умолчанию получается из `site_name` (нижний регистр, пробелы → `-`) |
| `nginx_version` | `latest` | vars | значение `state` для модуля apt: `latest` обновляет пакет, `present` только устанавливает |

Переменные из `defaults` переопределяются в inventory, `group_vars` или через `-e`. `nginx_version` задана в `vars` и имеет более высокий приоритет: её меняют только через `-e`.

## Использование

1. Указать IP-адрес сервера в `inventory.ini`:

   ```ini
   [webservers]
   web1 ansible_host=<IP_ВМ> ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/cloud_vm
   ```

2. Проверить связь и запустить плейбук:

   ```bash
   ansible -i inventory.ini webservers -m ping
   ansible-playbook -i inventory.ini site.yml
   ```

3. Проверить сайт:

   ```bash
   curl http://<IP_ВМ>
   ```

Повторный запуск не вносит изменений (`changed=0`).

### Свои значения переменных

```bash
ansible-playbook -i inventory.ini site.yml -e nginx_port=8080 -e "site_name='Terekhov DevOps'"
```

### Теги

| Тег | Задачи |
|---|---|
| `install` | установка Nginx |
| `config` | проверка порта, конфигурация хоста, включение сайта, отключение сайта по умолчанию |
| `content` | каталог сайта и `index.html` |
| `service` | запуск и автозагрузка Nginx |
| `nginx` | все задачи роли |

```bash
ansible-playbook -i inventory.ini site.yml --tags content
ansible-playbook -i inventory.ini site.yml --list-tags
```

## Что делает роль

1. Проверяет, что `nginx_port` в диапазоне 1–65535.
2. Устанавливает пакет `nginx`, предварительно обновив кеш apt.
3. Создаёт `/var/www/<site_dir>` и кладёт туда `index.html` из шаблона.
4. Создаёт `/etc/nginx/sites-available/<site_dir>.conf`, включает его ссылкой в `sites-enabled` и отключает сайт `default`, который иначе перехватывает запросы по IP.
5. Запускает Nginx и добавляет его в автозагрузку.
6. При изменении конфигурации проверяет её `nginx -t` и только после успешной проверки перезапускает Nginx.
