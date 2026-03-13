---
tags:
  - python
  - cheatsheet
  - quick-reference
  - summary
cssclasses:
  - git-quick
---

# ⚡_CheatSheet_Быстрый_старт

## 🚀 САМОЕ ВАЖНОЕ ЗА 5 МИНУТ

### Переменные и типы
```python
name = "Анна"        # str
age = 30             # int
height = 1.75        # float
is_student = True    # bool (с большой буквы!)
nothing = None       # null

# Множественное присваивание
x, y, z = 1, 2, 3
a = b = c = 0
```

### Строки (f-strings — мастхэв!)
```python
name = "Анна"
age = 30
print(f"Меня зовут {name}, мне {age} лет")  # f-строка!
print(f"Через 5 лет: {age + 5}")

# Срезы
s = "Python"
s[0]      # 'P'
s[-1]     # 'n'
s[1:4]    # 'yth'
s[::-1]   # 'nohtyP'
```

### Списки (главная структура)
```python
lst = [1, 2, 3]
lst.append(4)        # [1,2,3,4]
lst.pop()            # удалить последний
lst[0]               # первый элемент
lst[-1]              # последний
len(lst)             # длина

# Генератор списка (ОЧЕНЬ ВАЖНО!)
squares = [x**2 for x in range(10)]  # [0,1,4,9,...]
evens = [x for x in range(10) if x % 2 == 0]
```

### Словари
```python
d = {"name": "Анна", "age": 30}
d["name"]            # "Анна"
d.get("phone", "нет") # безопасное чтение
d["job"] = "Инженер"  # добавление
for k, v in d.items():
    print(f"{k}: {v}")
```

### Условия и циклы
```python
# if/elif/else (двоеточие и отступы!)
if x > 5 and y < 10:
    print("OK")
elif x == 5:
    print("x=5")
else:
    print("else")

# for (всегда по коллекции!)
for i in range(5):           # 0..4
    print(i)

for idx, val in enumerate(lst):
    print(idx, val)
```

### Функции
```python
def add(a, b):
    """Документация"""
    return a + b

# Лямбда
square = lambda x: x**2
```

### Файлы (всегда с with!)
```python
with open("file.txt", "r") as f:
    content = f.read()

with open("out.txt", "w") as f:
    f.write("Привет")
```

### Обработка ошибок
```python
try:
    x = int(input())
except ValueError:
    print("Это не число!")
except Exception as e:
    print(f"Ошибка: {e}")
finally:
    print("Всегда выполняется")
```

---

## 📚 ШПАРГАЛКА ПО РАЗДЕЛАМ

### [[01_🔤_Синтаксис_и_типы]]
| Тип | Пример | Проверка |
|-----|--------|----------|
| int | `x = 42` | `isinstance(x, int)` |
| float | `y = 3.14` | `isinstance(y, float)` |
| str | `s = "hello"` | `isinstance(s, str)` |
| bool | `b = True` | `isinstance(b, bool)` |
| None | `n = None` | `n is None` |

### [[02_📚_Структуры_данных]]
| Тип | Изменяемый | Упорядоч. | Индексация | Создание |
|-----|------------|-----------|------------|----------|
| list | ✅ | ✅ | ✅ | `[1,2,3]` |
| tuple | ❌ | ✅ | ✅ | `(1,2,3)` |
| dict | ✅ | ✅ (3.7+) | по ключу | `{"a":1}` |
| set | ✅ | ❌ | ❌ | `{1,2,3}` |

**Главные методы:**
- `len()`, `in`, `for x in collection:`
- list: `append()`, `pop()`, `sort()`
- dict: `keys()`, `values()`, `items()`, `get()`
- set: `add()`, `remove()`, `| & - ^`

### [[03_⚙️_Функции]]
```python
def func(pos1, pos2, /, pos_or_kwd, *, kwd1, kwd2="default"):
    """Позиционные до /, именованные после *"""
    return something

# *args — кортеж позиционных
# **kwargs — словарь именованных
```

### [[04_🏛️_ООП_и_классы]]
```python
class MyClass:
    class_attr = 0
    
    def __init__(self, name):
        self.name = name  # атрибут объекта
    
    def method(self):
        return f"{self.name}"
    
    @property
    def prop(self):
        return self._value
```

### [[05_📁_Работа_с_файлами]]
```python
from pathlib import Path

p = Path("folder") / "file.txt"
p.read_text()
p.write_text("data")
p.exists()
p.is_file()
```

### [[06_⚠️_Обработка_ошибок]]
```python
try:
    risky()
except SpecificError as e:
    print(e)
except Exception as e:
    print(f"Неизвестно: {e}")
else:
    print("OK")
finally:
    print("Cleanup")
```

### [[07_📦_Модули_и_пакеты]]
```python
# Создание модуля
if __name__ == "__main__":
    # только при прямом запуске
    main()

# pip
pip install requests
pip freeze > requirements.txt
```

