## 1. Инкапсуляция (Encapsulation)

### Что это такое?
**Инкапсуляция** - это механизм объединения данных и методов, которые работают с этими данными, в одной единице (классе), и ограничение прямого доступа к внутренним данным извне.

### Детальное объяснение:

```cpp
#include <iostream>
#include <string>
using namespace std;

class BankAccount {
private: // ← Уровень доступа: закрытые члены
    string accountNumber;    // Нельзя получить напрямую извне
    double balance;         // Нельзя изменить напрямую извне
    string ownerName;       // Защищено от некорректных изменений

public: // ← Уровень доступа: публичные методы
    // Конструктор - инициализирует данные
    BankAccount(string accNum, string owner, double initialBalance) {
        // Валидация данных при создании
        if (initialBalance < 0) {
            throw invalid_argument("Баланс не может быть отрицательным");
        }
        accountNumber = accNum;
        ownerName = owner;
        balance = initialBalance;
    }
    
    // Геттеры - контролируемый доступ для чтения
    string getAccountNumber() const {
        return accountNumber;
    }
    
    double getBalance() const {
        return balance;
    }
    
    string getOwnerName() const {
        return ownerName;
    }
    
    // Сеттеры - контролируемый доступ для записи
    void setOwnerName(string newName) {
        if (!newName.empty() && newName.length() >= 2) {
            ownerName = newName;
        } else {
            cout << "Ошибка: Имя владельца слишком короткое" << endl;
        }
    }
    
    // Бизнес-методы - единственный способ изменить баланс
    void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
            cout << "Внесено: " << amount << ". Новый баланс: " << balance << endl;
        } else {
            cout << "Ошибка: Сумма депозита должна быть положительной" << endl;
        }
    }
    
    bool withdraw(double amount) {
        if (amount > 0 && amount <= balance) {
            balance -= amount;
            cout << "Снято: " << amount << ". Новый баланс: " << balance << endl;
            return true;
        } else {
            cout << "Ошибка: Недостаточно средств или неверная сумма" << endl;
            return false;
        }
    }
    
    void displayInfo() const {
        cout << "Счет: " << accountNumber << endl;
        cout << "Владелец: " << ownerName << endl;
        cout << "Баланс: " << balance << endl;
    }
};
```

### Почему это важно?

**Без инкапсуляции:**
```cpp
// ПЛОХОЙ ПРИМЕР - нет инкапсуляции
struct BankAccountBad {
    string accountNumber;
    double balance;  // Любой может изменить напрямую!
    string ownerName;
};

int main() {
    BankAccountBad acc;
    acc.balance = -1000;  // Некорректные данные!
    acc.accountNumber = "";  // Пустой номер счета!
}
```

**С инкапсуляцией:**
```cpp
int main() {
    BankAccount acc("123456", "Иван Иванов", 1000);
    
    // acc.balance = -1000;  // ОШИБКА КОМПИЛЯЦИИ - приватное поле!
    // acc.accountNumber = "999";  // ОШИБКА - нельзя изменить напрямую!
    
    acc.deposit(500);     // OK - через контролируемый метод
    acc.withdraw(200);    // OK - с проверкой
    acc.setOwnerName("Петр Петров");  // OK - с валидацией
    
    cout << "Баланс: " << acc.getBalance() << endl;  // OK - только чтение
}
```

### Преимущества инкапсуляции:
1. **Защита данных** - предотвращение некорректных состояний
2. **Гибкость** - можно менять внутреннюю реализацию без влияния на внешний код
3. **Контроль доступа** - разные уровни доступа для разных частей программы
4. **Упрощение использования** - пользователь класса не должен знать о внутренней реализации

---

## 2. Наследование (Inheritance)

### Что это такое?
**Наследование** - это механизм создания нового класса на основе существующего, с возможностью reuse (повторного использования) и расширения функциональности.

### Детальное объяснение:

```cpp
#include <iostream>
#include <string>
using namespace std;

// Базовый класс (родительский)
class Animal {
protected: // ← Защищенные члены - доступны в производных классах
    string name;
    int age;
    double weight;

public:
    // Конструктор базового класса
    Animal(string n, int a, double w) : name(n), age(a), weight(w) {
        cout << "Создан объект Animal: " << name << endl;
    }
    
    // Виртуальный деструктор - ОЧЕНЬ ВАЖНО для полиморфизма!
    virtual ~Animal() {
        cout << "Уничтожен объект Animal: " << name << endl;
    }
    
    // Виртуальные методы - могут быть переопределены в производных классах
    virtual void makeSound() const {
        cout << name << " издает неопределенный звук" << endl;
    }
    
    virtual void move() const {
        cout << name << " перемещается" << endl;
    }
    
    // Невиртуальные методы - наследуются как есть
    void sleep() const {
        cout << name << " спит" << endl;
    }
    
    void eat() const {
        cout << name << " ест" << endl;
    }
    
    // Геттеры
    string getName() const { return name; }
    int getAge() const { return age; }
    double getWeight() const { return weight; }
    
    // Сеттеры
    void setAge(int a) { 
        if (a >= 0) age = a; 
    }
    
    void setWeight(double w) { 
        if (w > 0) weight = w; 
    }
};

// Производный класс (дочерний) - наследование public
class Dog : public Animal {
private:
    string breed;
    bool isTrained;

public:
    // Конструктор производного класса
    Dog(string n, int a, double w, string b, bool trained) 
        : Animal(n, a, w),  // Вызов конструктора базового класса
          breed(b), isTrained(trained) {
        cout << "Создан объект Dog: " << name << endl;
    }
    
    ~Dog() {
        cout << "Уничтожен объект Dog: " << name << endl;
    }
    
    // Переопределение виртуальных методов
    void makeSound() const override {
        cout << name << " гавкает: Гав-гав!" << endl;
    }
    
    void move() const override {
        cout << name << " бегает на четырех лапах" << endl;
    }
    
    // Новые методы, специфичные для Dog
    void fetch() const {
        cout << name << " приносит палку!" << endl;
    }
    
    void guard() const {
        if (isTrained) {
            cout << name << " охраняет территорию" << endl;
        }
    }
    
    // Геттеры для новых полей
    string getBreed() const { return breed; }
    bool getIsTrained() const { return isTrained; }
};

// Еще один производный класс
class Bird : public Animal {
private:
    double wingspan;
    bool canFly;

public:
    Bird(string n, int a, double w, double wings, bool fly) 
        : Animal(n, a, w), wingspan(wings), canFly(fly) {
        cout << "Создан объект Bird: " << name << endl;
    }
    
    ~Bird() {
        cout << "Уничтожен объект Bird: " << name << endl;
    }
    
    // Переопределение виртуальных методов
    void makeSound() const override {
        cout << name << " поет: Чик-чирик!" << endl;
    }
    
    void move() const override {
        if (canFly) {
            cout << name << " летает с размахом крыльев " << wingspan << " см" << endl;
        } else {
            cout << name << " прыгает по земле" << endl;
        }
    }
    
    // Новые методы, специфичные для Bird
    void buildNest() const {
        cout << name << " строит гнездо" << endl;
    }
    
    // Геттеры
    double getWingspan() const { return wingspan; }
    bool getCanFly() const { return canFly; }
};
```

### Типы наследования:

```cpp
class Base {
public:
    int publicVar;
protected:
    int protectedVar;
private:
    int privateVar;
};

// Public наследование - наиболее распространенное
class DerivedPublic : public Base {
    // publicVar остается public
    // protectedVar остается protected  
    // privateVar НЕДОСТУПНА
};

// Protected наследование
class DerivedProtected : protected Base {
    // publicVar становится protected
    // protectedVar остается protected
    // privateVar НЕДОСТУПНА
};

// Private наследование
class DerivedPrivate : private Base {
    // publicVar становится private
    // protectedVar становится private
    // privateVar НЕДОСТУПНА
};
```

### Использование наследования:

```cpp
int main() {
    // Создание объектов разных классов
    Dog dog("Бобик", 3, 15.5, "Овчарка", true);
    Bird bird("Кеша", 2, 0.5, 25.0, true);
    
    cout << "\n=== Использование методов ===" << endl;
    
    // Методы базового класса доступны всем производным
    dog.sleep();    // Унаследован от Animal
    dog.eat();      // Унаследован от Animal
    dog.fetch();    // Специфичен для Dog
    dog.guard();    // Специфичен для Dog
    
    bird.sleep();   // Унаследован от Animal  
    bird.eat();     // Унаследован от Animal
    bird.buildNest(); // Специфичен для Bird
    
    cout << "\n=== Полиморфное поведение ===" << endl;
    Animal* animals[] = {&dog, &bird};
    
    for (int i = 0; i < 2; i++) {
        animals[i]->makeSound();  // Вызовется переопределенная версия!
        animals[i]->move();       // Вызовется переопределенная версия!
        cout << "---" << endl;
    }
    
    return 0;
}
```

### Преимущества наследования:
1. **Повторное использование кода** - не нужно дублировать общую функциональность
2. **Расширяемость** - можно добавлять новую функциональность в производных классах
3. **Иерархия классов** - создание логических отношений между классами
4. **Полиморфизм** - основа для полиморфного поведения

---

## 3. Полиморфизм (Polymorphism)

### Что это такое?
**Полиморфизм** - это возможность объектов разных классов с общей базой обрабатываться единообразно, но выполнять специфичные для каждого класса действия.

### Детальное объяснение:

```cpp
#include <iostream>
#include <vector>
#include <memory>
using namespace std;

// Абстрактный базовый класс
class Shape {
protected:
    string color;
    string name;

public:
    Shape(string n, string c) : name(n), color(c) {}
    
    // Виртуальный деструктор - ОБЯЗАТЕЛЬНО для полиморфизма!
    virtual ~Shape() {}
    
    // Чисто виртуальные функции - делают класс абстрактным
    virtual double calculateArea() const = 0;
    virtual double calculatePerimeter() const = 0;
    
    // Виртуальная функция с реализацией по умолчанию
    virtual void draw() const {
        cout << "Рисую " << name << " цвета " << color << endl;
    }
    
    // Невиртуальная функция
    void displayInfo() const {
        cout << "Фигура: " << name << ", Цвет: " << color << endl;
        cout << "Площадь: " << calculateArea() << endl;
        cout << "Периметр: " << calculatePerimeter() << endl;
    }
    
    // Виртуальная функция, которая может быть переопределена
    virtual void scale(double factor) {
        cout << "Масштабирование " << name << " в " << factor << " раз" << endl;
    }
};

// Конкретный класс - круг
class Circle : public Shape {
private:
    double radius;

public:
    Circle(string c, double r) : Shape("Круг", c), radius(r) {}
    
    // Переопределение чисто виртуальных функций
    double calculateArea() const override {
        return 3.14159 * radius * radius;
    }
    
    double calculatePerimeter() const override {
        return 2 * 3.14159 * radius;
    }
    
    // Переопределение виртуальной функции
    void draw() const override {
        cout << "Рисую круг радиусом " << radius 
             << " цвета " << color << endl;
    }
    
    void scale(double factor) override {
        radius *= factor;
        cout << "Новый радиус круга: " << radius << endl;
    }
    
    double getRadius() const { return radius; }
};

// Конкретный класс - прямоугольник
class Rectangle : public Shape {
private:
    double width;
    double height;

public:
    Rectangle(string c, double w, double h) 
        : Shape("Прямоугольник", c), width(w), height(h) {}
    
    double calculateArea() const override {
        return width * height;
    }
    
    double calculatePerimeter() const override {
        return 2 * (width + height);
    }
    
    void draw() const override {
        cout << "Рисую прямоугольник " << width << "x" << height 
             << " цвета " << color << endl;
    }
    
    void scale(double factor) override {
        width *= factor;
        height *= factor;
        cout << "Новые размеры прямоугольника: " 
             << width << "x" << height << endl;
    }
    
    double getWidth() const { return width; }
    double getHeight() const { return height; }
};

// Конкретный класс - треугольник
class Triangle : public Shape {
private:
    double sideA, sideB, sideC;

public:
    Triangle(string c, double a, double b, double c) 
        : Shape("Треугольник", c), sideA(a), sideB(b), sideC(c) {}
    
    double calculateArea() const override {
        // Формула Герона
        double p = calculatePerimeter() / 2;
        return sqrt(p * (p - sideA) * (p - sideB) * (p - sideC));
    }
    
    double calculatePerimeter() const override {
        return sideA + sideB + sideC;
    }
    
    void draw() const override {
        cout << "Рисую треугольник со сторонами " << sideA << ", " 
             << sideB << ", " << sideC << " цвета " << color << endl;
    }
    
    void scale(double factor) override {
        sideA *= factor;
        sideB *= factor;
        sideC *= factor;
        cout << "Новые стороны треугольника: " << sideA << ", " 
             << sideB << ", " << sideC << endl;
    }
};
```

