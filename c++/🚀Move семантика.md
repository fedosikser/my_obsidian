---
tags: [cpp, move-semantics, rvalue, performance, cheatsheet]
cssclasses: [cpp-move, performance-guide, git-quick]
---

# 🚀 C++ шпаргалка: move семантика

> [!tip] Главная идея
> **Move = кража ресурсов** вместо их копирования.  
> Вместо `clone` → просто `перемещаем` указатель. **Быстро и дешево!**

---

## 📊 Копирование vs Перемещение

| Операция | Что делает | Сложность | Когда использовать |
|----------|------------|-----------|-------------------|
| **Copy** | создаёт полную копию | O(n) | когда нужен независимый объект |
| **Move** | крадёт внутренности | O(1) | когда старый объект больше не нужен |

```cpp
std::vector<int> a(1000000, 42);
std::vector<int> b = a;        // ❌ Копирование: 4MB выделено+скопировано
std::vector<int> c = std::move(a);  // ✅ Перемещение: просто обмен указателями
// a теперь пуст, c владеет данными
```

---

## 🎯 lvalue vs rvalue (ключ к пониманию)

| Термин | Что это | Пример |
|--------|---------|--------|
| **lvalue** | имеет имя, есть адрес | `int x = 5;` (`x` — lvalue) |
| **rvalue** | временное, нет имени | `5`, `x + y`, `std::move(x)` |

```cpp
void foo(int& x)  { /* lvalue reference */ }
void foo(int&& x) { /* rvalue reference */ }

int a = 10;
foo(a);           // вызывает foo(int&)  — a lvalue
foo(20);          // вызывает foo(int&&) — 20 rvalue
foo(std::move(a)); // вызывает foo(int&&) — std::move превращает lvalue в rvalue
```

---

## 🏗️ Конструктор перемещения

```cpp
class Buffer {
private:
    int* data;
    size_t size;
    
public:
    // ❌ Копирующий конструктор (дорогой)
    Buffer(const Buffer& other) 
        : size(other.size) {
        data = new int[size];
        std::copy(other.data, other.data + size, data);
    }
    
    // ✅ Перемещающий конструктор (дешёвый)
    Buffer(Buffer&& other) noexcept 
        : data(other.data), size(other.size) {
        other.data = nullptr;  // обнуляем источник!
        other.size = 0;
    }
    
    ~Buffer() {
        delete[] data;
    }
};
```

> [!important] noexcept
> Помечайте перемещающие операции `noexcept`. Это позволяет стандартной библиотеке использовать move (например, в `std::vector::push_back`).

---

## 📝 Правило пяти

Если вы определяете одну из этих функций, определите все пять:

```cpp
class MyClass {
public:
    // 1. Деструктор
    ~MyClass() = default;
    
    // 2. Копирующий конструктор
    MyClass(const MyClass&) = default;
    
    // 3. Копирующий оператор присваивания
    MyClass& operator=(const MyClass&) = default;
    
    // 4. Перемещающий конструктор
    MyClass(MyClass&&) noexcept = default;
    
    // 5. Перемещающий оператор присваивания
    MyClass& operator=(MyClass&&) noexcept = default;
};
```

---

## 🔄 Move-оператор присваивания

```cpp
class Buffer {
public:
    // Перемещающее присваивание
    Buffer& operator=(Buffer&& other) noexcept {
        if (this != &other) {           // проверка на самоприсваивание
            delete[] data;               // освобождаем старые ресурсы
            
            data = other.data;           // крадём
            size = other.size;
            
            other.data = nullptr;        // обнуляем источник
            other.size = 0;
        }
        return *this;
    }
};
```

### Идиома copy-and-swap (универсальный вариант)

```cpp
class Buffer {
public:
    friend void swap(Buffer& a, Buffer& b) noexcept {
        using std::swap;
        swap(a.data, b.data);
        swap(a.size, b.size);
    }
    
    // Один оператор на всё: copy AND move
    Buffer& operator=(Buffer other) {  // передача по значению!
        swap(*this, other);
        return *this;
    }
};
// other копируется (если lvalue) или перемещается (если rvalue)
// затем swap обменивает содержимое
```

---

## 🎲 Где move происходит автоматически

```cpp
// 1. Временные объекты (rvalue)
std::string a = "hello";
std::string b = a + " world";  // временный результат перемещается

// 2. return из функции (NRVO/RVO)
std::vector<int> createVector() {
    std::vector<int> v(1000);
    return v;  // автоматическое перемещение (или copy elision)
}

// 3. Когда явно используем std::move
std::vector<int> v1(1000);
std::vector<int> v2 = std::move(v1);  // явное перемещение

// 4. В std::vector при реаллокации
std::vector<MyClass> vec;
vec.push_back(MyClass());  // временный -> move
```

---

## 🚫 Когда НЕЛЬЗЯ использовать move