### [[08_📖_Стандартная_библиотека]]
```python
# Чаще всего:
import math, random, datetime, json, re
from collections import Counter, defaultdict
from itertools import chain, islice
from functools import wraps, partial
```

### [[09_♻️_Итераторы_и_генераторы]]
```python
def gen():
    for i in range(5):
        yield i  # лениво!

gen_exp = (x**2 for x in range(10))  # генераторное выражение
```

### [[10_🎨_Декораторы_и_продвинутое]]
```python
from functools import wraps

def decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        # до вызова
        result = func(*args, **kwargs)
        # после вызова
        return result
    return wrapper
```

---

## 🎯 ТОП-10 ФУНКЦИЙ, КОТОРЫЕ НУЖНО ЗНАТЬ

1. **`print()`** — вывод
2. **`len()`** — длина
3. **`type()`** — тип объекта
4. **`isinstance()`** — проверка типа
5. **`range()`** — генератор последовательности
6. **`enumerate()`** — индексы при итерации
7. **`zip()`** — объединение последовательностей
8. **`open()`** — работа с файлами
9. **`sorted()`** — сортировка
10. **`sum()`, `min()`, `max()`** — аггрегация

---

## 🔧 ПОЛЕЗНЫЕ ПАТТЕРНЫ

### Распаковка
```python
a, b, *rest = [1,2,3,4,5]  # a=1, b=2, rest=[3,4,5]
first, *mid, last = [1,2,3,4]  # first=1, mid=[2,3], last=4

def func(*args, **kwargs):
    pass

args = (1,2)
kwargs = {"a":3, "b":4}
func(*args, **kwargs)
```

### Тернарный оператор
```python
x = a if condition else b
```

### Проверка на None
```python
if x is None:
if x is not None:
```

### Безопасное чтение из словаря
```python
value = d.get("key", default)
```

### Обмен значений
```python
a, b = b, a
```

### Множественные сравнения
```python
if 1 < x < 10:
```

---

## 📋 ЧЕК-ЛИСТ НОВИЧКА

- [ ] Всегда использую `with` для файлов
- [ ] Не пишу `from module import *`
- [ ] Использую f-строки вместо конкатенации
- [ ] Для словарей использую `.get()`
- [ ] Понимаю разницу между `==` и `is`
- [ ] Знаю, что `{}` — пустой словарь, `set()` — пустое множество
- [ ] Использую генераторы списков вместо циклов где уместно
- [ ] Пишу `if __name__ == "__main__":` в скриптах
- [ ] Использую `enumerate` вместо `range(len())`
- [ ] Не использую изменяемые значения по умолчанию

---

## 📖 КАК БЫСТРО НАЙТИ НУЖНОЕ

| Если нужно... | Смотреть в... |
|----------------|---------------|
| Основы синтаксиса | [[01_🔤_Синтаксис_и_типы]] |
| Списки, словари | [[02_📚_Структуры_данных]] |
| Функции, лямбды | [[03_⚙️_Функции]] |
| Классы, ООП | [[04_🏛️_ООП_и_классы]] |
| Файлы, pathlib | [[05_📁_Работа_с_файлами]] |
| try/except | [[06_⚠️_Обработка_ошибок]] |
| import, pip | [[07_📦_Модули_и_пакеты]] |
| math, random, datetime | [[08_📖_Стандартная_библиотека]] |
| yield, генераторы | [[09_♻️_Итераторы_и_генераторы]] |
| @decorator | [[10_🎨_Декораторы_и_продвинутое]] |

---

## 🚨 ТИПИЧНЫЕ ОШИБКИ НОВИЧКОВ

```python
# ❌ Плохо
def func(lst=[]):  # список создается один раз!

# ✅ Хорошо
def func(lst=None):
    if lst is None:
        lst = []

# ❌ Плохо
if type(x) == int:  # не учитывает наследование

# ✅ Хорошо
if isinstance(x, int):

# ❌ Плохо
file = open("f.txt")
data = file.read()
# забыли close()

# ✅ Хорошо
with open("f.txt") as f:
    data = f.read()
```

---

## 🎓 СОВЕТЫ ДЛЯ C++ ПРОГРАММИСТА

| Вместо... | В Python... |
|-----------|-------------|
| `{}` для блоков | Отступы (4 пробела) |
| `&&`, `\|\|`, `!` | `and`, `or`, `not` |
| `int x = 5;` | `x = 5` |
| `for(int i=0; i<n; i++)` | `for i in range(n):` |
| `vector<int> v;` | `v = []` |
| `map<string, int> m;` | `m = {}` |
| `NULL` | `None` |
| `// комментарий` | `# комментарий` |
| `cout << x << endl;` | `print(x)` |
| `if (cond) { ... }` | `if cond:` |

---