### Демонстрация полиморфизма:

```cpp
// Функция, работающая с базовым классом
void processShape(Shape& shape) {
    shape.displayInfo();
    shape.draw();
    cout << "---" << endl;
}

// Функция, принимающая указатель на базовый класс
void scaleShape(Shape* shape, double factor) {
    shape->scale(factor);
    cout << "Новая площадь: " << shape->calculateArea() << endl;
    cout << "---" << endl;
}

int main() {
    cout << "=== ДЕМОНСТРАЦИЯ ПОЛИМОРФИЗМА ===" << endl;
    
    // Создание объектов разных классов
    Circle circle("Красный", 5.0);
    Rectangle rectangle("Синий", 4.0, 6.0);
    Triangle triangle("Зеленый", 3.0, 4.0, 5.0);
    
    cout << "\n=== Прямая работа с объектами ===" << endl;
    processShape(circle);
    processShape(rectangle);
    processShape(triangle);
    
    cout << "\n=== Работа через указатели на базовый класс ===" << endl;
    vector<Shape*> shapes = {&circle, &rectangle, &triangle};
    
    for (Shape* shape : shapes) {
        shape->displayInfo();
        shape->draw();
        cout << "---" << endl;
    }
    
    cout << "\n=== Масштабирование фигур ===" << endl;
    for (Shape* shape : shapes) {
        scaleShape(shape, 1.5);  // Увеличиваем все фигуры в 1.5 раза
    }
    
    cout << "\n=== Использование умных указателей ===" << endl;
    vector<unique_ptr<Shape>> shapeCollection;
    shapeCollection.push_back(make_unique<Circle>("Желтый", 3.0));
    shapeCollection.push_back(make_unique<Rectangle>("Фиолетовый", 2.0, 3.0));
    shapeCollection.push_back(make_unique<Triangle>("Оранжевый", 5.0, 6.0, 7.0));
    
    for (auto& shape : shapeCollection) {
        shape->displayInfo();
        shape->draw();
        cout << "---" << endl;
    }
    
    return 0;
}
```

### Ключевые моменты полиморфизма:

1. **Виртуальные функции** - позволяют переопределение в производных классах
2. **Чисто виртуальные функции** - делают класс абстрактным
3. **Виртуальный деструктор** - обязателен для корректного удаления объектов
4. **override** - явное указание переопределения (рекомендуется всегда использовать)
5. **final** - запрет дальнейшего переопределения

### Преимущества полиморфизма:
1. **Единообразие обработки** - один интерфейс для разных типов объектов
2. **Расширяемость** - можно добавлять новые классы без изменения существующего кода
3. **Гибкость** - поведение определяется во время выполнения (runtime)
4. **Упрощение кода** - меньше условных конструкций

---

## 4. Абстракция (Abstraction)

### Что это такое?
**Абстракция** - это процесс выделения существенных характеристик объекта и игнорирования несущественных деталей.

### Детальное объяснение:

```cpp
#include <iostream>
#include <string>
#include <vector>
using namespace std;

// Абстрактный класс - представляет общую концепцию устройства
class Device {
protected:
    string manufacturer;
    string model;
    bool isPoweredOn;

public:
    Device(string manuf, string mod) 
        : manufacturer(manuf), model(mod), isPoweredOn(false) {}
    
    virtual ~Device() {}
    
    // Чисто виртуальные функции - определяют интерфейс
    virtual void powerOn() = 0;
    virtual void powerOff() = 0;
    virtual void performMainFunction() = 0;
    
    // Общие методы для всех устройств
    virtual void displayInfo() const {
        cout << "Устройство: " << manufacturer << " " << model << endl;
        cout << "Состояние: " << (isPoweredOn ? "ВКЛ" : "ВЫКЛ") << endl;
    }
    
    string getManufacturer() const { return manufacturer; }
    string getModel() const { return model; }
    bool getIsPoweredOn() const { return isPoweredOn; }
};

// Конкретная реализация - принтер
class Printer : public Device {
private:
    int paperCount;
    int inkLevel;

public:
    Printer(string manuf, string mod, int paper, int ink) 
        : Device(manuf, mod), paperCount(paper), inkLevel(ink) {}
    
    void powerOn() override {
        if (!isPoweredOn) {
            isPoweredOn = true;
            cout << "Принтер " << model << " включен" << endl;
            selfTest();
        }
    }
    
    void powerOff() override {
        if (isPoweredOn) {
            isPoweredOn = false;
            cout << "Принтер " << model << " выключен" << endl;
        }
    }
    
    void performMainFunction() override {
        if (!isPoweredOn) {
            cout << "Ошибка: Принтер выключен!" << endl;
            return;
        }
        
        if (paperCount > 0 && inkLevel > 10) {
            cout << "Печатаю документ..." << endl;
            paperCount--;
            inkLevel -= 5;
            cout << "Бумага осталось: " << paperCount << endl;
            cout << "Уровень чернил: " << inkLevel << "%" << endl;
        } else {
            cout << "Ошибка: Проверьте бумагу или чернила!" << endl;
        }
    }
    
    void displayInfo() const override {
        Device::displayInfo();
        cout << "Бумага: " << paperCount << " листов" << endl;
        cout << "Чернила: " << inkLevel << "%" << endl;
    }
    
private:
    void selfTest() {
        cout << "Выполняется самотестирование принтера..." << endl;
        cout << "Все системы работают нормально" << endl;
    }
};

// Конкретная реализация - сканер
class Scanner : public Device {
private:
    int resolutionDPI;
    string scanType;

public:
    Scanner(string manuf, string mod, int res, string type) 
        : Device(manuf, mod), resolutionDPI(res), scanType(type) {}
    
    void powerOn() override {
        if (!isPoweredOn) {
            isPoweredOn = true;
            cout << "Сканер " << model << " включен" << endl;
            calibrate();
        }
    }
    
    void powerOff() override {
        if (isPoweredOn) {
            isPoweredOn = false;
            cout << "Сканер " << model << " выключен" << endl;
        }
    }
    
    void performMainFunction() override {
        if (!isPoweredOn) {
            cout << "Ошибка: Сканер выключен!" << endl;
            return;
        }
        
        cout << "Сканирую документ..." << endl;
        cout << "Разрешение: " << resolutionDPI << " DPI" << endl;
        cout << "Тип сканирования: " << scanType << endl;
        cout << "Сканирование завершено!" << endl;
    }
    
    void displayInfo() const override {
        Device::displayInfo();
        cout << "Разрешение: " << resolutionDPI << " DPI" << endl;
        cout << "Тип сканирования: " << scanType << endl;
    }
    
    // Специфичный метод для сканера
    void setResolution(int newResolution) {
        if (newResolution > 0) {
            resolutionDPI = newResolution;
            cout << "Разрешение установлено: " << resolutionDPI << " DPI" << endl;
        }
    }
    
private:
    void calibrate() {
        cout << "Калибровка сканера..." << endl;
        cout << "Калибровка завершена" << endl;
    }
};
```

### Использование абстракции:

```cpp
// Функция, работающая на уровне абстракции "Устройство"
void manageDevice(Device& device) {
    cout << "\n=== Управление устройством ===" << endl;
    device.displayInfo();
    device.powerOn();
    device.performMainFunction();
    device.powerOff();
}

int main() {
    cout << "=== ДЕМОНСТРАЦИЯ АБСТРАКЦИИ ===" << endl;
    
    // Создание конкретных устройств
    Printer printer("Canon", "PIXMA MG304", 50, 80);
    Scanner scanner("Epson", "Perfection V39", 1200, "Цветное");
    
    // Работа через абстрактный интерфейс
    manageDevice(printer);
    manageDevice(scanner);
    
    cout << "\n=== Работа через коллекцию устройств ===" << endl;
    vector<Device*> devices = {&printer, &scanner};
    
    for (Device* device : devices) {
        cout << "\n--- Обработка устройства ---" << endl;
        device->powerOn();
        device->performMainFunction();
        
        // Попытка доступа к специфичным методам требует dynamic_cast
        if (Scanner* scannerPtr = dynamic_cast<Scanner*>(device)) {
            scannerPtr->setResolution(2400);  // Только для сканеров
        }
        
        device->displayInfo();
    }
    
    // Выключение всех устройств
    cout << "\n=== Выключение всех устройств ===" << endl;
    for (Device* device : devices) {
        device->powerOff();
    }
    
    return 0;
}
```

