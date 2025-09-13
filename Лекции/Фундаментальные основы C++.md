# Фундаментальные основы C++

## 1. Структура простейшей программы

### Базовые элементы программы на C++

```cpp
// Однострочный комментарий - игнорируется компилятором

/*
 Многострочный комментарий
 Используется для описания сложных алгоритмов
*/

// Директива препроцессора - подключение библиотеки
#include <iostream>

// Использование пространства имен std
// Позволяет не писать std:: перед cout, cin и т.д.
using namespace std;

// Главная функция программы - точка входа
// int - тип возвращаемого значения
// main - обязательное имя функции
// () - параметры функции (в данном случае отсутствуют)
int main() {
    // Вывод данных в консоль
    // cout - объект для вывода (console output)
    // << - оператор перенаправления данных
    // "Hello, World!" - строковый литерал
    // endl - манипулятор конца строки
    cout << "Hello, World!" << endl;
    
    // Возвращаемое значение
    // 0 означает успешное завершение программы
    return 0;
}
// Конец блока функции main
```

**Критически важные моменты:**

1. **Точка входа**: Любая программа на C++ должна содержать функцию `main()`
2. **Директивы #include**: Обрабатываются препроцессором ДО компиляции
3. **Пространства имен**: `using namespace std` - спорная практика. В больших проектах лучше использовать `std::cout`
4. **Точка с запятой**: Каждая инструкция заканчивается точкой с запятой (кроме блоков кода в {})
5. **Регистрозависимость**: C++ чувствителен к регистру - `Cout` и `cout` это разные вещи

### Альтернативный вариант (без using namespace):
```cpp
#include <iostream>

int main() {
    std::cout << "Hello, World!" << std::endl;
    return 0;
}
```

## 2. Переменные и типы данных

### Основные типы данных в C++

```cpp
#include <iostream>
using namespace std;

int main() {
    // ========== ЦЕЛОЧИСЛЕННЫЕ ТИПЫ ==========
    int age = 25;           // Основной целочисленный тип (4 байта)
    short smallNumber = 10; // Короткое целое (2 байта)
    long bigNumber = 1000;  // Длинное целое (4 или 8 байт)
    long long veryBig = 1000000LL; // Очень большое целое (8 байт)
    
    // ========== ВЕЩЕСТВЕННЫЕ ТИПЫ ==========
    float price = 19.99f;   // Число с плавающей точкой (4 байта)
    double pi = 3.14159;    // Двойная точность (8 байт)
    long double precise = 3.141592653589793238L; // Высокая точность
    
    // ========== СИМВОЛЬНЫЕ ТИПЫ ==========
    char grade = 'A';       // Один символ (1 байт)
    char newLine = '\n';    // Escape-последовательность
    wchar_t wideChar = L'Я'; // Широкий символ (для Unicode)
    
    // ========== ЛОГИЧЕСКИЙ ТИП ==========
    bool isStudent = true;  // true (1) или false (0)
    
    // ========== БЕЗЗНАКОВЫЕ ТИПЫ ==========
    unsigned int positive = 100;    // Только положительные
    unsigned short us = 500;
    unsigned long ul = 4000000000;
    
    cout << "Возраст: " << age << endl;
    cout << "Цена: " << price << endl;
    cout << "Оценка: " << grade << endl;
    cout << "Студент: " << isStudent << endl;
    
    return 0;
}
```

### Определение размеров типов
```cpp
#include <iostream>
using namespace std;

int main() {
    cout << "=== РАЗМЕРЫ ТИПОВ ДАННЫХ ===" << endl;
    cout << "int: " << sizeof(int) << " байт" << endl;
    cout << "short: " << sizeof(short) << " байт" << endl;
    cout << "long: " << sizeof(long) << " байт" << endl;
    cout << "long long: " << sizeof(long long) << " байт" << endl;
    cout << "float: " << sizeof(float) << " байт" << endl;
    cout << "double: " << sizeof(double) << " байт" << endl;
    cout << "char: " << sizeof(char) << " байт" << endl;
    cout << "bool: " << sizeof(bool) << " байт" << endl;
    
    return 0;
}
```

