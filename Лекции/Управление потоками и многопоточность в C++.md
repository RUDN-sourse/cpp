# Полная лекция: Управление потоками, многопоточность и асинхронное программирование в C++

## Введение в многопоточность и асинхронность

### Что такое многопоточность и зачем она нужна?

**Объяснение для студентов:**
Представьте, что вы - шеф-повар на кухне. Если вы работаете один (однопоточная программа), вы должны делать все последовательно: нарезать овощи, затем варить суп, затем готовить мясо. Если же у вас есть помощники (многопоточность), вы можете делегировать задачи - один помощник режет овощи, другой варит суп, а вы готовите мясо. Все задачи выполняются параллельно, и общее время приготовления сокращается.

**Техническое определение:**
**Многопоточность** - это возможность программы выполнять несколько потоков выполнения одновременно. Каждый поток - это независимая последовательность инструкций, которые могут выполняться параллельно с другими потоками.

**Конкретные преимущества:**

1. **Повышение производительности** - использование нескольких ядер процессора
   - Пример: обработка изображения делится на части, каждая часть обрабатывается в отдельном потоке

2. **Улучшение отзывчивости** - основной поток UI не блокируется
   - Пример: при загрузке файла интерфейс продолжает отвечать на действия пользователя

3. **Эффективное использование ресурсов** - пока один поток ожидает I/O, другие могут работать
   - Пример: веб-сервер обрабатывает multiple запросов одновременно

4. **Упрощение архитектуры** - естественное разделение задач
   - Пример: игра с отдельными потоками для графики, физики и звука

**Асинхронное программирование** - это подход, при котором задачи запускаются без немедленного ожидания их завершения. Основной поток продолжает работу, а результат становится доступен позже, когда задача завершится.

---

## Часть 1: Основы многопоточности в C++

### 1.1 Создание и управление потоками

#### Подключение необходимых заголовков

```cpp
#include <iostream>
#include <thread>      // для работы с потоками
#include <chrono>      // для работы со временем
#include <vector>      // для хранения потоков
```

**Пошаговое объяснение:**

1. **`#include <iostream>`** - стандартная библиотека ввода/вывода для вывода сообщений
2. **`#include <thread>`** - самый важный заголовок, содержит класс `std::thread` и связанные функции
3. **`#include <chrono>`** - предоставляет функции для работы со временем: `sleep_for()`, `duration`
4. **`#include <vector>`** - понадобится для хранения multiple потоков или их результатов

#### Простейший пример создания потока

```cpp
void simpleFunction() {
    std::cout << "Hello from thread!" << std::endl;
}

int main() {
    // Шаг 1: Создание потока
    std::thread t(simpleFunction);
    
    // Шаг 2: Ожидание завершения потока
    t.join();
    
    return 0;
}
```

**Пошаговое объяснение выполнения:**

**Шаг 1: `std::thread t(simpleFunction);`**
- Создается объект `std::thread` с именем `t`
- Конструктор получает указатель на функцию `simpleFunction`
- В этот момент операционная система создает новый поток выполнения
- Новый поток немедленно начинает выполнять функцию `simpleFunction`
- Основной поток (main) продолжает выполнение следующей инструкции

**Что происходит в памяти:**
- Создается новый стек вызовов для потока `t`
- Все локальные переменные внутри `simpleFunction` размещаются в этом новом стеке
- Потоки разделяют общую heap-память и глобальные переменные

**Шаг 2: `t.join();`**
- Метод `join()` блокирует основной поток до полного завершения потока `t`
- Это КРИТИЧЕСКИ важно - если поток не присоединить, программа аварийно завершится
- После `join()` все ресурсы потока освобождаются

**Что произойдет без join():**
```cpp
int main() {
    std::thread t(simpleFunction);
    // Нет t.join()!
    return 0; // Аварийное завершение!
}
```
При выходе из `main()` деструктор `std::thread` проверяет, присоединен ли поток. Если нет - вызывается `std::terminate()`.

### 1.2 Передача параметров в потоки

#### Передача параметров по значению

```cpp
void printMessage(const std::string& message, int count) {
    for (int i = 0; i < count; ++i) {
        std::cout << message << " (" << i + 1 << "/" << count << ")" << std::endl;
        std::this_thread::sleep_for(std::chrono::milliseconds(500));
    }
}

int main() {
    // Передача параметров в поток
    std::thread t(printMessage, "Hello from thread", 3);
    
    t.join();
    return 0;
}
```

**Пошаговое объяснение:**

**Строка `std::thread t(printMessage, "Hello from thread", 3);`:**
1. **Первый аргумент** - указатель на функцию `printMessage`
2. **Последующие аргументы** - параметры для этой функции
3. **Процесс создания:**
   - Основной поток создает КОПИИ всех параметров
   - Запускается новый поток
   - В новом потоке вызывается функция со скопированными параметрами

**Что именно копируется:**
- `"Hello from thread"` - создается временный объект `std::string`
- `3` - копируется значение типа `int`

