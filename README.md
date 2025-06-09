# ansible_install_docker
install docker + docker-compose for ubuntu, debian, centos and other linux distributions

# Автоматизация получения последней версии Docker Compose

## Проблема
В исходном playbook версия docker-compose была захардкожена (`v2.36.0`), что требует ручного обновления при выходе новых версий.

## Решения

### Вариант 1: Переменная с lookup (Рекомендуемый)
```yaml
vars:
  docker_compose_version: "{{ lookup('url', 'https://api.github.com/repos/docker/compose/releases/latest', split_lines=False) | from_json | json_query('tag_name') }}"
```

**Преимущества:**
- Простой и элегантный
- Выполняется один раз при запуске playbook
- Использует встроенные возможности Ansible

### Вариант 2: URI модуль + переменная
```yaml
- name: Get latest docker-compose version
  uri:
    url: https://api.github.com/repos/docker/compose/releases/latest
    method: GET
    return_content: yes
  register: docker_compose_latest

- name: Install docker-compose
  # Используем {{ docker_compose_latest.json.tag_name }}
```

**Преимущества:**
- Больше контроля над запросом
- Можно добавить обработку ошибок
- Читаемый код

### Вариант 3: Shell с curl и jq
```bash
LATEST_VERSION=$(curl -s https://api.github.com/repos/docker/compose/releases/latest | grep '"tag_name":' | sed -E 's/.*"([^"]+)".*/\1/')
```

**Преимущества:**
- Не зависит от модулей Ansible
- Работает везде где есть curl

### Вариант 4: get_url модуль (Самый надёжный)
```yaml
- name: Download docker-compose
  get_url:
    url: "https://github.com/docker/compose/releases/download/{{ compose_release.json.tag_name }}/docker-compose-{{ ansible_system }}-{{ ansible_architecture }}"
    dest: /usr/local/bin/docker-compose
    mode: '0755'
    force: yes
```

**Преимущества:**
- Встроенная проверка целостности
- Автоматическое создание резервных копий
- Лучшая обработка ошибок

### Вариант 5: Готовая роль
```bash
ansible-galaxy install geerlingguy.docker
```

**Преимущества:**
- Поддерживается сообществом
- Включает все best practices
- Автоматические обновления

## Установка готовой роли

```bash
# Установка роли и коллекций
ansible-galaxy install -r requirements.yml

# Использование в playbook
- hosts: all
  roles:
    - geerlingguy.docker
  vars:
    docker_install_compose: true
    docker_compose_version: "latest"
```

## Рекомендация

Для установки Docker используйте:
- **playbook.yml** - основной файл с поддержкой Ubuntu, Debian, CentOS
- **playbook_universal.yml** - расширенная поддержка всех дистрибутивов  
- **playbook_no_jmespath.yml** - если возникают проблемы с зависимостями

Все файлы автоматически получают последнюю версию docker-compose через GitHub API без дополнительных зависимостей.

## Поддерживаемые операционные системы

### ✅ Полностью поддерживаемые:
- **Ubuntu** (18.04, 20.04, 22.04, 24.04)
- **Debian** (bullseye, bookworm, sid)
- **CentOS** (7, 8, Stream)
- **Rocky Linux** (8, 9)
- **AlmaLinux** (8, 9)
- **RHEL** (7, 8, 9)
- **Fedora** (последние версии)

### 📁 Доступные playbook файлы:
- `playbook.yml` - основной playbook с поддержкой Ubuntu, Debian и CentOS
- `playbook_universal.yml` - универсальный playbook с расширенной поддержкой дистрибутивов
- `playbook_alternative.yml` - альтернативные методы установки

## Быстрый старт

### Для Debian/Ubuntu/CentOS:
```bash
ansible-playbook -i inventory playbook.yml
```

### Для всех поддерживаемых дистрибутивов:
```bash
ansible-playbook -i inventory playbook_universal.yml
```

## Особенности для разных ОС

### Debian
- Использует официальный репозиторий Docker для Debian
- Поддерживает кодовые имена: bullseye, bookworm, sid
- Автоматически определяет архитектуру системы

