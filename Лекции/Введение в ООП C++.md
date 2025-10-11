# Объектно-Ориентированное Программирование (ООП) в C++

## Оглавление
1. [Введение в ООП](#введение-в-ооп)
2. [Классы и объекты](#классы-и-объекты)
3. [Инкапсуляция](#инкапсуляция)
4. [Конструкторы и деструкторы](#конструкторы-и-деструкторы)
5. [Наследование](#наследование)
6. [Полиморфизм](#полиморфизм)
7. [Абстрактные классы](#абстрактные-классы)
8. [Дружественные функции и классы](#дружественные-функции-и-классы)
9. [Статические члены класса](#статические-члены-класса)
10. [Шаблоны классов](#шаблоны-классов)

---

## Введение в ООП

**Объектно-Ориентированное Программирование** — это парадигма программирования, в которой основными концепциями являются объекты и классы.

### Основные принципы ООП:

1. **Инкапсуляция** - объединение данных и методов в одной единице (классе)
2. **Наследование** - создание новых классов на основе существующих
3. **Полиморфизм** - возможность объектов с одинаковой спецификацией иметь различную реализацию
4. **Абстракция** - выделение существенных характеристик объекта

---

## Классы и объекты

### Что такое класс?
**Класс** - это пользовательский тип данных, который содержит:
- Данные (поля, атрибуты)
- Методы (функции) для работы с этими данными

### Что такое объект?
**Объект** - это экземпляр класса.

### Синтаксис объявления класса:

```cpp
class ClassName {
private:
    // приватные члены (доступны только внутри класса)
    
protected:
    // защищенные члены (доступны внутри класса и производных классов)
    
public:
    // публичные члены (доступны извне)
};
```

### Пример:

```cpp
#include <iostream>
#include <string>
using namespace std;

class Student {
private:
    string name;
    int age;
    double averageGrade;

public:
    // Методы для работы с приватными данными
    void setName(string studentName) {
        name = studentName;
    }
    
    string getName() {
        return name;
    }
    
    void setAge(int studentAge) {
        if (studentAge > 0 && studentAge < 150) {
            age = studentAge;
        }
    }
    
    int getAge() {
        return age;
    }
    
    void displayInfo() {
        cout << "Студент: " << name << ", Возраст: " << age 
             << ", Средний балл: " << averageGrade << endl;
    }
};
```

### Вопрос: Почему использовать приватные поля?
**Ответ:** Для защиты данных от некорректного изменения и обеспечения контроля доступа.

---

## Инкапсуляция

**Инкапсуляция** - это механизм, который объединяет данные и методы, манипулирующие этими данными, и защищает их от внешнего вмешательства.

### Пример с полной инкапсуляцией:

```cpp
class BankAccount {
private:
    string accountNumber;
    double balance;
    string ownerName;

public:
    BankAccount(string accNum, string owner, double initialBalance) {
        accountNumber = accNum;
        ownerName = owner;
        balance = initialBalance;
    }
    
    // Геттеры (методы для получения значений)
    string getAccountNumber() const {
        return accountNumber;
    }
    
    double getBalance() const {
        return balance;
    }
    
    string getOwnerName() const {
        return ownerName;
    }
    
    // Сеттеры (методы для установки значений с проверкой)
    void setOwnerName(string newName) {
        if (!newName.empty()) {
            ownerName = newName;
        }
    }
    
    // Бизнес-методы
    void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
            cout << "Внесено: " << amount << ". Новый баланс: " << balance << endl;
        }
    }
    
    bool withdraw(double amount) {
        if (amount > 0 && amount <= balance) {
            balance -= amount;
            cout << "Снято: " << amount << ". Новый баланс: " << balance << endl;
            return true;
        }
        cout << "Недостаточно средств!" << endl;
        return false;
    }
};
```

### Вопрос: Что такое const методы?
**Ответ:** Методы, помеченные const, не могут изменять поля класса (кроме mutable полей). Они гарантируют, что объект не будет изменен.

---

## Конструкторы и деструкторы

### Конструкторы
**Конструктор** - специальный метод, который вызывается при создании объекта.

#### Типы конструкторов:

1. **Конструктор по умолчанию**
2. **Параметризованный конструктор**
3. **Конструктор копирования**
4. **Конструктор перемещения** (C++11)

```cpp
class Rectangle {
private:
    double width;
    double height;

public:
    // 1. Конструктор по умолчанию
    Rectangle() : width(1.0), height(1.0) {
        cout << "Вызван конструктор по умолчанию" << endl;
    }
    
    // 2. Параметризованный конструктор
    Rectangle(double w, double h) : width(w), height(h) {
        cout << "Вызван параметризованный конструктор" << endl;
    }
    
    // 3. Конструктор копирования
    Rectangle(const Rectangle& other) : width(other.width), height(other.height) {
        cout << "Вызван конструктор копирования" << endl;
    }
    
    // 4. Конструктор перемещения (C++11)
    Rectangle(Rectangle&& other) noexcept 
        : width(move(other.width)), height(move(other.height)) {
        cout << "Вызван конструктор перемещения" << endl;
    }
    
    // Деструктор
    ~Rectangle() {
        cout << "Вызван деструктор" << endl;
    }
    
    // Методы
    double getArea() const {
        return width * height;
    }
    
    double getPerimeter() const {
        return 2 * (width + height);
    }
};
```

### Список инициализации
**Важный момент:** Всегда используйте список инициализации для инициализации полей!

```cpp
// ПРАВИЛЬНО
Rectangle(double w, double h) : width(w), height(h) {}

// НЕПРАВИЛЬНО (сначала вызываются конструкторы по умолчанию, потом присваивание)
Rectangle(double w, double h) {
    width = w;  // Это присваивание, а не инициализация!
    height = h;
}
```

### Правило трех/пяти
Если классу нужен один из следующих методов, то обычно нужны все три/пять:

**Правило трех:**
- Деструктор
- Конструктор копирования
- Оператор присваивания копированием

**Правило пяти** (C++11):
- Деструктор
- Конструктор копирования
- Оператор присваивания копированием
- Конструктор перемещения
- Оператор присваивания перемещением

---

## Наследование

**Наследование** - механизм создания нового класса на основе существующего.

### Синтаксис наследования:

```cpp
class BaseClass {
    // базовая реализация
};

class DerivedClass : access-specifier BaseClass {
    // расширенная реализация
};
```

### Уровни доступа при наследовании:

```cpp
class Base {
public:
    int publicVar;
protected:
    int protectedVar;
private:
    int privateVar;
};

// public наследование
class DerivedPublic : public Base {
    // publicVar остается public
    // protectedVar остается protected
    // privateVar НЕДОСТУПНА
};

// protected наследование  
class DerivedProtected : protected Base {
    // publicVar становится protected
    // protectedVar остается protected
    // privateVar НЕДОСТУПНА
};

// private наследование
class DerivedPrivate : private Base {
    // publicVar становится private
    // protectedVar становится private
    // privateVar НЕДОСТУПНА
};
```

### Практический пример наследования:

```cpp
#include <iostream>
#include <string>
using namespace std;

// Базовый класс
class Animal {
protected:
    string name;
    int age;
    string species;

public:
    Animal(string n, int a, string s) : name(n), age(a), species(s) {}
    
    virtual void makeSound() const {
        cout << "Животное издает звук" << endl;
    }
    
    void sleep() const {
        cout << name << " спит" << endl;
    }
    
    void eat() const {
        cout << name << " ест" << endl;
    }
    
    virtual void displayInfo() const {
        cout << "Имя: " << name << ", Возраст: " << age 
             << ", Вид: " << species << endl;
    }
    
    virtual ~Animal() {}  // Виртуальный деструктор
};

// Производный класс
class Dog : public Animal {
private:
    string breed;
    bool isTrained;

public:
    Dog(string n, int a, string b, bool trained) 
        : Animal(n, a, "Собака"), breed(b), isTrained(trained) {}
    
    // Переопределение виртуального метода
    void makeSound() const override {
        cout << name << " гавкает: Гав-гав!" << endl;
    }
    
    void displayInfo() const override {
        Animal::displayInfo();  // Вызов метода базового класса
        cout << "Порода: " << breed << ", Дрессирована: " 
             << (isTrained ? "Да" : "Нет") << endl;
    }
    
    void fetch() const {
        cout << name << " приносит палку!" << endl;
    }
};

// Еще один производный класс
class Cat : public Animal {
private:
    string furColor;
    bool isIndoor;

public:
    Cat(string n, int a, string color, bool indoor) 
        : Animal(n, a, "Кошка"), furColor(color), isIndoor(indoor) {}
    
    void makeSound() const override {
        cout << name << " мяукает: Мяу-мяу!" << endl;
    }
    
    void displayInfo() const override {
        Animal::displayInfo();
        cout << "Цвет шерсти: " << furColor << ", Домашняя: " 
             << (isIndoor ? "Да" : "Нет") << endl;
    }
    
    void climb() const {
        cout << name << " лазает по деревьям!" << endl;
    }
};
```

### Вопрос: Когда использовать виртуальные деструкторы?
**Ответ:** Всегда, когда класс предназначен для наследования! Это предотвращает утечки памяти при удалении объектов через указатель на базовый класс.

---

## Полиморфизм

**Полиморфизм** - возможность объектов разных классов обрабатываться как объекты общего базового класса.

### Статический полиморфизм (перегрузка функций):

```cpp
class Calculator {
public:
    int add(int a, int b) {
        return a + b;
    }
    
    double add(double a, double b) {
        return a + b;
    }
    
    string add(string a, string b) {
        return a + b;
    }
};
```

### Динамический полиморфизм (виртуальные функции):

```cpp
class Shape {
protected:
    string color;

public:
    Shape(string c) : color(c) {}
    
    // Чисто виртуальная функция - делает класс абстрактным
    virtual double getArea() const = 0;
    
    // Виртуальная функция с реализацией по умолчанию
    virtual void draw() const {
        cout << "Рисую фигуру цвета " << color << endl;
    }
    
    virtual ~Shape() {}
};

class Circle : public Shape {
private:
    double radius;

public:
    Circle(string c, double r) : Shape(c), radius(r) {}
    
    double getArea() const override {
        return 3.14159 * radius * radius;
    }
    
    void draw() const override {
        cout << "Рисую круг радиуса " << radius 
             << " цвета " << color << endl;
    }
};

class Rectangle : public Shape {
private:
    double width;
    double height;

public:
    Rectangle(string c, double w, double h) : Shape(c), width(w), height(h) {}
    
    double getArea() const override {
        return width * height;
    }
    
    void draw() const override {
        cout << "Рисую прямоугольник " << width << "x" << height 
             << " цвета " << color << endl;
    }
};
```

### Использование полиморфизма:

```cpp
int main() {
    // Массив указателей на базовый класс
    Shape* shapes[3];
    
    shapes[0] = new Circle("Красный", 5.0);
    shapes[1] = new Rectangle("Синий", 4.0, 6.0);
    shapes[2] = new Circle("Зеленый", 3.0);
    
    // Полиморфное поведение
    for (int i = 0; i < 3; ++i) {
        shapes[i]->draw();
        cout << "Площадь: " << shapes[i]->getArea() << endl;
        cout << "---" << endl;
    }
    
    // Освобождение памяти
    for (int i = 0; i < 3; ++i) {
        delete shapes[i];
    }
    
    return 0;
}
```

### Вопрос: override vs final
**Ответ:**
- `override` - явно указывает, что метод переопределяет виртуальный метод базового класса
- `final` - запрещает дальнейшее переопределение метода в производных классах

---

## Абстрактные классы

**Абстрактный класс** - класс, содержащий хотя бы одну чисто виртуальную функцию. Нельзя создать объект абстрактного класса.

```cpp
class Vehicle {
protected:
    string brand;
    int year;
    double maxSpeed;

public:
    Vehicle(string b, int y, double speed) 
        : brand(b), year(y), maxSpeed(speed) {}
    
    // Чисто виртуальные функции - делают класс абстрактным
    virtual void start() const = 0;
    virtual void stop() const = 0;
    virtual void displayInfo() const = 0;
    
    // Виртуальный деструктор
    virtual ~Vehicle() {}
};

class Car : public Vehicle {
private:
    int doors;
    string fuelType;

public:
    Car(string b, int y, double speed, int d, string fuel)
        : Vehicle(b, y, speed), doors(d), fuelType(fuel) {}
    
    void start() const override {
        cout << "Автомобиль " << brand << " заводится с ключа" << endl;
    }
    
    void stop() const override {
        cout << "Автомобиль " << brand << " останавливается" << endl;
    }
    
    void displayInfo() const override {
        cout << "Автомобиль: " << brand << ", Год: " << year 
             << ", Макс. скорость: " << maxSpeed << " км/ч" << endl;
        cout << "Количество дверей: " << doors 
             << ", Тип топлива: " << fuelType << endl;
    }
};

class Motorcycle : public Vehicle {
private:
    bool hasSidecar;

public:
    Motorcycle(string b, int y, double speed, bool sidecar)
        : Vehicle(b, y, speed), hasSidecar(sidecar) {}
    
    void start() const override {
        cout << "Мотоцикл " << brand << " заводится кнопкой" << endl;
    }
    
    void stop() const override {
        cout << "Мотоцикл " << brand << " останавливается" << endl;
    }
    
    void displayInfo() const override {
        cout << "Мотоцикл: " << brand << ", Год: " << year 
             << ", Макс. скорость: " << maxSpeed << " км/ч" << endl;
        cout << "Имеет коляску: " << (hasSidecar ? "Да" : "Нет") << endl;
    }
    
    void wheelie() const {
        cout << "Мотоцикл " << brand << " делает вилли!" << endl;
    }
};
```

---

## Дружественные функции и классы

**Дружественные функции/классы** имеют доступ к приватным и защищенным членам класса.

```cpp
class Student {
private:
    string name;
    int age;
    double grades[5];
    double averageGrade;

    void calculateAverage() {
        double sum = 0;
        for (int i = 0; i < 5; ++i) {
            sum += grades[i];
        }
        averageGrade = sum / 5;
    }

public:
    Student(string n, int a) : name(n), age(a), averageGrade(0) {
        for (int i = 0; i < 5; ++i) {
            grades[i] = 0;
        }
    }
    
    // Дружественная функция
    friend void printStudentDetails(const Student& s);
    
    // Дружественный класс
    friend class StudentManager;
    
    void setGrade(int subject, double grade) {
        if (subject >= 0 && subject < 5 && grade >= 0 && grade <= 5) {
            grades[subject] = grade;
            calculateAverage();
        }
    }
};

// Дружественная функция
void printStudentDetails(const Student& s) {
    cout << "Имя: " << s.name << ", Возраст: " << s.age << endl;
    cout << "Оценки: ";
    for (int i = 0; i < 5; ++i) {
        cout << s.grades[i] << " ";
    }
    cout << endl << "Средний балл: " << s.averageGrade << endl;
}

// Дружественный класс
class StudentManager {
public:
    static void promoteStudent(Student& s) {
        s.age++;  // Доступ к приватному полю
        cout << s.name << " переведен на следующий курс!" << endl;
    }
    
    static void resetGrades(Student& s) {
        for (int i = 0; i < 5; ++i) {
            s.grades[i] = 0;  // Доступ к приватному полю
        }
        s.calculateAverage();  // Доступ к приватному методу
    }
};
```

### Вопрос: Когда использовать дружественные функции?
**Ответ:** Когда функции нужен доступ к приватным членам, но она не должна быть методом класса (например, операторы ввода/вывода).

---

## Статические члены класса

**Статические члены** принадлежат классу, а не конкретному объекту.

```cpp
class Employee {
private:
    string name;
    double salary;
    static int employeeCount;      // Статическая переменная
    static double totalSalary;     // Статическая переменная

public:
    Employee(string n, double s) : name(n), salary(s) {
        employeeCount++;
        totalSalary += salary;
    }
    
    ~Employee() {
        employeeCount--;
        totalSalary -= salary;
    }
    
    // Статические методы
    static int getEmployeeCount() {
        return employeeCount;
    }
    
    static double getAverageSalary() {
        return employeeCount > 0 ? totalSalary / employeeCount : 0;
    }
    
    static double getTotalSalary() {
        return totalSalary;
    }
    
    void displayInfo() const {
        cout << "Сотрудник: " << name << ", Зарплата: " << salary << endl;
    }
};

// Инициализация статических переменных
int Employee::employeeCount = 0;
double Employee::totalSalary = 0.0;
```

### Вопрос: Где инициализировать статические переменные?
**Ответ:** Вне класса, в глобальной области видимости.

---

## Шаблоны классов

**Шаблоны классов** позволяют создавать обобщенные классы.

```cpp
template<typename T>
class Stack {
private:
    T* data;
    int capacity;
    int topIndex;

public:
    Stack(int size = 10) : capacity(size), topIndex(-1) {
        data = new T[capacity];
    }
    
    ~Stack() {
        delete[] data;
    }
    
    // Конструктор копирования
    Stack(const Stack& other) : capacity(other.capacity), topIndex(other.topIndex) {
        data = new T[capacity];
        for (int i = 0; i <= topIndex; ++i) {
            data[i] = other.data[i];
        }
    }
    
    // Оператор присваивания
    Stack& operator=(const Stack& other) {
        if (this != &other) {
            delete[] data;
            capacity = other.capacity;
            topIndex = other.topIndex;
            data = new T[capacity];
            for (int i = 0; i <= topIndex; ++i) {
                data[i] = other.data[i];
            }
        }
        return *this;
    }
    
    void push(const T& value) {
        if (topIndex < capacity - 1) {
            data[++topIndex] = value;
        } else {
            cout << "Стек переполнен!" << endl;
        }
    }
    
    T pop() {
        if (topIndex >= 0) {
            return data[topIndex--];
        } else {
            cout << "Стек пуст!" << endl;
            return T();  // Возвращаем значение по умолчанию для типа T
        }
    }
    
    T top() const {
        if (topIndex >= 0) {
            return data[topIndex];
        } else {
            cout << "Стек пуст!" << endl;
            return T();
        }
    }
    
    bool isEmpty() const {
        return topIndex == -1;
    }
    
    int size() const {
        return topIndex + 1;
    }
};
```

### Использование шаблонов:

```cpp
int main() {
    // Стек для целых чисел
    Stack<int> intStack(5);
    intStack.push(10);
    intStack.push(20);
    intStack.push(30);
    
    cout << "Верхний элемент: " << intStack.top() << endl;
    cout << "Размер стека: " << intStack.size() << endl;
    
    // Стек для строк
    Stack<string> stringStack(3);
    stringStack.push("Hello");
    stringStack.push("World");
    stringStack.push("C++");
    
    while (!stringStack.isEmpty()) {
        cout << stringStack.pop() << " ";
    }
    cout << endl;
    
    return 0;
}
```

---

## Заключение

ООП в C++ предоставляет мощные инструменты для создания сложных программных систем. Ключевые моменты для запоминания:

1. **Всегда используйте инкапсуляцию** - защищайте данные класса
2. **Используйте списки инициализации** в конструкторах
3. **Соблюдайте правило пяти** для классов, управляющих ресурсами
4. **Используйте виртуальные деструкторы** в базовых классах
5. **Применяйте override** для явного указания переопределения методов
6. **Используйте const** для методов, которые не изменяют объект
7. **Рассмотрите шаблоны** для создания обобщенного кода

ООП требует практики - создавайте собственные классы, экспериментируйте с наследованием и полиморфизмом, и со временем эти концепции станут интуитивно понятными.