**Строка `std::this_thread::sleep_for(std::chrono::milliseconds(500));`:**
- `std::this_thread::sleep_for()` - статическая функция, приостанавливающая текущий поток
- `std::chrono::milliseconds(500)` - создает объект времени, представляющий 500 миллисекунд
- Поток "засыпает" на указанное время, освобождая процессор для других потоков

#### Передача параметров по ссылке

```cpp
void modifyValue(int& value) {
    value *= 2;
    std::cout << "Value inside thread: " << value << std::endl;
}

int main() {
    int x = 10;
    
    // Для передачи по ссылке используем std::ref
    std::thread t(modifyValue, std::ref(x));
    t.join();
    
    std::cout << "Value after thread: " << x << std::endl; // x = 20
    return 0;
}
```

**Пошаговое объяснение:**

**Проблема без std::ref:**
```cpp
std::thread t(modifyValue, x); // ПЕРЕДАЧА ПО ЗНАЧЕНИЮ!
```
Даже если функция объявлена как принимающая ссылку, параметр будет передан по значению, потому что поток работает с копиями.

**Решение с std::ref:**
1. **`std::ref(x)`** создает обертку ссылки вокруг переменной `x`
2. Эта обертка копируется в новый поток
3. Внутри потока обертка "разворачивается" обратно в ссылку
4. Поток получает доступ к ОРИГИНАЛЬНОЙ переменной `x`

**Что происходит в памяти:**
```
Основная память:
x = 10 (адрес 0x1000)

Поток main:
std::ref(x) → создает ссылку на 0x1000

Поток t:
получает копию ссылки → тоже указывает на 0x1000
изменяет значение по адресу 0x1000 → x становится 20
```

**Критически важное предупреждение:**
Убедитесь, что время жизни переменной покрывает время выполнения потока!
```cpp
void dangerousExample() {
    int localVar = 10;
    std::thread t(modifyValue, std::ref(localVar));
    // ОПАСНО! localVar уничтожится при выходе из функции!
    t.detach(); // Еще опаснее!
}
```

### 1.3 Методы класса std::thread

```cpp
class Worker {
public:
    void doWork(int id) {
        std::cout << "Worker " << id << " started" << std::endl;
        std::this_thread::sleep_for(std::chrono::seconds(1));
        std::cout << "Worker " << id << " finished" << std::endl;
    }
};

int main() {
    Worker worker;
    
    // Создание потока для метода класса
    std::thread t(&Worker::doWork, &worker, 1);
    
    // Проверка, присоединен ли поток
    if (t.joinable()) {
        std::cout << "Thread is joinable" << std::endl;
    }
    
    // Получение идентификатора потока
    std::thread::id threadId = t.get_id();
    std::cout << "Thread ID: " << threadId << std::endl;
    
    t.join();
    return 0;
}
```

**Пошаговое объяснение:**

**Создание потока для метода класса:**
`std::thread t(&Worker::doWork, &worker, 1);`

1. **`&Worker::doWork`** - берем указатель на метод класса (не обычная функция!)
2. **`&worker`** - указатель на объект, для которого вызывается метод
3. **`1`** - параметр, передаваемый в метод `doWork`

**Почему это работает:**
Компилятор преобразует вызов метода в примерно такой код:
```cpp
// Внутренняя реализация (упрощенно)
void thread_function(Worker* obj, int id) {
    obj->doWork(id);  // Вызов метода через указатель на объект
}
```

**Методы класса std::thread:**

1. **`joinable()`** - проверяет, можно ли присоединить поток
   - Возвращает `false` если: поток уже присоединен, отсоединен, или не ассоциирован с функцией

2. **`get_id()`** - возвращает уникальный идентификатор потока
   - Тип `std::thread::id` можно сравнивать и выводить
   - Если поток не запущен, возвращает специальное значение `std::thread::id()`

3. **`join()`** - блокирует текущий поток до завершения целевого
   - Можно вызывать только один раз
   - После `join()` поток становится неприсоединяемым

4. **`detach()`** - отсоединяет поток, позволяя ему работать независимо
   - После `detach()` поток управляется операционной системой
   - Невозможно получить результат или дождаться завершения

---

## Часть 2: Синхронизация потоков

### 2.1 Проблема гонки данных (Data Race)

```cpp
int counter = 0;

void incrementCounter(int iterations) {
    for (int i = 0; i < iterations; ++i) {
        // ПРОБЛЕМА: несколько потоков одновременно читают и изменяют counter
        counter++;
    }
}

int main() {
    std::thread t1(incrementCounter, 100000);
    std::thread t2(incrementCounter, 100000);
    
    t1.join();
    t2.join();
    
    // Результат может быть меньше 200000 из-за гонки данных
    std::cout << "Final counter: " << counter << std::endl;
    return 0;
}
```

**Пошаговое объяснение проблемы:**

**Что такое гонка данных?**
Гонка данных происходит, когда два или более потоков одновременно обращаются к одной памяти, и хотя бы один выполняет запись.

**Почему `counter++` не атомарен?**
Операция `counter++` компилируется в три машинные инструкции:

```assembly
; Шаг 1: Чтение из памяти в регистр
mov eax, [counter]    ; Читаем значение counter в регистр eax

; Шаг 2: Увеличение значения в регистре
inc eax               ; Увеличиваем значение в eax на 1

; Шаг 3: Запись результата обратно в память
mov [counter], eax    ; Записываем результат обратно в counter
```

