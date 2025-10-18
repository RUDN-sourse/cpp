# Лекция: Структура проекта на C++ и сборка в Visual Studio

## Оглавление
1. [Основы структуры C++ проекта](#основы-структуры-c-проекта)
2. [Разделение кода между .h и .cpp файлами](#разделение-кода-между-h-и-cpp-файлами)
3. [Практический пример структуры проекта](#практический-пример-структуры-проекта)
4. [Сборка проекта в Visual Studio](#сборка-проекта-в-visual-studio)
5. [Частые проблемы и их решение](#частые-проблемы-и-их-решение)

---

## Основы структуры C++ проекта

### Зачем нужна правильная структура?

**✅ Правильная структура позволяет:**
- Упростить навигацию по коду
- Избежать циклических зависимостей
- Ускорить компиляцию
- Облегчить командную разработку
- Упростить тестирование и отладку

### Базовые компоненты проекта:

```
MyProject/
├── include/           # Публичные заголовочные файлы
├── src/              # Исходные файлы (.cpp)
├── lib/              # Внешние библиотеки
├── tests/            # Тесты
├── docs/             # Документация
└── CMakeLists.txt    # Файл для сборки (если используется CMake)
```

---

## Разделение кода между .h и .cpp файлами

### 📁 Заголовочные файлы (.h / .hpp)

**Что должно находиться в .h файлах:**

```cpp
// MathUtils.h
#pragma once  // Современная замена #ifndef/#define

// 1. Включение необходимых заголовков
#include <vector>
#include <string>

// 2. Предварительные объявления (forward declarations)
class SomeClass;  // Вместо #include "SomeClass.h"

// 3. Пространства имен
namespace Math {
    
// 4. Объявления констант
extern const double PI;

// 5. Объявления функций
double calculateCircleArea(double radius);
int factorial(int n);

// 6. Объявления классов
class Calculator {
public:
    // Только объявления методов!
    Calculator();
    explicit Calculator(double initialValue);
    
    // 💡 В заголовках - только объявления, без реализаций
    double add(double value);
    double subtract(double value);
    double getCurrentValue() const;
    
private:
    // Объявления приватных членов
    double currentValue;
    
    // 💡 Встроенные (inline) функции - исключение
    void reset() { currentValue = 0; }  // Простая реализация в заголовке
};

} // namespace Math
```

**❌ ЧЕГО НЕ ДОЛЖНО БЫТЬ в .h файлах:**
- Сложные реализации функций (кроме простых inline)
- Определения глобальных переменных
- Большие участки исполняемого кода

### 📁 Исходные файлы (.cpp)

**Что должно находиться в .cpp файлах:**

```cpp
// MathUtils.cpp
#include "MathUtils.h"  // Соответствующий заголовок всегда первый
#include <iostream>     // Затем системные заголовки
#include <stdexcept>

// 1. Определения констант
const double Math::PI = 3.141592653589793;

// 2. Реализации функций
double Math::calculateCircleArea(double radius) {
    if (radius < 0) {
        throw std::invalid_argument("Radius cannot be negative");
    }
    return PI * radius * radius;
}

int Math::factorial(int n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);
}

// 3. Реализации методов классов
Math::Calculator::Calculator() : currentValue(0) {}

Math::Calculator::Calculator(double initialValue) 
    : currentValue(initialValue) {}

double Math::Calculator::add(double value) {
    currentValue += value;
    return currentValue;
}

double Math::Calculator::subtract(double value) {
    currentValue -= value;
    return currentValue;
}

double Math::Calculator::getCurrentValue() const {
    return currentValue;
}
```

---

## Практический пример структуры проекта

### 🎯 Пример: Проект "Банковская система"

```
BankingSystem/
├── include/
│   ├── account/
│   │   ├── Account.h
│   │   └── SavingsAccount.h
│   ├── transaction/
│   │   └── Transaction.h
│   └── bank/
│       └── Bank.h
├── src/
│   ├── account/
│   │   ├── Account.cpp
│   │   └── SavingsAccount.cpp
│   ├── transaction/
│   │   └── Transaction.cpp
│   ├── bank/
│   │   └── Bank.cpp
│   └── main.cpp
├── tests/
│   ├── test_account.cpp
│   └── test_bank.cpp
└── docs/
    └── README.md
```

### Пример кода:

**Account.h:**
```cpp
#pragma once
#include <string>
#include <vector>

namespace Banking {
    
class Transaction;  // Forward declaration вместо #include

class Account {
public:
    Account(const std::string& accountNumber, double initialBalance = 0);
    virtual ~Account() = default;
    
    // 💡 Виртуальные функции для полиморфизма
    virtual void deposit(double amount);
    virtual bool withdraw(double amount);
    virtual double getBalance() const;
    
    std::string getAccountNumber() const;
    void addTransaction(const Transaction& transaction);
    
protected:
    // 💡 Protected для доступа в наследниках
    double balance;
    
private:
    std::string accountNumber;
    std::vector<Transaction> transactions;
};

} // namespace Banking
```

**Account.cpp:**
```cpp
#include "Account.h"
#include "Transaction.h"  // Теперь включаем для использования
#include <stdexcept>

namespace Banking {

Account::Account(const std::string& accNumber, double initialBalance)
    : accountNumber(accNumber), balance(initialBalance) {
    if (initialBalance < 0) {
        throw std::invalid_argument("Initial balance cannot be negative");
    }
}

void Account::deposit(double amount) {
    if (amount <= 0) {
        throw std::invalid_argument("Deposit amount must be positive");
    }
    balance += amount;
}

bool Account::withdraw(double amount) {
    if (amount <= 0) {
        throw std::invalid_argument("Withdrawal amount must be positive");
    }
    if (amount > balance) {
        return false;
    }
    balance -= amount;
    return true;
}

double Account::getBalance() const {
    return balance;
}

std::string Account::getAccountNumber() const {
    return accountNumber;
}

void Account::addTransaction(const Transaction& transaction) {
    transactions.push_back(transaction);
}

} // namespace Banking
```

---

## Сборка проекта в Visual Studio

### 🛠️ Пошаговая инструкция

#### Шаг 1: Создание нового проекта

1. **Запустите Visual Studio**
2. **Создайте новый проект:** File → New → Project
3. **Выберите шаблон:** C++ → Windows → Console App → Empty Project
4. **Укажите имя проекта:** `BankingSystem`
5. **Выберите расположение**

#### Шаг 2: Организация структуры папок в Solution Explorer

1. **Создайте фильтры (виртуальные папки):**
   - Правая кнопка на проекте → Add → New Filter
   - Создайте: `headers`, `sources`, `tests`

2. **Добавьте заголовочные файлы:**
   - Правой кнопкой на `headers` → Add → New Item
   - Выберите "Header File (.h)"
   - Создайте все необходимые .h файлы

3. **Добавьте исходные файлы:**
   - Правой кнопкой на `sources` → Add → New Item
   - Выберите "C++ File (.cpp)"
   - Создайте все необходимые .cpp файлы

#### Шаг 3: Настройка путей включения (Include Directories)

1. **Откройте свойства проекта:**
   - Правой кнопкой на проекте → Properties

2. **Настройте дополнительные include-директории:**
   - Configuration: All Configurations
   - C/C++ → General → Additional Include Directories
   - Добавьте: `./headers` или `$(ProjectDir)headers`

   ![Include Directories Setup](https://docs.microsoft.com/en-us/cpp/build/media/vs2017-property-page-additional-include-directories.png?view=msvc-170)

#### Шаг 4: Настройка компиляции

1. **Предварительно скомпилированные заголовки (рекомендуется отключить для небольших проектов):**
   - C/C++ → Precompiled Headers
   - Precompiled Header: Not Using Precompiled Headers

2. **Уровень предупреждений:**
   - C/C++ → General → Warning Level
   - Рекомендуется: Level4 (/W4)

#### Шаг 5: Сборка и запуск

1. **Выберите конфигурацию:** Debug или Release
2. **Соберите проект:** Build → Build Solution (Ctrl+Shift+B)
3. **Запустите:** Debug → Start Without Debugging (Ctrl+F5)

### 🔧 Альтернативный способ: CMake с Visual Studio

**CMakeLists.txt:**
```cmake
cmake_minimum_required(VERSION 3.15)
project(BankingSystem)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Добавление исполняемого файла
add_executable(BankingSystem
    src/main.cpp
    src/account/Account.cpp
    src/account/SavingsAccount.cpp
    src/transaction/Transaction.cpp
    src/bank/Bank.cpp
)

# Установка путей для включения заголовков
target_include_directories(BankingSystem PRIVATE include)

# Настройка компилятора
target_compile_options(BankingSystem PRIVATE /W4)
```

**Использование:**
1. File → Open → CMake
2. Выберите CMakeLists.txt
3. Visual Studio автоматически сгенерирует проект

---

## Частые проблемы и их решение

### ❌ Проблема 1: Циклические включения

**Симптомы:** Ошибки компиляции "неполный тип", бесконечная рекурсия включений

**Решение:**
```cpp
// ❌ ПЛОХО - циклическое включение
// FileA.h
#include "FileB.h"

// FileB.h  
#include "FileA.h"

// ✅ ХОРОШО - использование предварительных объявлений
// FileA.h
class ClassB;  // Forward declaration вместо #include "FileB.h"

class ClassA {
    ClassB* b;  // Можно использовать указатели/ссылки
};
```

### ❌ Проблема 2: Множественное определение

**Симптомы:** LNK2005 ошибка линковки

**Решение:**
```cpp
// ❌ ПЛОХО - определение в заголовке
// Constants.h
int globalVariable = 42;  // Определение в заголовке

// ✅ ХОРОШО - объявление в заголовке, определение в .cpp
// Constants.h
extern int globalVariable;  // Объявление

// Constants.cpp
int globalVariable = 42;    // Определение
```

### ❌ Проблема 3: Неправильные пути включения

**Симптомы:** Ошибка "cannot open source file", C1083

**Решение:**
- Используйте относительные пути правильно
- Настройте Additional Include Directories в свойствах проекта
- Используйте угловые скобки для системных заголовков, кавычки для своих:

```cpp
#include <iostream>      // Системные заголовки
#include "Account.h"     // Пользовательские заголовки
#include "bank/Bank.h"   // Заголовки из поддиректорий
```

### ❌ Проблема 4: Нарушение ODR (One Definition Rule)

**Симптомы:** Непредсказуемое поведение, ошибки линковки

**Решение:**
```cpp
// В заголовках используйте inline для функций
inline int helperFunction(int x) {
    return x * 2;
}

// Или объявляйте как static
static int helperFunction(int x) {
    return x * 2;
}
```

### ❌ Проблема 5: Проблемы с пространствами имен

**Симптомы:** Неразрешенные символы при линковке

**Решение:**
```cpp
// ❌ ПЛОХО
// Header.h
namespace MyNamespace {
    void function();
}

// Source.cpp  
void function() {}  // Забыли namespace!

// ✅ ХОРОШО
// Source.cpp
namespace MyNamespace {
    void function() {}  // Правильно!
}

// Или используйте полную квалификацию
void MyNamespace::function() {}
```

---

## 🎯 Лучшие практики

### Организация кода:
1. **Один класс - один заголовочный файл**
2. **Имена файлов соответствуют именам классов**
3. **Используйте пространства имен для логической группировки**
4. **Разделяйте код по функциональным модулям**

### Защита от множественного включения:
```cpp
// Современный способ (рекомендуется)
#pragma once

// Традиционный способ
#ifndef MYHEADER_H
#define MYHEADER_H
// код заголовка
#endif
```

### Порядок включения заголовков:
```cpp
// 1. Соответствующий .h файл
#include "MyClass.h"

// 2. Системные заголовки
#include <vector>
#include <string>

// 3. Другие заголовки проекта
#include "OtherClass.h"
```

### Использование forward declarations:
```cpp
// Используйте предварительные объявления когда возможно
class OtherClass;  // Вместо #include "OtherClass.h"

class MyClass {
    OtherClass* ptr;  // OK - указатель
    OtherClass& ref;  // OK - ссылка
    // OtherClass obj; // ОШИБКА - нужен полный тип
};
```

Эта структура обеспечит чистоту, поддерживаемость и масштабируемость вашего C++ проекта!