**Важные особенности:**

1. **Целочисленное деление**: `5 / 2 = 2` (потеря дробной части)
2. **Суффиксы**: `f` для float, `L` для long double, `LL` для long long
3. **Escape-последовательности**: `\n` - новая строка, `\t` - табуляция, `\\` - обратный слеш
4. **Размеры типов**: Зависят от компилятора и архитектуры процессора

## 3. Константы

### Способы объявления констант

```cpp
#include <iostream>
using namespace std;

// Константы времени компиляции
#define MAX_SIZE 100       // Стиль C (макроподстановка)
const int MIN_SIZE = 10;   // Стиль C++ (рекомендуется)

// Глобальная константа
const double PI = 3.1415926535;

int main() {
    // Локальные константы
    const int HOURS_IN_DAY = 24;
    const string GREETING = "Добро пожаловать!";
    
    // Константные выражения (C++11)
    constexpr int ARRAY_SIZE = 100;     // Вычисляется на этапе компиляции
    constexpr double CIRCLE = 2 * PI;   // Можно использовать другие constexpr
    
    // Ошибки при работе с константами:
    // PI = 3.14;          // ОШИБКА: константу нельзя изменить
    // HOURS_IN_DAY = 25;  // ОШИБКА: константу нельзя изменить
    
    // Указатели на константу
    int value = 50;
    const int* ptr = &value;    // Данные нельзя изменить через указатель
    // *ptr = 60;               // ОШИБКА
    
    // Константный указатель
    int* const constPtr = &value;   // Адрес нельзя изменить
    *constPtr = 60;                 // OK - данные можно менять
    // constPtr = &otherValue;      // ОШИБКА
    
    cout << "PI: " << PI << endl;
    cout << "Часов в дне: " << HOURS_IN_DAY << endl;
    cout << "Размер массива: " << ARRAY_SIZE << endl;
    
    return 0;
}
```

**Рекомендации по использованию констант:**

1. Используйте `const` вместо `#define` (типобезопасность)
2. Имена констант принято писать в UPPER_CASE
3. `constexpr` для значений, известных на этапе компиляции

## 4. Операции

### Арифметические операции

```cpp
#include <iostream>
using namespace std;

int main() {
    int a = 10, b = 3;
    
    cout << "a = " << a << ", b = " << b << endl;
    cout << "a + b = " << (a + b) << endl;  // Сложение: 13
    cout << "a - b = " << (a - b) << endl;  // Вычитание: 7
    cout << "a * b = " << (a * b) << endl;  // Умножение: 30
    cout << "a / b = " << (a / b) << endl;  // Деление: 3 (целочисленное!)
    cout << "a % b = " << (a % b) << endl;  // Остаток от деления: 1
    
    // Вещественное деление
    double x = 10.0, y = 3.0;
    cout << "x / y = " << (x / y) << endl;  // 3.33333
    
    // Инкремент и декремент
    int counter = 5;
    cout << "Исходное значение: " << counter << endl;
    cout << "counter++: " << counter++ << endl; // Постфиксный: 5
    cout << "После counter++: " << counter << endl; // 6
    cout << "++counter: " << ++counter << endl; // Префиксный: 7
    
    // Составные операторы присваивания
    int number = 10;
    number += 5;    // number = number + 5
    number -= 3;    // number = number - 3
    number *= 2;    // number = number * 2
    number /= 4;    // number = number / 4
    
    return 0;
}
```

### Операции сравнения и логические операции

```cpp
#include <iostream>
using namespace std;

int main() {
    int x = 5, y = 10;
    
    // Операции сравнения (возвращают bool)
    cout << "x == y: " << (x == y) << endl; // Равно: false
    cout << "x != y: " << (x != y) << endl; // Не равно: true
    cout << "x < y: " << (x < y) << endl;   // Меньше: true
    cout << "x > y: " << (x > y) << endl;   // Больше: false
    cout << "x <= y: " << (x <= y) << endl; // Меньше или равно: true
    cout << "x >= y: " << (x >= y) << endl; // Больше или равно: false
    
    // Логические операции
    bool condition1 = true, condition2 = false;
    
    cout << "condition1 && condition2: " << (condition1 && condition2) << endl; // AND: false
    cout << "condition1 || condition2: " << (condition1 || condition2) << endl; // OR: true
    cout << "!condition1: " << (!condition1) << endl; // NOT: false
    
    // Тернарный оператор
    int max = (x > y) ? x : y; // Если x > y, то max = x, иначе max = y
    cout << "Максимум: " << max << endl;
    
    return 0;
}
```