**Сценарий гонки данных на примере:**

| Время | Поток 1 | Поток 2 | counter в памяти |
|-------|---------|---------|------------------|
| t=0   | читает 100 | - | 100 |
| t=1   | увеличивает до 101 | читает 100 | 100 |
| t=2   | записывает 101 | увеличивает до 101 | 101 |
| t=3   | - | записывает 101 | 101 |

**Результат:** Оба потока увеличивали счетчик, но значение изменилось только на 1 вместо 2.

**Почему это серьезная проблема:**
- Результат непредсказуем и зависит от timing'а
- Очень сложно отлаживать - проблема может проявляться случайно
- Может привести к corrupted data и сбоям программы

### 2.2 Решение с помощью мьютексов

```cpp
#include <mutex>

int counter = 0;
std::mutex counterMutex;  // Мьютекс для синхронизации доступа к counter

void safeIncrementCounter(int iterations) {
    for (int i = 0; i < iterations; ++i) {
        // Захватываем мьютекс перед доступом к разделяемым данным
        counterMutex.lock();
        counter++;
        counterMutex.unlock();
    }
}

void saferIncrementCounter(int iterations) {
    for (int i = 0; i < iterations; ++i) {
        // Более безопасный способ с использованием std::lock_guard
        std::lock_guard<std::mutex> lock(counterMutex);
        counter++;
    } // Мьютекс автоматически освобождается здесь
}

int main() {
    std::thread t1(saferIncrementCounter, 100000);
    std::thread t2(saferIncrementCounter, 100000);
    
    t1.join();
    t2.join();
    
    std::cout << "Final counter: " << counter << std::endl; // Всегда 200000
    return 0;
}
```

**Пошаговое объяснение мьютексов:**

**Что такое мьютекс?**
Мьютекс (mutual exclusion) - это примитив синхронизации, который позволяет только одному потоку выполнять определенный участок кода в данный момент.

**Как работает мьютекс:**
1. **Захват (lock)** - поток пытается захватить мьютекс
   - Если мьютекс свободен - поток захватывает его и продолжает выполнение
   - Если мьютекс занят - поток блокируется до освобождения

2. **Освобождение (unlock)** - поток освобождает мьютекс
   - Один из ожидающих потоков получает мьютекс и продолжает работу

**Проблемы ручного управления:**
```cpp
counterMutex.lock();
// Если здесь выбросится исключение...
counter++;
// ...эта строка не выполнится!
counterMutex.unlock();
```

**Решение: RAII с std::lock_guard**
`std::lock_guard<std::mutex> lock(counterMutex);`

1. **В конструкторе** - автоматически захватывает мьютекс
2. **В деструкторе** - автоматически освобождает мьютекс
3. **Гарантия** - мьютекс будет освобожден даже при выбросе исключения

**Что происходит под капотом:**
```cpp
// Упрощенная реализация lock_guard
class lock_guard {
private:
    std::mutex& mtx;
public:
    lock_guard(std::mutex& m) : mtx(m) { mtx.lock(); }  // Конструктор - захват
    ~lock_guard() { mtx.unlock(); }                     // Деструктор - освобождение
};
```

### 2.3 Более сложные механизмы синхронизации

#### std::unique_lock

```cpp
std::mutex resourceMutex;
std::string sharedResource;

void workerWithUniqueLock(int id) {
    // unique_lock более гибкий, чем lock_guard
    std::unique_lock<std::mutex> lock(resourceMutex);
    
    sharedResource = "Modified by thread " + std::to_string(id);
    std::cout << sharedResource << std::endl;
    
    // Можно временно освободить мьютекс
    lock.unlock();
    
    // Выполняем работу, не требующую синхронизации
    std::this_thread::sleep_for(std::chrono::milliseconds(100));
    
    // И снова захватить при необходимости
    lock.lock();
    sharedResource = "Final modification by thread " + std::to_string(id);
}
```

**Пошаговое объяснение std::unique_lock:**

**Отличия от std::lock_guard:**
- `std::unique_lock` более гибкий, но использует больше памяти
- Позволяет явно управлять захватом/освобождением
- Поддерживает отложенный захват
- Можно передавать владение

**Ключевые возможности:**

1. **Отложенный захват:**
```cpp
std::unique_lock<std::mutex> lock(mutex, std::defer_lock);
// Мьютекс еще не захвачен!
lock.lock(); // Захватываем явно
```

2. **Временное освобождение:**
```cpp
lock.unlock(); // Временно освобождаем
// Выполняем работу без блокировки
lock.lock();   // Снова захватываем
```

3. **Передача владения:**
```cpp
std::unique_lock<std::mutex> lock1(mutex);
std::unique_lock<std::mutex> lock2 = std::move(lock1); // Владение передано
```

**Когда использовать:**
- Когда нужен тонкий контроль над временем удержания блокировки
- Для использования с условными переменными
- Когда нужно временно освобождать мьютекс во время работы

#### std::condition_variable