```cpp
class Widget {
    int id;
    std::string name;
    
public:
    // ❌ ОШИБКА: move после использования
    void printAndMove() {
        std::cout << name;           // используем name
        return std::move(name);      // ❌ name в неопределённом состоянии!
    }
    
    // ✅ ПРАВИЛЬНО: move только когда объект больше не нужен
    std::string extractName() {
        return std::move(name);      // ✅ мы больше не используем name
    }
    
    // ❌ ОШИБКА: move из const объекта
    void bad(const std::string& s) {
        auto copy = std::move(s);    // ❌ const -> копирование, не move!
    }
};
```

---

## 📦 std::move — это просто каст

```cpp
// Реализация std::move (упрощённо)
template<typename T>
typename std::remove_reference<T>::type&& move(T&& t) {
    return static_cast<typename std::remove_reference<T>::type&&>(t);
}

// То есть:
std::move(x)  ==  static_cast<T&&>(x)
```

> [!note] std::move ничего не перемещает  
> Он просто превращает lvalue в rvalue. Реальное перемещение делают конструктор или оператор присваивания.

---

## 🎯 Практические правила

| Ситуация | Решение |
|----------|---------|
| **Функция возвращает большой объект** | просто `return obj;` (copy elision сделает своё дело) |
| **Хотим передать владение внутрь функции** | `void sink(std::vector<int> v)` (передача по значению + move на месте вызова) |
| **Параметр только читаем** | `const T&` (не нужно move) |
| **Параметр забираем (sink-параметр)** | `T` (по значению) или `T&&` |
| **Вектор уникальных указателей** | `std::vector<std::unique_ptr<T>>` — автоматический move |
| **Забираем поле объекта в локальную переменную** | `auto local = std::move(obj.field);` |

---

## 🔧 Примеры использования

```cpp
// 1. Перемещение в контейнер
std::vector<std::string> vec;
std::string s = "hello";
vec.push_back(std::move(s));  // s теперь пуст
// vec[0] == "hello"

// 2. Перемещение unique_ptr
std::unique_ptr<int> p1 = std::make_unique<int>(42);
std::unique_ptr<int> p2 = std::move(p1);  // p1 = nullptr
// p2 владеет объектом

// 3. Move-семантика в лямбдах
auto ptr = std::make_unique<int>(10);
auto lambda = [capture = std::move(ptr)]() {
    return *capture;  // lambda владеет unique_ptr
};

// 4. Emplace vs push
std::vector<MyClass> vec;
vec.push_back(MyClass(1, 2));     // создаём временный + move
vec.emplace_back(1, 2);            // создаём прямо в векторе (лучше)
```

---

## 📊 Move-семантика для стандартных типов

| Тип | Результат move |
|-----|---------------|
| `std::vector<T>` | O(1), источник становится пустым |
| `std::string` | O(1), источник становится пустым |
| `std::unique_ptr<T>` | O(1), источник = `nullptr` |
| `std::shared_ptr<T>` | O(1), счётчик не меняется |
| `int, double, char` | копирование (нет выгоды) |
| `std::array<T, N>` | O(N), копирование (элементы перемещаются) |

---

## ⚠️ Частые ошибки

```cpp
// ❌ 1. Лишний std::move при return
std::vector<int> bad() {
    std::vector<int> v;
    return std::move(v);  // НЕ НАДО! мешает copy elision
}

// ✅
std::vector<int> good() {
    std::vector<int> v;
    return v;  // RVO или move автоматически
}

// ❌ 2. Использование перемещённого объекта
std::string s = "hello";
std::string t = std::move(s);
std::cout << s;  // ❌ undefined behavior (s в неопределённом состоянии)

// ✅
s = "new value";  // присвоить новое значение можно

// ❌ 3. std::move от const
const std::string cs = "hello";
auto cs2 = std::move(cs);  // ❌ копирование, не move
```

---

## 🎓 Perfect forwarding (продвинутый уровень)

```cpp
// Передаём аргументы дальше с сохранением lvalue/rvalue

template<typename T, typename... Args>
std::unique_ptr<T> make_unique(Args&&... args) {
    return std::unique_ptr<T>(new T(std::forward<Args>(args)...));
}

// std::forward сохраняет категорию:
// - lvalue → lvalue
// - rvalue → rvalue
```

---

## 📝 Быстрая памятка

```cpp
// Когда использовать std::move:
std::vector<int> v1 = {1, 2, 3};
std::vector<int> v2 = std::move(v1);  // ✅ v1 больше не нужен

// Когда НЕ использовать:
const std::string& getName() { return name; }
auto s = std::move(getName());  // ❌ возвращает const rvalue -> копирование

// Move полезен для:
// - умных указателей
// - контейнеров (vector, string, map)
// - любых типов с дешёвым перемещением
```

---

## 📎 Связанные заметки

 [[🔹Умные указатели]]
 [[🛡️RAII принцип]]
 [[🧵Многопоточность]]
[[⚰️Декомпозиция]]