**Особенности операций:**

1. **Целочисленное деление**: При делении целых чисел дробная часть отбрасывается
2. **Инкремент**: `i++` (возвращает значение, затем увеличивает) vs `++i` (увеличивает, затем возвращает)
3. **Приоритет операций**: Умножение/деление выполняются перед сложением/вычитанием
4. **Тернарный оператор**: Условное выражение `условие ? значение1 : значение2`

## 5. Условные операторы

### Оператор if-else

```cpp
#include <iostream>
using namespace std;

int main() {
    int number;
    cout << "Введите число: ";
    cin >> number;
    
    // Простое условие
    if (number > 0) {
        cout << "Число положительное" << endl;
    }
    
    // if-else
    if (number % 2 == 0) {
        cout << "Число четное" << endl;
    } else {
        cout << "Число нечетное" << endl;
    }
    
    // Цепочка условий else if
    if (number > 100) {
        cout << "Число больше 100" << endl;
    } else if (number > 50) {
        cout << "Число между 51 и 100" << endl;
    } else if (number > 0) {
        cout << "Число между 1 и 50" << endl;
    } else if (number == 0) {
        cout << "Это ноль" << endl;
    } else {
        cout << "Число отрицательное" << endl;
    }
    
    // Вложенные условия
    if (number != 0) {
        if (number > 0) {
            cout << "Положительное ненулевое" << endl;
        } else {
            cout << "Отрицательное" << endl;
        }
    }
    
    return 0;
}
```

### Оператор switch

```cpp
#include <iostream>
using namespace std;

int main() {
    int day;
    cout << "Введите номер дня недели (1-7): ";
    cin >> day;
    
    switch (day) {
        case 1:
            cout << "Понедельник" << endl;
            break; // Важно! Без break выполнение пойдет дальше
        case 2:
            cout << "Вторник" << endl;
            break;
        case 3:
            cout << "Среда" << endl;
            break;
        case 4:
            cout << "Четверг" << endl;
            break;
        case 5:
            cout << "Пятница" << endl;
            break;
        case 6:
            cout << "Суббота" << endl;
            break;
        case 7:
            cout << "Воскресенье" << endl;
            break;
        default: // Выполняется если ни один case не подошел
            cout << "Неверный номер дня" << endl;
            break;
    }
    
    // Switch с несколькими case
    char grade;
    cout << "Введите оценку (A, B, C, D, F): ";
    cin >> grade;
    
    switch (grade) {
        case 'A':
        case 'a':
            cout << "Отлично!" << endl;
            break;
        case 'B':
        case 'b':
            cout << "Хорошо" << endl;
            break;
        case 'C':
        case 'c':
            cout << "Удовлетворительно" << endl;
            break;
        case 'D':
        case 'd':
            cout << "Плохо" << endl;
            break;
        case 'F':
        case 'f':
            cout << "Неудовлетворительно" << endl;
            break;
        default:
            cout << "Неверная оценка" << endl;
    }
    
    return 0;
}
```

**Важные моменты:**

1. **Фигурные скобки**: В if можно не ставить, если одна инструкция, но лучше всегда ставить
2. **break в switch**: Обязателен для предотвращения "проваливания"
3. **Типы в switch**: Можно использовать только целочисленные типы и enum
4. **default**: Не обязателен, но рекомендуется для обработки неожиданных значений

## 6. Циклы

### Цикл for