```cpp
#include <condition_variable>
#include <queue>

std::queue<int> dataQueue;
std::mutex queueMutex;
std::condition_variable queueCondition;
bool producerFinished = false;

void producer() {
    for (int i = 0; i < 10; ++i) {
        {
            std::lock_guard<std::mutex> lock(queueMutex);
            dataQueue.push(i);
            std::cout << "Produced: " << i << std::endl;
        }
        // Уведомляем один ожидающий потребитель
        queueCondition.notify_one();
        std::this_thread::sleep_for(std::chrono::milliseconds(100));
    }
    
    {
        std::lock_guard<std::mutex> lock(queueMutex);
        producerFinished = true;
    }
    queueCondition.notify_all();  // Уведомляем всех потребителей
}

void consumer(int id) {
    while (true) {
        std::unique_lock<std::mutex> lock(queueMutex);
        
        // Ожидаем, пока в очереди появятся данные или производитель завершит работу
        queueCondition.wait(lock, []() {
            return !dataQueue.empty() || producerFinished;
        });
        
        // Если очередь пуста и производитель завершил работу - выходим
        if (dataQueue.empty() && producerFinished) {
            break;
        }
        
        // Извлекаем данные из очереди
        if (!dataQueue.empty()) {
            int value = dataQueue.front();
            dataQueue.pop();
            lock.unlock();  // Освобождаем мьютекс как можно раньше
            
            std::cout << "Consumer " << id << " consumed: " << value << std::endl;
        } else {
            lock.unlock();
        }
    }
}
```

**Пошаговое объяснение условных переменных:**

**Что такое condition_variable?**
Это механизм, позволяющий потоку ждать определенного условия, освобождая мьютекс на время ожидания.

**Как работает `wait()`:**
1. **Освобождает мьютекс** (поток перестает блокировать другие потоки)
2. **Блокирует поток** до получения уведомления
3. **После уведомления** снова захватывает мьютекс
4. **Проверяет условие** (второй параметр - предикат)

**Разбор вызова `wait()`:**
```cpp
queueCondition.wait(lock, []() {
    return !dataQueue.empty() || producerFinished;
});
```

1. **Первый параметр** - `unique_lock`, который будет временно освобожден
2. **Второй параметр** - лямбда-функция, проверяющая условие
3. **Алгоритм работы:**
   - Проверяет условие: если true - сразу продолжает
   - Если false - освобождает мьютекс и ждет
   - При получении `notify()` проверяет условие снова
   - Если условие истинно - захватывает мьютекс и продолжает

**Уведомления:**
- `notify_one()` - будит один случайный ожидающий поток
- `notify_all()` - будит все ожидающие потоки

**Преимущества перед активным ожиданием:**
```cpp
// ПЛОХО: активное ожидание (busy waiting)
while (dataQueue.empty()) {
    // Постоянно проверяет условие, тратит процессорное время
}
```

### 2.4 Атомарные операции

```cpp
#include <atomic>

// Атомарная переменная - операции с ней неделимы
std::atomic<int> atomicCounter(0);

void atomicIncrement(int iterations) {
    for (int i = 0; i < iterations; ++i) {
        atomicCounter++;  // Атомарная операция, не требует мьютекса
    }
}

int main() {
    std::thread t1(atomicIncrement, 100000);
    std::thread t2(atomicIncrement, 100000);
    
    t1.join();
    t2.join();
    
    std::cout << "Atomic counter: " << atomicCounter << std::endl; // Всегда 200000
    return 0;
}
```

**Пошаговое объяснение атомарных операций:**

**Что такое атомарная операция?**
Операция, которая выполняется как единое целое - она либо полностью выполнена, либо не выполнена вовсе. Невозможно наблюдать частично выполненную атомарную операцию.

**Как работает `atomicCounter++`:**
- Компилятор и процессор гарантируют, что операция выполняется атомарно
- Используются специальные процессорные инструкции (LOCK INC в x86)
- Никакой другой поток не может увидеть промежуточное состояние

**Сравнение с мьютексом:**
```cpp
// С мьютексом
std::lock_guard<std::mutex> lock(mutex);
counter++;  // 3 инструкции под защитой мьютекса

// С атомарной переменной
atomicCounter++;  // 1 атомарная инструкция
```

**Преимущества атомарных операций:**
- **Быстрее** - нет накладных расходов на блокировки
- **Проще** - не нужно явно управлять мьютексами
- **Безопаснее** - невозможно забыть освободить блокировку

**Ограничения:**
- Подходят только для простых операций (инкремент, декремент, замена)
- Не заменяют мьютексы для сложных критических секций

**Когда использовать атомарные операции:**
- Счетчики, флаги, простые shared состояния
- Когда операции простые и изолированные

**Когда использовать мьютексы:**
- Сложные операции над multiple переменными
- Когда нужны condition variables
- Для защиты сложных структур данных

---

## Часть 3: Асинхронное программирование в C++

### 3.1 Базовые компоненты асинхронности

#### Подключение необходимых заголовков

```cpp
#include <iostream>
#include <future>      // для std::async, std::future, std::promise
#include <chrono>      // для работы со временем
#include <thread>      // для std::thread
#include <vector>      // для хранения результатов
#include <numeric>     // для std::accumulate
```

