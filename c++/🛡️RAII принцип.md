---
tags: [cpp, raii, memory-management, resource-management, cheatsheet]
cssclasses: [cpp-raii, resource-guide, git-quick]
---

# 🛡️ C++ шпаргалка: RAII принцип

> [!tip] Что такое RAII?
> **R**esource **A**cquisition **I**s **I**nitialization  
> *(Получение ресурса есть инициализация)*

**Главная идея:** Выделение ресурса происходит в конструкторе, освобождение — в деструкторе. **Ресурс автоматически освободится при выходе из области видимости.**

---

## 🎯 Почему RAII важен?

| Без RAII (ручное управление) | С RAII (автоматическое) |
|------------------------------|------------------------|
| ❌ Забыли `delete` → утечка | ✅ Автоматическое удаление |
| ❌ Исключение → пропуск `delete` | ✅ Безопасно при исключениях |
| ❌ Множество `return` → сложно отследить | ✅ Один деструктор на всё |
| ❌ Код загромождён очисткой | ✅ Чистый, понятный код |

```cpp
// ❌ БЕЗ RAII (опасно!)
void bad() {
    int* ptr = new int[1000];
    if (something_wrong()) return;  // утечка! delete не вызван
    delete[] ptr;  // если дошли...
}

// ✅ С RAII (безопасно!)
void good() {
    std::vector<int> data(1000);  // ресурс выделен в конструкторе
    if (something_wrong()) return;  // вектор сам очистится!
    // data уничтожится автоматически
}
```

---

## 📚 Основные RAII-обёртки в стандартной библиотеке

| Тип ресурса | RAII-обёртка | Что делает деструктор |
|-------------|--------------|----------------------|
| Динамическая память | `std::unique_ptr<T>` | `delete` |
| Динамическая память | `std::shared_ptr<T>` | `delete` (с счётчиком) |
| Массив в куче | `std::vector<T>` | `delete[]` |
| Файловый дескриптор | `std::fstream` | `close()` |
| Мьютекс | `std::lock_guard` | `unlock()` |
| Нить | `std::thread` | `join()` или `detach()` |
| Обобщённый ресурс | кастомный RAII-класс | пользовательский cleanup |

---

## 🏗️ Создаём свой RAII-класс

### Пример 1: Управление файлом

```cpp
class FileRAII {
private:
    FILE* file;
    
public:
    // Конструктор — захватывает ресурс
    FileRAII(const char* filename, const char* mode) 
        : file(fopen(filename, mode)) {
        if (!file) {
            throw std::runtime_error("Cannot open file");
        }
    }
    
    // Деструктор — освобождает ресурс
    ~FileRAII() {
        if (file) {
            fclose(file);  // автоматически закроется
        }
    }
    
    // Запрещаем копирование
    FileRAII(const FileRAII&) = delete;
    FileRAII& operator=(const FileRAII&) = delete;
    
    // Разрешаем перемещение
    FileRAII(FileRAII&& other) noexcept 
        : file(std::exchange(other.file, nullptr)) {}
    
    FileRAII& operator=(FileRAII&& other) noexcept {
        if (this != &other) {
            if (file) fclose(file);
            file = std::exchange(other.file, nullptr);
        }
        return *this;
    }
    
    // Доступ к ресурсу
    FILE* get() const { return file; }
    
    void write(const std::string& data) {
        fprintf(file, "%s", data.c_str());
    }
};

// Использование
void useFile() {
    FileRAII file("test.txt", "w");
    file.write("Hello, RAII!");
    // file автоматически закроется здесь
}
```

### Пример 2: Управление сокетом (WinSock)

```cpp
class SocketRAII {
private:
    SOCKET sock;
    
public:
    SocketRAII() : sock(socket(AF_INET, SOCK_STREAM, 0)) {
        if (sock == INVALID_SOCKET) {
            throw std::runtime_error("Socket creation failed");
        }
    }
    
    ~SocketRAII() {
        if (sock != INVALID_SOCKET) {
            closesocket(sock);  // автоматическое закрытие
        }
    }
    
    SOCKET get() const { return sock; }
    
    // Остальные методы...
};
```