```cpp
#include <iostream>
using namespace std;

int main() {
    // Классический цикл for
    cout << "Цикл for:" << endl;
    for (int i = 0; i < 5; i++) {
        cout << "i = " << i << endl;
    }
    
    // For с несколькими переменными
    cout << "\nЦикл for с двумя переменными:" << endl;
    for (int i = 0, j = 10; i < j; i++, j--) {
        cout << "i = " << i << ", j = " << j << endl;
    }
    
    // Бесконечный цикл
    // for (;;) {
    //     // Бесконечное выполнение
    // }
    
    // Обход массива
    int numbers[] = {1, 2, 3, 4, 5};
    cout << "\nОбход массива:" << endl;
    for (int i = 0; i < 5; i++) {
        cout << "numbers[" << i << "] = " << numbers[i] << endl;
    }
    
    return 0;
}
```

### Цикл while

```cpp
#include <iostream>
using namespace std;

int main() {
    // Цикл while
    cout << "Цикл while:" << endl;
    int counter = 0;
    while (counter < 5) {
        cout << "counter = " << counter << endl;
        counter++;
    }
    
    // Цикл do-while (выполняется хотя бы один раз)
    cout << "\nЦикл do-while:" << endl;
    int number;
    do {
        cout << "Введите положительное число: ";
        cin >> number;
    } while (number <= 0);
    
    cout << "Вы ввели: " << number << endl;
    
    return 0;
}
```

### Управление циклами (break, continue)

```cpp
#include <iostream>
using namespace std;

int main() {
    // Оператор break - выход из цикла
    cout << "Break example:" << endl;
    for (int i = 0; i < 10; i++) {
        if (i == 5) {
            break; // Выход из цикла при i = 5
        }
        cout << i << " ";
    }
    cout << endl;
    
    // Оператор continue - переход к следующей итерации
    cout << "Continue example:" << endl;
    for (int i = 0; i < 10; i++) {
        if (i % 2 == 0) {
            continue; // Пропуск четных чисел
        }
        cout << i << " ";
    }
    cout << endl;
    
    // Вложенные циклы
    cout << "Вложенные циклы:" << endl;
    for (int i = 1; i <= 3; i++) {
        for (int j = 1; j <= 3; j++) {
            cout << "(" << i << "," << j << ") ";
        }
        cout << endl;
    }
    
    return 0;
}
```

**Рекомендации по циклам:**

1. **For**: Когда известно количество итераций
2. **While**: Когда условие продолжения известно заранее
3. **Do-while**: Когда нужно выполнить хотя бы одну итерацию
4. **Бесконечные циклы**: Всегда должны иметь условие выхода

## 7. Функции

### Объявление и определение функций

```cpp
#include <iostream>
using namespace std;

// Прототип функции (объявление)
int add(int a, int b); // Только типы параметров
double multiply(double x, double y);

// Функция без параметров и возвращаемого значения
void printHello() {
    cout << "Hello, World!" << endl;
}

// Функция с параметрами по умолчанию
void printMessage(string message = "Сообщение по умолчанию") {
    cout << message << endl;
}

int main() {
    // Вызов функций
    int result = add(5, 3);
    cout << "5 + 3 = " << result << endl;
    
    double product = multiply(2.5, 4.0);
    cout << "2.5 * 4.0 = " << product << endl;
    
    printHello();
    printMessage(); // Используется значение по умолчанию
    printMessage("Специальное сообщение");
    
    return 0;
}

// Определение функции (реализация)
int add(int a, int b) {
    return a + b; // Возврат значения
}

double multiply(double x, double y) {
    return x * y;
}
```

### Перегрузка функций

```cpp
#include <iostream>
using namespace std;

// Перегрузка функций - разные функции с одним именем
int max(int a, int b) {
    return (a > b) ? a : b;
}

double max(double a, double b) {
    return (a > b) ? a : b;
}

int max(int a, int b, int c) {
    return max(max(a, b), c);
}

int main() {
    cout << "max(5, 10) = " << max(5, 10) << endl;
    cout << "max(3.14, 2.71) = " << max(3.14, 2.71) << endl;
    cout << "max(1, 2, 3) = " << max(1, 2, 3) << endl;
    
    return 0;
}
```

### Рекурсивные функции