**Объяснение заголовков:**
- **`<future>`** - основной заголовок для асинхронного программирования
- **`std::async`** - запуск асинхронных задач
- **`std::future`** - получение результатов асинхронных операций  
- **`std::promise`** - ручное управление асинхронными результатами
- Остальные заголовки - вспомогательные для примеров

### 3.2 std::async и std::future

#### Базовое использование std::async

```cpp
// Функция, которая выполняется длительное время
int longRunningTask(int x, int y) {
    std::cout << "Long running task started in thread: " 
              << std::this_thread::get_id() << std::endl;
    
    // Имитация длительной работы
    std::this_thread::sleep_for(std::chrono::seconds(2));
    
    std::cout << "Long running task finished" << std::endl;
    return x + y;
}

int main() {
    std::cout << "Main thread: " << std::this_thread::get_id() << std::endl;
    
    // Запускаем асинхронную задачу
    std::future<int> result = std::async(std::launch::async, longRunningTask, 10, 20);
    
    // Основной поток продолжает работу
    std::cout << "Main thread is doing other work..." << std::endl;
    std::this_thread::sleep_for(std::chrono::seconds(1));
    std::cout << "Main thread still working..." << std::endl;
    
    // Получаем результат (блокируется, если задача еще не завершена)
    int value = result.get();
    
    std::cout << "Result: " << value << std::endl; // Result: 30
    return 0;
}
```

**Пошаговое объяснение:**

**Шаг 1: Запуск асинхронной задачи**
`std::future<int> result = std::async(std::launch::async, longRunningTask, 10, 20);`

1. **`std::launch::async`** - политика запуска, гарантирующая создание нового потока
2. **`longRunningTask`** - функция, которая будет выполнена асинхронно
3. **`10, 20`** - параметры для функции
4. **Возвращает** `std::future<int>` - объект, который БУДЕТ содержать результат

**Что происходит внутри:**
- Создается новый поток (или используется thread pool)
- Функция `longRunningTask` выполняется в этом потоке
- Основной поток немедленно продолжает выполнение

**Шаг 2: Работа основного потока**
```cpp
std::cout << "Main thread is doing other work..." << std::endl;
std::this_thread::sleep_for(std::chrono::seconds(1));
```
Основной поток НЕ БЛОКИРУЕТСЯ и может выполнять другую работу.

**Шаг 3: Получение результата**
`int value = result.get();`

1. Если задача уже завершена - немедленно возвращает результат
2. Если задача еще выполняется - БЛОКИРУЕТ текущий поток до завершения
3. После получения результата `future` становится невалидным

**Аналогия из жизни:**
Представьте, что вы заказываете еду в ресторане:
- `std::async` - вы делаете заказ (еда готовится на кухне)
- Основной поток - вы общаетесь с друзьями (не ждете у кухни)
- `future.get()` - когда еда готова, вы ее получаете

#### Различные политики запуска

```cpp
int simpleTask(int x) {
    std::cout << "Task executed in thread: " << std::this_thread::get_id() << std::endl;
    return x * x;
}

int main() {
    // 1. std::launch::async - всегда создает новый поток
    auto future1 = std::async(std::launch::async, simpleTask, 5);
    
    // 2. std::launch::deferred - выполнение откладывается до вызова get()
    auto future2 = std::async(std::launch::deferred, simpleTask, 10);
    
    // 3. По умолчанию (implementation-defined)
    auto future3 = std::async(simpleTask, 15);
    
    std::cout << "Main thread: " << std::this_thread::get_id() << std::endl;
    
    // future1 уже выполняется в другом потоке
    // future2 выполнится только здесь:
    std::cout << "Deferred result: " << future2.get() << std::endl;
    std::cout << "Async result: " << future1.get() << std::endl;
    std::cout << "Default result: " << future3.get() << std::endl;
    
    return 0;
}
```

**Пошаговое объяснение политик:**

**1. `std::launch::async`:**
- Гарантирует запуск в отдельном потоке
- Задача начинает выполняться НЕМЕДЛЕННО после создания
- Подходит для CPU-intensive задач

**2. `std::launch::deferred`:**
- Выполнение ОТКЛАДЫВАЕТСЯ до вызова `get()` или `wait()`
- Задача выполняется в ТЕКУЩЕМ потоке при запросе результата
- Подходит для ленивых вычислений

**3. По умолчанию:**
- Реализация выбирает стратегию автоматически
- Может быть как `async`, так и `deferred`
- НЕ рекомендуется для production кода - поведение непредсказуемо

**Визуализация выполнения:**

```
Время:    t0     t1     t2     t3
Main:     [async][defer][def?][work]
Async:    [task1....................]
Deferred:         [task2]
Default:                 [task3?]
```

### 3.3 Обработка исключений в асинхронных задачах

