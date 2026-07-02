# SRE Hello World Ansible

Ansible-проект для автоматического развёртывания тестового приложения **SRE Hello World** на **Debian 12**.

В результате работы плейбука на сервере будут автоматически установлены и настроены:

- Docker Engine + Docker Compose Plugin
- PostgreSQL
- Nginx
- PHP-приложение в Docker-контейнере

Проект содержит два сценария развёртывания:

- **site.yml** — полное развёртывание инфраструктуры и приложения;
- **deploy.yml** — обновление только PHP-приложения без изменения инфраструктуры.

---

# Требования

## Локальная машина

- Git
- Python 3
- Ansible Core 2.18

## Целевой сервер

- Debian 12
- Пользователь с правами sudo
- SSH-доступ

---

# Подготовка

## 1. Клонировать репозиторий

```bash
git clone git@github.com:mrMedov/sre-hello-world-ansible.git
cd sre-hello-world-ansible
```

---

## 2. Создать виртуальное окружение Python

### Bash / Zsh

```bash
python -m venv .venv
source .venv/bin/activate
```

### Fish

```fish
python -m venv .venv
source .venv/bin/activate.fish
```

---

## 3. Установить Ansible

```bash
pip install "ansible-core==2.18.*"
```

Проверить:

```bash
ansible --version
```

---

## 4. Установить необходимые Ansible Collection

```bash
ansible-galaxy collection install -r requirements.yml
```

---

# Настройка SSH

На локальной машине должен быть создан SSH-ключ.

Если его нет:

```bash
ssh-keygen -t ed25519
```

Скопировать публичный ключ на сервер:

```bash
ssh-copy-id USER@SERVER_IP
```

Проверить подключение:

```bash
ssh USER@SERVER_IP
```

---

# Настройка sudo

Для удобной работы Ansible рекомендуется разрешить выполнение sudo без запроса пароля.

Открыть:

```bash
sudo visudo
```

Добавить:

```text
%USER% ALL=(ALL) NOPASSWD: ALL
```

После этого запуск Ansible не потребует ввода sudo-пароля.

---

# Настройка inventory

Отредактировать файл

```
inventories/production/hosts.yml
```

Пример:

```yaml
all:
  children:
    sre_app:
      hosts:
        sre-01:
          ansible_host: 83.217.xxx.xxx
          ansible_user: viktor
```

---

# Настройка переменных

Основные параметры приложения находятся в

```
inventories/production/group_vars/sre_app.yml
```

---

# Настройка секретов

Скопировать пример:

```bash
cp inventories/production/group_vars/vault.yml.example \
   inventories/production/group_vars/vault.yml
```

Указать пароль PostgreSQL.

При необходимости зашифровать файл:

```bash
ansible-vault encrypt inventories/production/group_vars/vault.yml
```

---

# Проверка подключения

```bash
ansible all -m ping
```

Ожидаемый результат:

```text
SUCCESS => pong
```

---

# Проверка плейбука без изменений

```bash
ansible-playbook playbooks/site.yml --check --diff
```

---

# Использование

## Полное развёртывание

Устанавливает и настраивает:

- Docker
- PostgreSQL
- Nginx
- PHP-приложение

```bash
ansible-playbook playbooks/site.yml
```

---

## Деплой новой версии приложения

Подтягивает последнюю версию приложения из Git-репозитория, обновляет конфигурацию (при необходимости) и перезапускает PHP-контейнер.

Инфраструктура (Docker, PostgreSQL, Nginx) при этом не переустанавливается.

```bash
ansible-playbook playbooks/deploy.yml
```

---

# Проверка

После завершения развёртывания приложение должно быть доступно по адресу

```
http://SERVER_IP
```

Либо проверить с сервера:

```bash
curl http://localhost
```

---

# Структура проекта

```
roles/
├── common/        Базовая настройка системы
├── docker/        Установка Docker
├── postgresql/    Установка и настройка PostgreSQL
├── nginx/         Установка и настройка Nginx
└── app/           Развёртывание приложения
```

---

# Поддерживаемые ОС

На данный момент протестировано на

- Debian 12

Архитектура проекта позволяет добавить поддержку других дистрибутивов (например Debian 13, Ubuntu, Rocky Linux, AlmaLinux и др.) через отдельные task-файлы ролей.

---

# Идемпотентность

Плейбук является идемпотентным.

Повторный запуск не изменяет уже корректно настроенную систему и применяет изменения только при необходимости.