```cpp
#include <iostream>
using namespace std;

// Рекурсивная функция - вычисление факториала
unsigned long long factorial(int n) {
    if (n <= 1) { // Базовый случай
        return 1;
    }
    return n * factorial(n - 1); // Рекурсивный вызов
}

// Рекурсивное вычисление чисел Фибоначчи
int fibonacci(int n) {
    if (n <= 1) {
        return n;
    }
    return fibonacci(n - 1) + fibonacci(n - 2);
}

int main() {
    int number = 5;
    cout << "Факториал " << number << " = " << factorial(number) << endl;
    
    cout << "Числа Фибоначчи: ";
    for (int i = 0; i < 10; i++) {
        cout << fibonacci(i) << " ";
    }
    cout << endl;
    
    return 0;
}
```

**Особенности функций:**

1. **Прототипы**: Объявление до использования (можно без имен параметров)
2. **Параметры по умолчанию**: Должны быть в конце списка параметров
3. **Перегрузка**: Разные функции с одним именем, но разными параметрами
4. **Рекурсия**: Должен быть базовый случай для остановки

## 8. Ввод и вывод (I/O)

### Базовый ввод-вывод

```cpp
#include <iostream>
#include <string> // Для работы со строками
using namespace std;

int main() {
    // Вывод данных
    cout << "=== ВЫВОД ДАННЫХ ===" << endl;
    cout << "Hello, World!" << endl; // endl - перевод строки
    cout << "Число: " << 42 << endl;
    cout << "Дробное: " << 3.14 << endl;
    cout << "Символ: " << 'A' << endl;
    cout << "Булево: " << true << endl;
    
    // Форматирование вывода
    cout << "\n=== ФОРМАТИРОВАНИЕ ===" << endl;
    cout << "Табуляция:\t" << "текст после табуляции" << endl;
    cout << "Новая\nстрока" << endl;
    cout << "Кавычки: \"Hello\"" << endl;
    cout << "Обратный слеш: \\" << endl;
    
    // Ввод данных
    cout << "\n=== ВВОД ДАННЫХ ===" << endl;
    int age;
    double salary;
    string name;
    char initial;
    
    cout << "Введите ваше имя: ";
    cin >> name; // Чтение одного слова
    
    cout << "Введите инициал: ";
    cin >> initial;
    
    cout << "Введите возраст: ";
    cin >> age;
    
    cout << "Введите зарплату: ";
    cin >> salary;
    
    // Вывод введенных данных
    cout << "\n=== РЕЗУЛЬТАТ ===" << endl;
    cout << "Имя: " << name << endl;
    cout << "Инициал: " << initial << endl;
    cout << "Возраст: " << age << endl;
    cout << "Зарплата: " << salary << endl;
    
    // Чтение всей строки
    cout << "\nВведите полное имя: ";
    cin.ignore(); // Очистка буфера после предыдущего ввода
    getline(cin, name); // Чтение всей строки
    cout << "Полное имя: " << name << endl;
    
    return 0;
}
```

### Работа с файлами

```cpp
#include <iostream>
#include <fstream> // Для работы с файлами
#include <string>
using namespace std;

int main() {
    // Запись в файл
    ofstream outFile("example.txt"); // Создание файла для записи
    
    if (outFile.is_open()) {
        outFile << "Это первая строка" << endl;
        outFile << "Вторая строка" << endl;
        outFile << "Число: " << 42 << endl;
        outFile.close();
        cout << "Запись в файл завершена" << endl;
    } else {
        cout << "Ошибка открытия файла для записи" << endl;
    }
    
    // Чтение из файла
    ifstream inFile("example.txt");
    string line;
    
    if (inFile.is_open()) {
        cout << "\nСодержимое файла:" << endl;
        while (getline(inFile, line)) {
            cout << line << endl;
        }
        inFile.close();
    } else {
        cout << "Ошибка открытия файла для чтения" << endl;
    }
    
    // Дописывание в файл
    ofstream appFile("example.txt", ios::app); // Режим дописывания
    
    if (appFile.is_open()) {
        appFile << "Дописанная строка" << endl;
        appFile.close();
        cout << "Дописывание завершено" << endl;
    }
    
    return 0;
}
```

**Особенности ввода-вывода:**