```cpp
double riskyCalculation(double a, double b) {
    if (b == 0) {
        throw std::invalid_argument("Division by zero!");
    }
    
    std::this_thread::sleep_for(std::chrono::seconds(1));
    return a / b;
}

int main() {
    try {
        // Запускаем асинхронную задачу, которая может выбросить исключение
        auto future = std::async(std::launch::async, riskyCalculation, 10.0, 0.0);
        
        // Исключение будет проброшено при вызове get()
        double result = future.get();
        std::cout << "Result: " << result << std::endl;
    }
    catch (const std::exception& e) {
        std::cout << "Exception caught: " << e.what() << std::endl;
    }
    
    return 0;
}
```

**Пошаговое объяснение обработки исключений:**

**Что происходит при исключении в асинхронной задаче:**

1. **В асинхронном потоке:**
   - Функция `riskyCalculation` бросает исключение
   - Исключение ПЕРЕХВАТЫВАЕТСЯ внутренним механизмом `std::async`
   - Сохраняется в объекте `std::future`

2. **В основном потоке:**
   - При вызове `future.get()` исключение ПРОБРАСЫВАЕТСЯ заново
   - Обрабатывается в блоке `catch` основного потока

**Альтернативный сценарий без try-catch:**
```cpp
auto future = std::async(std::launch::async, riskyCalculation, 10.0, 0.0);
double result = future.get(); // Исключение пробросится здесь!
// Если не перехватить - программа завершится
```

**Ключевое преимущество:**
Исключения передаются между потоками прозрачно, как будто функция вызывалась синхронно.

### 3.4 std::promise и std::future для ручного управления

```cpp
void complexCalculation(std::promise<int> resultPromise, int input) {
    try {
        std::cout << "Starting complex calculation with input: " << input << std::endl;
        
        // Имитация сложных вычислений
        std::this_thread::sleep_for(std::chrono::seconds(2));
        
        if (input < 0) {
            throw std::invalid_argument("Negative input not allowed");
        }
        
        int result = input * input;
        
        // Устанавливаем значение в promise
        resultPromise.set_value(result);
        std::cout << "Calculation completed" << std::endl;
    }
    catch (...) {
        // Перехватываем любое исключение и передаем через promise
        resultPromise.set_exception(std::current_exception());
    }
}

void workerWithPromise(int input) {
    // Создаем promise-future пару
    std::promise<int> promise;
    std::future<int> future = promise.get_future();
    
    // Передаем promise в поток (перемещаем, так как promise нельзя копировать)
    std::thread worker(complexCalculation, std::move(promise), input);
    
    // Основной поток может делать другую работу
    std::cout << "Main thread working..." << std::endl;
    
    try {
        // Ожидаем результат
        int result = future.get();
        std::cout << "Result: " << result << std::endl;
    }
    catch (const std::exception& e) {
        std::cout << "Error: " << e.what() << std::endl;
    }
    
    worker.join();
}
```

**Пошаговое объяснение std::promise:**

**Что такое promise-future пара?**
Это два связанных объекта:
- **`std::promise`** - производитель, устанавливает значение
- **`std::future`** - потребитель, получает значение

**Шаг 1: Создание пары**
```cpp
std::promise<int> promise;
std::future<int> future = promise.get_future();
```
- `promise` и `future` теперь связаны
- `future` будет получать значения от `promise`

**Шаг 2: Передача promise в поток**
```cpp
std::thread worker(complexCalculation, std::move(promise), input);
```
- `std::move(promise)` - promise нельзя копировать, только перемещать
- Поток получает владение promise

**Шаг 3: Установка значения в потоке**
```cpp
resultPromise.set_value(result);  // Успешное завершение
// ИЛИ
resultPromise.set_exception(std::current_exception());  // Ошибка
```

**Шаг 4: Получение результата в основном потоке**
```cpp
int result = future.get();  // Блокируется до установки значения
```

**Когда использовать promise вместо async:**
- Когда нужно больше контроля над потоком выполнения
- Для интеграции с callback-based API
- Когда результат устанавливается в multiple местах
- Для реализации custom асинхронных паттернов

### 3.5 std::shared_future для нескольких потребителей

```cpp
void producer(std::promise<int> promise) {
    std::this_thread::sleep_for(std::chrono::seconds(1));
    promise.set_value(42);
}

void consumer(std::shared_future<int> future, int id) {
    // shared_future можно копировать, в отличие от future
    std::cout << "Consumer " << id << " waiting..." << std::endl;
    
    int result = future.get();  // Может быть вызвано многократно
    std::cout << "Consumer " << id << " got: " << result << std::endl;
}

int main() {
    std::promise<int> promise;
    
    // Конвертируем future в shared_future
    std::shared_future<int> sharedFuture = promise.get_future().share();
    
    // Запускаем производителя
    std::thread prod(producer, std::move(promise));
    
    // Запускаем нескольких потребителей
    std::vector<std::thread> consumers;
    for (int i = 0; i < 3; ++i) {
        consumers.emplace_back(consumer, sharedFuture, i);
    }
    
    prod.join();
    for (auto& t : consumers) {
        t.join();
    }
    
    return 0;
}
```

**Пошаговое объяснение std::shared_future:**

**Проблема с std::future:**
- `std::future` можно использовать только ОДИН раз
- `get()` можно вызвать только ОДИН раз
- Нельзя копировать - только перемещать

