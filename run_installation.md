# Как запустить установку Docker + Docker Compose

## ✅ Проблемы решены!

1. **jmespath ошибка** - установлена библиотека + изменен playbook
2. **Debian поддержка** - добавлены правильные репозитории для Debian
3. **Автоматические версии** - последняя версия docker-compose скачивается автоматически
4. **GPG ключи** - исправлена команда добавления GPG ключей + автоочистка
5. **SSL/TLS ошибки** - заменили uri модуль на простой curl

## 🚀 Запуск установки

### Рекомендуемый способ (SSL проблема решена):
```bash
# Сначала очистка (опционально)
ansible-playbook -i ~/ansible/linux/hosts clean_docker.yml

# Затем установка исправленной версией
ansible-playbook -i ~/ansible/linux/hosts playbook_universal.yml -K
```

### Альтернативные варианты:
```bash
# Основной playbook  
ansible-playbook -i ~/ansible/linux/hosts playbook.yml -K

# Дополнительная исправленная версия
ansible-playbook -i ~/ansible/linux/hosts playbook_universal_fixed.yml -K

# Без зависимостей
ansible-playbook -i ~/ansible/linux/hosts playbook_no_jmespath.yml -K
```

## 🔧 Что исправлено в новой версии

### SSL/TLS проблема:
- ❌ **Было**: `uri` модуль с проблемными SSL параметрами
- ✅ **Стало**: `curl` в shell команде с фолбэк версией
- ✅ **Результат**: Автоматическое получение v2.37.0 (последняя версия)

## 🔧 Новые исправления для GPG проблемы

### Что было исправлено:
- ✅ **Разделили команды**: GPG ключ и репозиторий добавляются отдельно
- ✅ **Убрали sudo**: используем `become: yes` 
- ✅ **Исправили escape символы**: больше нет проблем с кавычками
- ✅ **Добавили lsb-release**: используем `$(lsb_release -cs)` вместо переменных
- ✅ **Автоочистка**: удаляем старые ключи и репозитории перед установкой

### Для вашего Debian bookworm:
- ✅ Правильный репозиторий: `https://download.docker.com/linux/debian/`
- ✅ Правильный GPG ключ для Debian
- ✅ Поддержка codename "bookworm"
- ✅ Автоматическая очистка старых конфликтующих файлов

## 🔍 Что изменилось

### Для Debian bookworm:
- ✅ Правильный репозиторий: `https://download.docker.com/linux/debian/`
- ✅ Правильный GPG ключ для Debian
- ✅ Поддержка codename "bookworm"

### Для автоматизации версий:
- ✅ Убрали `lookup` + `json_query` (требует jmespath)
- ✅ Используем `uri` + `register.json.tag_name`
- ✅ Автоматическое получение последней версии

## 📋 Что установится

- **Docker CE** (последняя стабильная версия)
- **Docker Compose Plugin** (через apt/yum)
- **Docker Compose Standalone** (последняя версия с GitHub)
- **Все зависимости** (containerd, buildx, etc.)

## ⚡ После установки

```bash
# Проверка версий
docker --version
docker-compose --version

# Тест работы
docker run hello-world
``` 