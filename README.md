# Лаб. П06 — Деплой веб-сервера с помощью роли Ansible + CI

Роль `nginx_vhosts` устанавливает nginx и настраивает виртуальные сайты.
Список сайтов задаётся параметром роли `sites` (playbook / play / vars).
Для каждого vhost создаются каталог, конфиг сервера из шаблона и индексная
страница из шаблона. После изменения конфигов nginx перезапускается через
handler (идемпотентно).

## Стенд

vm1 (control, 192.168.227.15) → vm2 (managed, 192.168.227.16),
Ubuntu 24.04, VMware NAT.

## Структура

```
.
├── ansible.cfg, inventory.ini
├── playbook.yml                       # play с ролью + правка /etc/hosts на control-ноде
├── roles/nginx_vhosts/
│   ├── defaults/main.yml              # sites (по умолчанию), nginx_vhosts_web_root
│   ├── tasks/main.yml                 # установка, каталоги, конфиги, симлинки, index, service
│   ├── handlers/main.yml              # Restart nginx
│   ├── templates/vhost.conf.j2        # конфиг виртуального хоста
│   ├── templates/index.html.j2        # стартовая страница vhost
│   └── meta/main.yml
├── .github/workflows/
│   ├── devops_course_pipeline.yml     # тестовый workflow (смотрим на runner GitHub)
│   └── lint.yml                       # ansible-lint
└── screenshots/
```

## Запуск (на vm1)

```bash
ansible-playbook playbook.yml
curl http://mehmat.ru              # имена добавлены в /etc/hosts вторым play
curl -H "Host: fizfak.ru" http://192.168.227.16   # либо через HTTP-заголовок Host
```

Сайты (`mehmat.ru`, `fizfak.ru`, `etis.com`) можно переопределить:

```yaml
vars:
  sites: [site1.ru, site2.ru]
```

## CI (GitHub Actions)

- `devops_course_pipeline.yml` — демонстрационный workflow на `ubuntu-latest`
  (`uptime`, `pwd`, `whoami`), запускается на каждый push.
- `lint.yml` — `ansible-lint` на push и pull request в ветку `main`.
  Локально линтер проходит на профиле `production`. Правило
  `var-naming[no-role-prefix]` подавлено точечно для переменной `sites`,
  потому что её имя задано условием работы.

Статусы запусков — на вкладке **Actions** репозитория.

## Скриншоты

1. `screenshots/01-playbook-run.png` — первый запуск (роль, handler, правка hosts).
2. `screenshots/02-idempotent-and-lint.png` — повторный запуск (`changed=0`) и результат `ansible-lint`.
3. `screenshots/03-curl-by-name.png` — три сайта отвечают по имени, проверка через заголовок Host.
4. `screenshots/04-github-actions.png` — успешные запуски workflows на GitHub.