**Решение: std::shared_future**
- Можно копировать и передавать в multiple потоки
- `get()` можно вызывать многократно
- Каждый вызов `get()` возвращает одно и то же значение

**Шаг 1: Конвертация**
```cpp
std::shared_future<int> sharedFuture = promise.get_future().share();
```
- `future.share()` конвертирует `std::future` в `std::shared_future`
- Оригинальный `future` становится невалидным

**Шаг 2: Использование в multiple потоках**
```cpp
consumers.emplace_back(consumer, sharedFuture, i);
```
- Каждый поток получает КОПИЮ `shared_future`
- Все копии ссылаются на один и тот же shared state

**Сценарии использования:**
- Broadcast результатов multiple потребителям
- Когда результат нужен в разных частях программы
- Для реализации publish-subscribe паттернов

### 3.6 Ожидание результатов с таймаутами

```cpp
#include <future>

std::string slowNetworkRequest(const std::string& url) {
    std::cout << "Starting request to: " << url << std::endl;
    
    // Имитация сетевого запроса с случайной длительностью
    std::random_device rd;
    std::mt19937 gen(rd());
    std::uniform_int_distribution<> dis(1, 5);
    int seconds = dis(gen);
    
    std::this_thread::sleep_for(std::chrono::seconds(seconds));
    
    if (seconds > 3) {
        throw std::runtime_error("Request timeout simulated");
    }
    
    return "Response from " + url + " (took " + std::to_string(seconds) + "s)";
}

int main() {
    auto future = std::async(std::launch::async, slowNetworkRequest, "http://example.com");
    
    // Ожидаем результат с таймаутом
    std::future_status status;
    
    do {
        status = future.wait_for(std::chrono::seconds(1));
        
        switch (status) {
            case std::future_status::ready:
                std::cout << "Task completed!" << std::endl;
                break;
            case std::future_status::timeout:
                std::cout << "Still waiting..." << std::endl;
                break;
            case std::future_status::deferred:
                std::cout << "Task is deferred" << std::endl;
                break;
        }
    } while (status != std::future_status::ready);
    
    try {
        auto result = future.get();
        std::cout << "Result: " << result << std::endl;
    }
    catch (const std::exception& e) {
        std::cout << "Error: " << e.what() << std::endl;
    }
    
    return 0;
}
```

**Пошаговое объяснение таймаутов:**

**Методы ожидания с таймаутом:**

1. **`wait_for(duration)`** - ждет указанное время
2. **`wait_until(time_point)`** - ждет до указанного момента

**Статусы выполнения:**

- **`std::future_status::ready`** - задача завершена, результат доступен
- **`std::future_status::timeout`** - время ожидания истекло, задача еще выполняется  
- **`std::future_status::deferred`** - выполнение задачи отложено

**Алгоритм работы цикла ожидания:**

1. **Начало цикла** - проверяем статус задачи
2. **`wait_for(1s)`** - ждем 1 секунду или завершение задачи
3. **Анализ статуса:**
   - `ready` - выходим из цикла, получаем результат
   - `timeout` - выводим сообщение, продолжаем ждать
   - `deferred` - (редко) задача еще не запущена
4. **Повтор** - продолжаем пока статус не `ready`

**Преимущества подхода с таймаутами:**

1. **Предотвращение бесконечного блокирования**
2. **Возможность показа прогресса** пользователю
3. **Освобождение ресурсов** при долгих операциях
4. **Улучшение отзывчивости** приложения

**Реальный пример:**
```cpp
// В GUI приложении
auto future = std::async(loadUserData, userId);
while (future.wait_for(100ms) != std::future_status::ready) {
    updateProgressBar();  // Обновляем интерфейс
    processEvents();      // Обрабатываем пользовательский ввод
}
// Получаем результат когда готов
```

---

## Часть 4: Практические примеры и лучшие практики

### 4.1 Практический пример: параллельная обработка данных

```cpp
#include <algorithm>
#include <random>

// Функция для параллельной сортировки вектора
void parallelSort(std::vector<int>& data, unsigned numThreads) {
    if (data.empty() || numThreads == 0) return;
    
    // Разделяем данные на части для каждого потока
    size_t chunkSize = data.size() / numThreads;
    std::vector<std::thread> threads;
    std::vector<std::vector<int>> chunks(numThreads);
    
    // Разделяем данные на чанки
    auto start = data.begin();
    for (unsigned i = 0; i < numThreads; ++i) {
        auto end = (i == numThreads - 1) ? data.end() : start + chunkSize;
        chunks[i].assign(start, end);
        start = end;
    }
    
    // Запускаем потоки для сортировки чанков
    for (unsigned i = 0; i < numThreads; ++i) {
        threads.emplace_back([&chunks, i]() {
            std::sort(chunks[i].begin(), chunks[i].end());
        });
    }
    
    // Ожидаем завершения всех потоков
    for (auto& t : threads) {
        t.join();
    }
    
    // Сливаем отсортированные чанки
    data.clear();
    for (auto& chunk : chunks) {
        data.insert(data.end(), chunk.begin(), chunk.end());
    }
    
    // Финальная сортировка слиянием (упрощенная версия)
    std::sort(data.begin(), data.end());
}
```