### Пример 3: Универсальный RAII через лямбду

```cpp
template<typename T, typename Deleter>
class GenericRAII {
private:
    T resource;
    Deleter deleter;
    
public:
    GenericRAII(T res, Deleter del) 
        : resource(res), deleter(del) {}
    
    ~GenericRAII() {
        if (resource) {
            deleter(resource);
        }
    }
    
    T get() const { return resource; }
    
    // Запрещаем копирование
    GenericRAII(const GenericRAII&) = delete;
    GenericRAII& operator=(const GenericRAII&) = delete;
    
    // Разрешаем перемещение
    GenericRAII(GenericRAII&& other) noexcept 
        : resource(std::exchange(other.resource, nullptr)), 
          deleter(std::move(other.deleter)) {}
};

// Упрощаем создание
template<typename T, typename Deleter>
auto make_raii(T res, Deleter del) {
    return GenericRAII<T, Deleter>(res, del);
}

// Использование
void useCustomRAII() {
    auto file = make_raii(fopen("test.txt", "r"), fclose);
    auto ptr = make_raii(malloc(1024), free);
    // всё автоматически очистится
}
```

---

## 🔐 RAII для мьютексов

```cpp
#include <mutex>

std::mutex mtx;
int shared_data = 0;

// ❌ Без RAII (опасно!)
void bad_increment() {
    mtx.lock();
    if (some_condition()) {
        return;  // забыли unlock()! deadlock!
    }
    shared_data++;
    mtx.unlock();
}

// ✅ С RAII (безопасно!)
void good_increment() {
    std::lock_guard<std::mutex> lock(mtx);  // автоматический lock/unlock
    if (some_condition()) {
        return;  // lock_guard сам вызовет unlock()
    }
    shared_data++;
}  // unlock() здесь
```

---

## 🎲 RAII и исключения — идеальная пара

```cpp
// Без RAII — кошмар при исключениях
void nightmare() {
    int* arr = new int[1000];
    FILE* f = fopen("data.txt", "r");
    
    try {
        risky_operation();  // может бросить исключение
        delete[] arr;
        fclose(f);
    } catch (...) {
        delete[] arr;  // дублируем очистку!
        fclose(f);
        throw;
    }
}

// С RAII — чистота и безопасность
void beautiful() {
    std::vector<int> arr(1000);      // RAII для памяти
    std::ifstream f("data.txt");     // RAII для файла
    
    risky_operation();  // если исключение — всё само очистится
    
    // деструкторы вызовутся автоматически
}
```

---

## 📊 Правило нуля (Rule of Zero)

> **Если не управляете ресурсами вручную — не пишите конструкторы/деструкторы**

```cpp
// ✅ ПРАВИЛЬНО — полагаемся на RAII-обёртки
class User {
    std::string name;           // string сам управляет памятью
    std::vector<int> scores;    // vector сам управляет
    std::unique_ptr<Logger> logger;  // умный указатель
    
public:
    // Компилятор сгенерирует всё сам
    // Конструкторы, деструктор, move/copy — всё корректно
};

// ❌ НЕПРАВИЛЬНО — зачем изобретать велосипед?
class BadUser {
    char* name;                 // ручное управление!
    int* scores;                // ручное управление!
    Logger* logger;             // ручное управление!
    
public:
    BadUser(const char* n) {
        name = new char[strlen(n)+1];
        strcpy(name, n);
        scores = new int[100];
        logger = new Logger();
    }
    
    ~BadUser() {
        delete[] name;
        delete[] scores;
        delete logger;
    }
    // и ещё куча boilerplate...
};
```

---

## 🎯 Практические правила RAII