### Преимущества абстракции:

1. **Сокрытие сложности** - пользователь видит только необходимый интерфейс
2. **Упрощение использования** - не нужно знать внутреннюю реализацию
3. **Стандартизация** - общий интерфейс для родственных классов
4. **Снижение耦合ности** - зависимости только от абстракций, а не от конкретных реализаций

---

## Связь всех принципов ООП

Все четыре принципа тесно связаны между собой:

- **Инкапсуляция** защищает внутреннее состояние объектов
- **Наследование** создает иерархии классов и повторно использует код  
- **Полиморфизм** обеспечивает единообразную работу с объектами разных классов
- **Абстракция** определяет чистые интерфейсы и скрывает сложность

### Пример, объединяющий все принципы:

```cpp
// Абстракция + Инкапсуляция
class Vehicle {
protected:
    string brand;
    string model;
    int year;
    double speed;  // Инкапсуляция
    
public:
    Vehicle(string b, string m, int y) : brand(b), model(m), year(y), speed(0) {}
    virtual ~Vehicle() {}
    
    // Абстрактный интерфейс
    virtual void start() = 0;
    virtual void stop() = 0;
    virtual void accelerate(double amount) = 0;
    
    // Инкапсуляция - контролируемый доступ
    double getSpeed() const { return speed; }
    void setSpeed(double s) { 
        if (s >= 0) speed = s; 
    }
    
    virtual void displayInfo() const {
        cout << brand << " " << model << " (" << year << ")" << endl;
        cout << "Текущая скорость: " << speed << " км/ч" << endl;
    }
};

// Наследование
class Car : public Vehicle {
private:
    int doors;
    string fuelType;

public:
    Car(string b, string m, int y, int d, string fuel) 
        : Vehicle(b, m, y), doors(d), fuelType(fuel) {}
    
    // Полиморфизм - переопределение виртуальных методов
    void start() override {
        cout << "Автомобиль " << brand << " заводится с ключа" << endl;
        setSpeed(0);
    }
    
    void stop() override {
        cout << "Автомобиль " << brand << " останавливается" << endl;
        setSpeed(0);
    }
    
    void accelerate(double amount) override {
        setSpeed(getSpeed() + amount);
        cout << "Автомобиль разгоняется до " << getSpeed() << " км/ч" << endl;
    }
    
    void displayInfo() const override {
        Vehicle::displayInfo();
        cout << "Количество дверей: " << doors << endl;
        cout << "Тип топлива: " << fuelType << endl;
    }
};

class Motorcycle : public Vehicle {
private:
    bool hasSidecar;

public:
    Motorcycle(string b, string m, int y, bool sidecar) 
        : Vehicle(b, m, y), hasSidecar(sidecar) {}
    
    void start() override {
        cout << "Мотоцикл " << brand << " заводится кнопкой" << endl;
        setSpeed(0);
    }
    
    void stop() override {
        cout << "Мотоцикл " << brand << " останавливается" << endl;
        setSpeed(0);
    }
    
    void accelerate(double amount) override {
        setSpeed(getSpeed() + amount * 1.2);  // Мотоциклы разгоняются быстрее
        cout << "Мотоцикл разгоняется до " << getSpeed() << " км/ч" << endl;
    }
    
    void displayInfo() const override {
        Vehicle::displayInfo();
        cout << "Имеет коляску: " << (hasSidecar ? "Да" : "Нет") << endl;
    }
    
    void wheelie() {  // Уникальный метод для мотоцикла
        cout << "Мотоцикл " << brand << " делает вилли!" << endl;
    }
};
```

Это подробное объяснение охватывает все ключевые аспекты принципов ООП в C++. Каждый принцип демонстрируется на практических примерах с объяснением, почему он важен и как правильно его применять.