**Пошаговое объяснение параллельной сортировки:**

**Шаг 1: Подготовка данных**
```cpp
size_t chunkSize = data.size() / numThreads;
std::vector<std::vector<int>> chunks(numThreads);
```
- Вычисляем размер каждой части
- Создаем вектор для хранения частей

**Шаг 2: Разделение данных**
```cpp
auto start = data.begin();
for (unsigned i = 0; i < numThreads; ++i) {
    auto end = (i == numThreads - 1) ? data.end() : start + chunkSize;
    chunks[i].assign(start, end);
    start = end;
}
```
- Делим исходный вектор на равные части
- Последняя часть может быть больше если размер не делится нацело

**Шаг 3: Параллельная сортировка**
```cpp
threads.emplace_back([&chunks, i]() {
    std::sort(chunks[i].begin(), chunks[i].end());
});
```
- Каждый поток сортирует свою часть
- Используется лямбда-функция для захвата `chunks` и `i`

**Шаг 4: Слияние результатов**
```cpp
data.clear();
for (auto& chunk : chunks) {
    data.insert(data.end(), chunk.begin(), chunk.end());
}
std::sort(data.begin(), data.end());
```
- Собираем отсортированные части обратно
- Финальная сортировка для корректного слияния

### 4.2 Лучшие практики многопоточного и асинхронного программирования

#### Избегайте deadlock

```cpp
std::mutex mutex1, mutex2;

// ПРАВИЛЬНЫЙ способ - использовать std::lock для одновременного захвата
void safeFunction() {
    std::unique_lock<std::mutex> lock1(mutex1, std::defer_lock);
    std::unique_lock<std::mutex> lock2(mutex2, std::defer_lock);
    std::lock(lock1, lock2);  // Захватываем оба мьютекса атомарно
    // Работа с разделяемыми данными
}
```

**Объяснение проблемы deadlock:**

**Что такое deadlock?**
Взаимная блокировка, когда два или более потока ждут друг друга, и ни один не может продолжить.

**Сценарий deadlock:**
```
Поток 1: захватывает мьютекс A, пытается захватить B
Поток 2: захватывает мьютекс B, пытается захватить A
Результат: оба потока заблокированы навсегда
```

**Решение: всегда захватывать мьютексы в одном порядке**
```cpp
// std::lock гарантирует атомарный захват multiple мьютексов
std::lock(lock1, lock2);  // Захватывает оба или ни одного
```

#### Используйте thread_local для данных, специфичных для потока

```cpp
// Каждый поток получит свою копию этой переменной
thread_local int threadSpecificData = 0;

void threadFunction(int id) {
    threadSpecificData = id;  // Не влияет на значение в других потоках
    std::cout << "Thread " << id << " data: " << threadSpecificData << std::endl;
}
```

**Объяснение thread_local:**

**Что такое thread_local?**
Ключевое слово, которое создает отдельную копию переменной для каждого потока.

**Как работает:**
- Каждый поток имеет свою собственную копию переменной
- Изменения в одном потоке не видны в других
- Инициализация происходит при первом использовании в каждом потоке

**Использование:**
- Кэши, специфичные для потока
- Временные буферы
- Счетчики и статистика по потокам

#### Избегайте блокировок в асинхронных задачах

```cpp
// ХОРОШО: без блокировок или с минимальными блокировками
void goodAsyncTask(std::promise<int> promise) {
    // Выполняем вычисления без блокировок
    int result = 0;
    for (int i = 0; i < 1000000; ++i) {
        result += i;
    }
    promise.set_value(result);
}
```

**Почему важно избегать блокировок:**

1. **Снижение производительности** - блокировки создают contention
2. **Возможность deadlock** - особенно в сложных системах
3. **Усложнение отладки** - timing-related issues сложно воспроизвести

**Альтернативы блокировкам:**
- lock-free структуры данных
- Атомарные операции
- Разделение данных между потоками
- Асинхронные message passing

## Заключение

### Ключевые выводы для студентов:

1. **Многопоточность** - мощный инструмент для повышения производительности, но требующий осторожного использования

2. **Синхронизация** - обязательна при работе с общими данными. Без нее - гонки данных и неопределенное поведение

3. **Инструменты C++** предоставляют полный набор:
   - `std::thread` - базовые потоки
   - Мьютексы и lock_guard - синхронизация
   - `std::async` и `std::future` - высокоуровневая асинхронность
   - `std::promise` - ручное управление асинхронными операциями

4. **Лучшие практики:**
   - Всегда используйте RAII (`lock_guard`, `unique_lock`)
   - Избегайте deadlock с помощью упорядоченного захвата мьютексов
   - Обрабатывайте исключения в асинхронных задачах
   - Используйте атомарные операции для простых случаев
   - Минимизируйте время удержания блокировок

5. **Отладка многопоточных программ:**
   - Используйте thread sanitizers
   - Логируйте операции с мьютексами
   - Тестируйте на разных конфигурациях hardware

Правильное применение многопоточности и асинхронного программирования - ключ к созданию производительных и отзывчивых приложений в современном C++.