1. **cin**: Читает до первого пробела, оставляет символы в буфере
2. **getline**: Читает всю строку, включая пробелы
3. **cin.ignore()**: Очищает буфер ввода
4. **Файловые потоки**: `ifstream` для чтения, `ofstream` для записи
5. **Режимы открытия**: `ios::app` - дописывание, `ios::binary` - бинарный режим

## 9. Область видимости (Scope)

### Локальная и глобальная область видимости

```cpp
#include <iostream>
using namespace std;

// Глобальная переменная - видна во всей программе
int globalVar = 100;

void testFunction() {
    // Локальная переменная функции
    int localVar = 50;
    cout << "В функции: globalVar = " << globalVar << endl;
    cout << "В функции: localVar = " << localVar << endl;
    
    // Можно изменить глобальную переменную
    globalVar = 200;
}

int main() {
    // Локальная переменная main
    int localVar = 10;
    
    cout << "В main: globalVar = " << globalVar << endl;
    cout << "В main: localVar = " << localVar << endl;
    
    testFunction();
    
    cout << "После функции: globalVar = " << globalVar << endl;
    
    // Блоки кода создают свою область видимости
    {
        int blockVar = 999;
        cout << "В блоке: blockVar = " << blockVar << endl;
        cout << "В блоке: localVar = " << localVar << endl; // Доступна
    }
    
    // cout << blockVar; // ОШИБКА: blockVar не видна вне блока
    
    return 0;
}
```

### Статические переменные

```cpp
#include <iostream>
using namespace std;

void counter() {
    // Статическая переменная - сохраняет значение между вызовами
    static int count = 0;
    count++;
    cout << "Вызов номер: " << count << endl;
}

int main() {
    counter(); // 1
    counter(); // 2
    counter(); // 3
    
    return 0;
}
```

### Область видимости в циклах и условиях

```cpp
#include <iostream>
using namespace std;

int main() {
    int x = 5;
    
    if (x > 0) {
        int y = 10; // Видна только внутри блока if
        cout << "x = " << x << ", y = " << y << endl;
    }
    
    // cout << y; // ОШИБКА: y не видна вне блока if
    
    for (int i = 0; i < 3; i++) {
        int loopVar = i * 2; // Видна только внутри цикла
        cout << "i = " << i << ", loopVar = " << loopVar << endl;
    }
    
    // cout << i; // ОШИБКА: i не видна вне цикла
    // cout << loopVar; // ОШИБКА: loopVar не видна вне цикла
    
    return 0;
}
```

### Пространства имен

```cpp
#include <iostream>
using namespace std;

// Собственное пространство имен
namespace MyMath {
    const double PI = 3.14159;
    
    double square(double x) {
        return x * x;
    }
    
    double cube(double x) {
        return x * x * x;
    }
}

namespace Geometry {
    const double PI = 3.14; // Другое пространство имен - другое PI
    
    double circleArea(double radius) {
        return PI * radius * radius;
    }
}

int main() {
    // Использование пространств имен
    cout << "MyMath::PI = " << MyMath::PI << endl;
    cout << "Geometry::PI = " << Geometry::PI << endl;
    
    cout << "Квадрат 5: " << MyMath::square(5) << endl;
    cout << "Куб 3: " << MyMath::cube(3) << endl;
    cout << "Площадь круга: " << Geometry::circleArea(2) << endl;
    
    // Можно создать псевдоним
    namespace MM = MyMath;
    cout << "Через псевдоним: " << MM::square(4) << endl;
    
    return 0;
}
```

**Правила области видимости:**

1. **Локальные переменные**: Видны только в своем блоке кода
2. **Глобальные переменные**: Видны везде, но их использование не рекомендуется
3. **Статические переменные**: Сохраняют значение между вызовами функции
4. **Пространства имен**: Позволяют избежать конфликтов имен
5. **Приоритет**: Локальные переменные имеют приоритет над глобальными

## Заключение

Этот материал охватывает фундаментальные основы C++, которые являются критически важными для дальнейшего изучения языка. Каждая тема содержит практические примеры и объяснения потенциально сложных моментов.

**Рекомендации:**
Практикуйтесь написанием кода, экспериментируйте с примерами и не бойтесь совершать ошибки - это лучший способ обучения!
