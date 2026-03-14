---
tags: [git, cheatsheet, quick-reference, qwen, lmstudio, terminal, macos, aliases]
cssclasses:
  - git-quick
---
code  ~/.zshrc
## 🚀 **Основные команды**

| Команда | Что делает |
|---------|------------|
| `0` | Запрос **с контекстом** + поток |
| `00` | Запрос **без контекста** + поток |
| `qchat` | Интерактивный чат |
| `qcheck` | Проверка сервера |

## 🧹 **Управление контекстом**

| Команда | Что делает |
|---------|------------|
| `0c` | 🧹 Очистить историю |
| `qshow` | 📖 Показать историю |

## 📋 **Буфер обмена**

| Команда | Что делает |
|---------|------------|
| `0p` | Из буфера **с контекстом** |
| `00p` | Из буфера **без контекста** |

## 🎮 **LM Studio (`lms`)**

| Команда             | Что делает              |
| ------------------- | ----------------------- |
| `lms server start`  | Запустить сервер        |
| `lms server stop`   | Остановить сервер       |
| `lms server status` | Статус сервера          |
| `lms ls`            | Список моделей          |
| `lms ps`            | Модели в памяти         |
| `lms load <модель>` | Загрузить модель        |
| `lms unload --all`  | Выгрузить все           |
| `lms log stream`    | Логи в реальном времени |

## 💡 **Примеры**

```bash
0 как отменить последний коммит     # с контекстом
00 привет                           # быстрый запрос
0p                                  # спросить из буфера
0c                                  # начать новый диалог
lms server status                   # проверка сервера
```

---




КОД СКОПИРОВАТЬ 



# ----------------------------------------

# Qwen Coder для LM Studio (С ВАШЕЙ СХЕМОЙ)

# ----------------------------------------

  

# Переменная для хранения истории

QCHAT_HISTORY=""

  

# Проверка сервера

qcheck() {

if curl -s http://localhost:1234/v1/models > /dev/null; then

echo "✅ Сервер LM Studio работает!"

echo "📋 Доступные модели:"

curl -s http://localhost:1234/v1/models | grep -o '"id":"[^"]*"' | sed 's/"id":"//;s/"$//' || echo " Не удалось получить список моделей"

else

echo "❌ Сервер LM Studio не отвечает!"

echo " Запустите LM Studio, включите сервер на вкладке Developer"

fi

}

  

# Функция без контекста (всегда потоковый вывод)

qwen-simple-stream() {

local prompt="$*"

if [[ -z "$prompt" ]]; then

echo "📝 Введите запрос:"

read -r prompt

fi

echo "🤔 Ответ:"

curl -N -s http://localhost:1234/v1/chat/completions \

-H "Content-Type: application/json" \

-d "{

\"model\": \"qwen2.5-coder-7b-instruct\",

\"messages\": [{\"role\": \"user\", \"content\": \"$prompt\"}],

\"temperature\": 0.7,

\"stream\": true

}" | while read -r line; do

if [[ "$line" =~ ^data:\ (.*) ]]; then

content=$(echo "$line" | sed 's/^data: //' | jq -r '.choices[0].delta.content // ""' 2>/dev/null)

echo -n "$content"

fi

done

echo ""

}

  

# Функция с контекстом (всегда потоковый вывод)

qwen-context-stream() {

local prompt="$*"

if [[ -z "$prompt" ]]; then

echo "📝 Введите запрос:"

read -r prompt

fi

# Добавляем новое сообщение в историю

if [[ -n "$QCHAT_HISTORY" ]]; then

local full_context="${QCHAT_HISTORY}\nUser: ${prompt}"

else

local full_context="User: ${prompt}"

fi

echo "🤔 Ответ:"

# Экранируем для JSON

local escaped_context=$(echo "$full_context" | sed 's/"/\\"/g' | awk '{printf "%s\\n", $0}' | sed 's/\\n$//')

# Отправляем запрос с потоковым выводом

local response=$(curl -N -s http://localhost:1234/v1/chat/completions \

-H "Content-Type: application/json" \

-d "{

\"model\": \"qwen2.5-coder-7b-instruct\",

\"messages\": [

{\"role\": \"system\", \"content\": \"Ты полезный ассистент. Отвечай кратко и по делу.\"},

{\"role\": \"user\", \"content\": \"$escaped_context\"}

],

\"temperature\": 0.7,

\"stream\": true

}")

# Собираем полный ответ для сохранения в историю

local full_answer=""

while read -r line; do

if [[ "$line" =~ ^data:\ (.*) ]]; then

content=$(echo "$line" | sed 's/^data: //' | jq -r '.choices[0].delta.content // ""' 2>/dev/null)

echo -n "$content"

full_answer+="$content"

fi

done <<< "$response"

echo ""

# Сохраняем в историю

QCHAT_HISTORY="${full_context}\nAssistant: ${full_answer}"

QCHAT_HISTORY=$(echo "$QCHAT_HISTORY" | tail -20)

}

  

# Интерактивный чат (тоже с потоковым выводом)

qchat() {

python3 -c "

import sys

import json

import subprocess

  

history = []

print('🤖 Чат с Qwen (Ctrl+C для выхода)')

  

while True:

try:

prompt = input('\n👤 Вы: ')

if prompt.lower() in ['exit', 'quit']:

break

history.append({'role': 'user', 'content': prompt})

messages = [{'role': 'system', 'content': 'Ты полезный ассистент.'}] + history[-10:]

cmd = ['curl', '-N', '-s', 'http://localhost:1234/v1/chat/completions',

'-H', 'Content-Type: application/json',

'-d', json.dumps({

'model': 'qwen2.5-coder-7b-instruct',

'messages': messages,

'temperature': 0.7,

'stream': True

})]

process = subprocess.Popen(cmd, stdout=subprocess.PIPE, text=True)

answer = ''

print('🤖 Qwen: ', end='', flush=True)

for line in process.stdout:

if line.startswith('data: '):

try:

data = json.loads(line[6:])

content = data['choices'][0]['delta'].get('content', '')

print(content, end='', flush=True)

answer += content

except:

pass

print()

history.append({'role': 'assistant', 'content': answer})

except KeyboardInterrupt:

print('\n👋 Пока!')

break

except Exception as e:

print(f'❌ Ошибка: {e}')

" "$@"

}

  

# ----------------------------------------

# АЛИАСЫ (ПО ВАШЕЙ СХЕМЕ)

# ----------------------------------------

  

# Основные

alias 00='qwen-simple-stream' # Без контекста + потоковый вывод

alias 0='qwen-context-stream' # С контекстом + потоковый вывод

alias qchat='qchat' # Интерактивный чат

alias qcheck='qcheck' # Проверка сервера

  

# Управление контекстом

alias 0c='QCHAT_HISTORY="" && echo "🧹 Контекст очищен"' # Очистка контекста для 00

alias qshow='echo -e "📋 Текущая история:\n$QCHAT_HISTORY"' # Показать историю

  

# Работа с буфером обмена

alias 00p='pbpaste | qwen-simple-stream' # Отправить из буфера (без контекста)

alias 0p='pbpaste | qwen-context-stream' # Отправить из буфера (с контекстом)