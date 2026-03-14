---
tags:
  - github
  - gh
  - cli
  - tutorial
  - cheatsheet
  - git
  - terminal
cssclasses:
  - git-quick
---
[[⚡Git]]
[[📝 GitHub Commits]]
# 🐙 GitHub CLI (`gh`): Полный туториал

## 📋 Содержание
1. [Что такое GitHub CLI?](#что-такое-github-cli)
2. [Установка и настройка](#установка-и-настройка)
3. [Базовые команды](#базовые-команды)
4. [Работа с репозиториями](#работа-с-репозиториями)
5. [Pull Requests](#pull-requests)
6. [Issues](#issues)
7. [Gists](#gists)
8. [Полезные трюки](#полезные-трюки)
9. [Шпаргалка](#шпаргалка)

---

## 1. 🤔 Что такое GitHub CLI?

**GitHub CLI** (`gh`) — это официальная утилита от GitHub, которая позволяет управлять GitHub прямо из терминала. Никаких переключений на браузер!

### Зачем это нужно?
- **🚀 Скорость** — создание репозитория одной командой
- **💻 Не выходя из терминала** — всё в одном месте
- **🤖 Автоматизация** — можно встраивать в скрипты
- **🎯 Удобство** — интерактивный режим с подсказками

---

## 2. 🔧 Установка и настройка

### Установка

#### **macOS** (через Homebrew)
```bash
brew install gh
```

#### **Windows** (через winget)
```bash
winget install --id GitHub.cli
```
Или через scoop:
```bash
scoop install gh
```

#### **Linux** (Ubuntu/Debian)
```bash
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg
sudo chmod go+r /usr/share/keyrings/githubcli-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null
sudo apt update
sudo apt install gh
```

#### Проверка установки
```bash
gh --version
# GitHub CLI version
```

### Авторизация

```bash
# Запустить авторизацию
gh auth login

# Интерактивный процесс:
# 1. Выбрать протокол: GitHub.com или GitHub Enterprise
# 2. Выбрать способ: HTTPS или SSH (рекомендуется SSH)
# 3. Подтвердить код в браузере
# 4. Готово!

# Проверить авторизацию
gh auth status

# Посмотреть аккаунт
gh api user
```

---

## 3. 🎯 Базовые команды

### Справка
```bash
# Общая справка
gh help

# Справка по конкретной команде
gh repo --help
gh pr --help
gh issue --help

# Посмотреть версию
gh --version
```

### Конфигурация
```bash
# Посмотреть настройки
gh config list

# Установить редактор
gh config set editor "code --wait"

# Установить браузер
gh config set browser "firefox"
```

---

## 4. 📁 Работа с репозиториями (самое важное!)

### Создание репозитория

#### Интерактивный режим (для новичков)
```bash
gh repo create
# Ответишь на вопросы:
# 1. Что создать? (с нуля / из существующей папки)
# 2. Имя репозитория
# 3. Описание
# 4. Публичный/приватный
# 5. Добавить README?
# 6. Добавить .gitignore?
# 7. Добавить лицензию?
# 8. Клонировать локально?
```

#### Быстрые команды (для профи)

```bash
# 🔥 САМАЯ ЧАСТАЯ КОМАНДА:
# Создать публичный репозиторий из текущей папки
gh repo create my-project --public --source=. --remote=origin --push

# Разберем по флагам:
# --public       # публичный репозиторий
# --source=.     # использовать текущую папку
# --remote=origin # назвать удаленный репозиторий origin
# --push         # запушить существующие коммиты
```

#### Все варианты создания:

```bash
# Создать пустой репозиторий и склонировать
gh repo create my-project --public --clone

# Создать приватный репозиторий
gh repo create my-private --private

# Создать в организации
gh repo create my-org/my-project --public

# Создать с README и .gitignore для Python
gh repo create my-project --public --add-readme --gitignore Python

# Создать и сразу добавить описание
gh repo create my-project --public --description "Мой крутой проект"
```

### Просмотр репозиториев

```bash
# Список твоих репозиториев
gh repo list

# Список с фильтром
gh repo list --limit 50
gh repo list --language python

# Посмотреть информацию о репозитории
gh repo view
gh repo view owner/repo

# Открыть в браузере
gh repo view --web
gh repo view owner/repo --web
```

### Клонирование
```bash
# Склонировать репозиторий
gh repo clone owner/repo

# Склонировать и перейти в папку
gh repo clone owner/repo && cd repo
```

### Форки
```bash
# Создать форк
gh repo fork owner/repo

# Создать форк и клонировать
gh repo fork owner/repo --clone

# Создать форк и добавить upstream
gh repo fork owner/repo --remote
```

---

## 5. 🔀 Pull Requests

### Создание PR

```bash
# Создать PR (интерактивно)
gh pr create

# Создать PR с заголовком и описанием
gh pr create --title "Добавил новую фичу" --body "Описание изменений"

# Создать PR в draft режиме
gh pr create --draft

# Создать PR и сразу назначить ревьюера
gh pr create --reviewer username

# Создать PR и добавить метки
gh pr create --label "enhancement" --label "bug"
```

### Просмотр PR

```bash
# Список PR
gh pr list
gh pr list --state closed
gh pr list --assignee @me
gh pr list --label bug

# Посмотреть конкретный PR
gh pr view 123
gh pr view --web

# Посмотреть текущий PR (если в ветке)
gh pr view
```

### Работа с PR

```bash
# Переключиться на ветку PR
gh pr checkout 123

# Смержить PR
gh pr merge 123
gh pr merge 123 --merge     # обычный merge
gh pr merge 123 --squash    # squash
gh pr merge 123 --rebase     # rebase

# Закрыть PR без мержа
gh pr close 123

# Добавить комментарий
gh pr comment 123 --body "Отличная работа!"
```

---

## 6. 🎫 Issues

### Создание issues

```bash
# Создать issue (интерактивно)
gh issue create

# Создать с заголовком и описанием
gh issue create --title "Баг в интерфейсе" --body "Описание проблемы"

# Добавить метки и назначить
gh issue create --label bug --assignee username
```

### Просмотр issues

```bash
# Список issues
gh issue list
gh issue list --state closed
gh issue list --assignee @me
gh issue list --label bug

# Посмотреть конкретный issue
gh issue view 123
gh issue view --web
```

### Работа с issues

```bash
# Закрыть issue
gh issue close 123

# Открыть заново
gh issue reopen 123

# Добавить комментарий
gh issue comment 123 --body "Исправлено в последнем коммите"
```

---

## 7. 📝 Gists

### Создание gist'ов

```bash
# Создать публичный gist из файла
gh gist create script.py

# Создать приватный gist
gh gist create script.py --private

# Создать gist с описанием
gh gist create script.py --desc "Мой скрипт для парсинга"

# Создать gist из нескольких файлов
gh gist create script.py utils.py config.json
```

### Просмотр gist'ов

```bash
# Список твоих gist'ов
gh gist list

# Посмотреть gist
gh gist view 123abc
gh gist view --web 123abc

# Клонировать gist
gh gist clone 123abc
```

### Редактирование gist'ов

```bash
# Редактировать gist
gh gist edit 123abc script.py

# Удалить gist
gh gist delete 123abc
```

---

## 8. 🎯 Полезные трюки

### Алиасы (свои команды)

```bash
# Создать алиас для частой команды
gh alias set prs 'pr list --assignee @me'
# Теперь можно: gh prs

# Создать алиас с аргументами
gh alias set mypr 'pr list --search "$1"'
# Использование: gh mypr "bug"

# Посмотреть все алиасы
gh alias list
```

### Работа с Actions

```bash
# Посмотреть запуски Actions
gh run list

# Посмотреть логи конкретного запуска
gh run view 123

# Перезапустить
gh run rerun 123
```

### Работа с репозиториями (продвинутое)

```bash
# Посмотреть зависимости
gh repo dependency-list

# Посмотреть статистику
gh repo view --json name,description,updatedAt,stargazerCount

# Архивировать репозиторий
gh repo archive

# Удалить репозиторий (осторожно!)
gh repo delete
```

### Поиск

```bash
# Поиск репозиториев
gh search repos "machine learning" --language=python

# Поиск issues
gh search issues "bug" --assignee=@me

# Поиск кода
gh search code "function main" --language=python
```

---

## 9. 🚀 Интеграция с git

### GitHub CLI + git = идеальная пара

```bash
# Сценарий: начать новый проект
mkdir my-awesome-project
cd my-awesome-project
git init
echo "# My Project" > README.md
git add .
git commit -m "Initial commit"

# Создать репозиторий на GitHub и запушить одной командой!
gh repo create my-awesome-project --public --source=. --remote=origin --push
```

### Сценарий: форк и PR

```bash
# Форкнуть чужой репозиторий
gh repo fork owner/repo --clone
cd repo

# Создать ветку для фичи
git checkout -b my-feature

# Внести изменения
echo "print('hello')" > script.py
git add .
git commit -m "Add my feature"

# Запушить и создать PR одной командой!
git push -u origin my-feature
gh pr create --title "Моя фича" --body "Описание"
```

---

## 10. 📊 Шпаргалка (быстрый доступ)

### Самые частые команды

| Команда | Что делает |
|---------|------------|
| `gh repo create` | Создать репозиторий |
| `gh repo clone user/repo` | Склонировать репозиторий |
| `gh repo list` | Список репозиториев |
| `gh repo view --web` | Открыть в браузере |
| `gh pr create` | Создать Pull Request |
| `gh pr list` | Список PR |
| `gh pr checkout 123` | Переключиться на PR |
| `gh issue list` | Список issues |
| `gh gist create file.py` | Создать gist |
| `gh auth login` | Авторизоваться |

### Полезные флаги

```bash
# Для repo create
--public           # публичный репозиторий
--private          # приватный репозиторий
--source=.         # из текущей папки
--push             # запушить после создания
--clone            # склонировать локально
--remote=origin    # имя remote
--add-readme       # добавить README
--gitignore=Python # добавить .gitignore

# Для pr create
--title "T"       # заголовок
--body "B"        # описание
--draft           # черновик
--reviewer user   # назначить ревьюера
--label bug       # добавить метку
```

### Комбинации с git

```bash
# Старт нового проекта
git init && echo "# Readme" > README.md && git add . && git commit -m "init" && gh repo create --public --source=. --remote=origin --push

# Форк и PR
gh repo fork user/repo --clone && cd repo && git checkout -b fix && echo "fix" >> file && git add . && git commit -m "fix" && git push -u origin fix && gh pr create --title "Fix" --body "Description"
```

---

## 🎓 Заключение

**GitHub CLI (`gh`)** — это мощный инструмент, который ускоряет работу с GitHub в разы. После того как запомнишь основные команды, ты будешь создавать репозитории и PR за секунды, не отвлекаясь на браузер.

### Три команды, которые нужно запомнить:
1. `gh repo create` — создать репозиторий
2. `gh pr create` — создать Pull Request
3. `gh issue list` — посмотреть задачи