### Ubuntu  
- Использует официальный репозиторий Docker для Ubuntu
- Поддерживает LTS и промежуточные версии
- Совместим с всеми поддерживаемыми архитектурами

### CentOS/RHEL/Rocky/Alma
- Использует официальный репозиторий Docker для CentOS
- Поддерживает YUM и DNF менеджеры пакетов
- Автоматическая настройка репозиториев

## Решение проблем

### Ошибка: "You need to install jmespath prior to running json_query filter"

**Проблема:** Отсутствует библиотека jmespath на control node.

**Решение 1 - Установить jmespath:**
```bash
pip3 install jmespath
```

**Решение 2 - Использовать playbook без jmespath:**
```bash
# Используйте файлы без lookup фильтров
ansible-playbook -i inventory playbook_no_jmespath.yml
```

**Что изменилось:**
- Заменили `lookup('url', '...') | json_query('tag_name')` 
- На `uri` модуль + `{{ register_var.json.tag_name }}`

### Ошибка: "404 Not Found" для Ubuntu репозитория на Debian

**Проблема:** Playbook использовал Ubuntu репозиторий для всех Debian-based систем.

**Решение:** Разделили логику для Ubuntu и Debian:
- Ubuntu: `https://download.docker.com/linux/ubuntu/`
- Debian: `https://download.docker.com/linux/debian/`

### Ошибка: "NO_PUBKEY 7EA0A9C3F273FCD8" (GPG ключ)

**Проблема:** GPG ключ Docker не был добавлен корректно или репозиторий поврежден.

**Решение 1 - Очистка и переустановка:**
```bash
# Сначала очистить старые установки
ansible-playbook -i inventory clean_docker.yml

# Затем установить заново
ansible-playbook -i inventory playbook.yml
```

**Решение 2 - Ручная очистка на сервере:**
```bash
# На целевом сервере выполнить:
sudo rm -f /etc/apt/sources.list.d/docker.list
sudo rm -f /etc/apt/keyrings/docker.gpg
sudo rm -f /usr/share/keyrings/docker-archive-keyring.gpg
sudo apt update
```

**Что изменилось в новой версии:**
- Разделили команды добавления GPG ключа и репозитория
- Убрали проблемные escape символы в командах  
- Добавили автоматическую очистку старых файлов
- Используем `lsb_release -cs` вместо переменных окружения

### Ошибка: "HTTPSConnection.__init__() got an unexpected keyword argument 'cert_file'"

**Проблема:** Конфликт версий Python SSL библиотек или устаревший Ansible.

**Диагностика:**
```bash
# Проверить версии и SSL поддержку
ansible-playbook diagnose.yml
```

**Решение 1 - Использовать исправленные playbook:**
```bash
# Используйте исправленные версии без uri модуля
ansible-playbook -i ~/ansible/linux/hosts playbook_universal_fixed.yml
# или обновленный оригинальный
ansible-playbook -i ~/ansible/linux/hosts playbook_universal.yml
```

**Решение 2 - Обновить зависимости:**
```bash
# Обновить urllib3 и requests
pip3 install --upgrade urllib3 requests ansible

# Или использовать виртуальную среду
python3 -m venv ansible_env
source ansible_env/bin/activate
pip install ansible
```

**Решение 3 - Обойти проблему:**
- Заменили `uri` модуль на простой `curl` в shell команде
- Добавили фолбэк версию docker-compose  
- Используем `curl` + `grep` + `sed` вместо JSON парсинга

## Требования

### Control Node (машина с Ansible):
- Ansible >= 2.9
- Python 3.6+
- pip3 (для установки jmespath, если нужно)

### Managed Nodes (целевые серверы):
- SSH доступ
- sudo права
- Поддерживаемая ОС (см. список выше)

## Рекомендация

Для вашего случая рекомендую **Вариант 1** (переменная с lookup), так как он:
- Минимально изменяет существующий код
- Прост в понимании и поддержке  
- Автоматически получает последнюю версию
- Использует официальный GitHub API