| Ресурс | Как управлять RAII |
|--------|-------------------|
| Динамическая память | `std::unique_ptr`, `std::shared_ptr`, `std::vector` |
| Файлы | `std::fstream`, `std::ifstream`, `std::ofstream` |
| Мьютексы | `std::lock_guard`, `std::unique_lock` |
| Сокеты/BMP/OpenGL | написать свой RAII-класс или найти обёртку |
| C-библиотеки (malloc/free) | `std::unique_ptr<T, Deleter>` |
| БД соединения | свой RAII-класс с `connect()`/`disconnect()` |
| Транзакции | RAII для commit/rollback |

---

## 🚀 Продвинутые RAII-паттерны

### ScopeGuard (выполнить код при выходе)

```cpp
template<typename Func>
class ScopeGuard {
    Func func;
    bool dismissed = false;
    
public:
    ScopeGuard(Func f) : func(std::move(f)) {}
    
    ~ScopeGuard() {
        if (!dismissed) func();
    }
    
    void dismiss() { dismissed = true; }
};

// Использование
void risky_function() {
    FILE* f = fopen("log.txt", "w");
    auto guard = ScopeGuard([&]() { 
        fclose(f); 
        std::cout << "File closed\n";
    });
    
    // ... какой-то код, даже с исключениями
    if (error) return;  // guard закроет файл
    
    guard.dismiss();  // всё ок, не нужно закрывать
}
```

### RAII для транзакций БД

```cpp
class Transaction {
    Database& db;
    bool committed = false;
    
public:
    Transaction(Database& db_) : db(db_) {
        db.begin_transaction();
    }
    
    ~Transaction() {
        if (!committed) {
            db.rollback();  // откат при исключении
        }
    }
    
    void commit() {
        db.commit();
        committed = true;
    }
};

void transferMoney(Database& db, int from, int to, int amount) {
    Transaction tx(db);  // начало транзакции
    db.debit(from, amount);
    db.credit(to, amount);
    tx.commit();  // если дошли — фиксируем
}
```

---

## ⚠️ Частые ошибки с RAII

```cpp
// ❌ 1. Утечка в конструкторе (неполная инициализация)
class Leaky {
    int* a;
    int* b;
    
public:
    Leaky() {
        a = new int(10);
        risky();        // если исключение — a не удалится!
        b = new int(20);
    }
    
    ~Leaky() {
        delete a;
        delete b;
    }
};

// ✅ Исправление: сначала полностью инициализировать
class Fixed {
    std::unique_ptr<int> a;
    std::unique_ptr<int> b;
    
public:
    Fixed() 
        : a(std::make_unique<int>(10))
        , b(std::make_unique<int>(20)) 
    {
        risky();  // теперь безопасно
    }
};

// ❌ 2. Голый new в конструкторе без обёртки
class Bad {
    int* data;
public:
    Bad() : data(new int[100]) {}
    ~Bad() { delete[] data; }
    // забыли move/copy! будет double delete или утечка
};

// ✅ Исправление: используйте RAII-обёртку
class Good {
    std::vector<int> data;
public:
    Good() : data(100) {}
    // всё автоматически
};
```

---

## 📝 Быстрая памятка

```cpp
// Основные правила RAII:
// 1. Каждый ресурс — в своём RAII-классе
// 2. Конструктор выделяет, деструктор освобождает
// 3. RAII-класс должен быть move-способным
// 4. RAII-класс не должен быть copy-способным (обычно)

// Самые полезные RAII-обёртки в C++:
std::unique_ptr<T>     // один владелец
std::shared_ptr<T>     // много владельцев
std::vector<T>         // массив в куче
std::string            // строка
std::lock_guard        // мьютекс
std::ifstream          // файл
std::thread            // поток
```

---

## 🔗 Связь с другими концепциями

| Концепция | Связь с RAII |
|-----------|--------------|
| **Исключения** | RAII делает код безопасным при исключениях |
| **Move-семантика** | RAII-классы должны поддерживать move |
| **Умные указатели** | Это и есть RAII для динамической памяти |
| **Rule of Five** | Правильное управление ресурсами через RAII |

---

## 📎 Связанные заметки

 [[🔹Умные указатели]]
 [[🛡️RAII принцип]]
 [[🧵Многопоточность]]
[[⚰️Декомпозиция]]