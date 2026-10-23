# nginx-role

Устанавливает Nginx, создаёт виртуальный хост и статическую страницу.

| Переменная | По умолчанию | Где задана |
|---|---|---|
| `nginx_port` | `80` | defaults |
| `site_name` | `My Cool Website` | defaults |
| `site_dir` | `site_name` в нижнем регистре, пробелы → `-` | defaults |
| `nginx_version` | `latest` (значение `state` модуля apt) | vars |

Пример и запуск — в README.md репозитория.
