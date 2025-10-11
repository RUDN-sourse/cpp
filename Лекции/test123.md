# Детальное объяснение ООП в C++

## 1. Введение в ООП - Подробное объяснение

### Что такое ООП на практике?

**Объектно-Ориентированное Программирование** - это не просто синтаксис, а способ мышления. Давайте представим реальный пример:

```cpp
// Без ООП - процедурный подход
struct Car {
    string brand;
    string model;
    int year;
    double engineVolume;
    bool isRunning;
};

void startCar(Car* car) {
    car->isRunning = true;
    cout << car->brand << " " << car->model << " завелась" << endl;
}

void stopCar(Car* car) {
    car->isRunning = false;
    cout << car->brand << " " << car->model << " остановилась" << endl;
}

// С ООП - объектный подход
class Car {
private:
    string brand;
    string model;
    int year;
    double engineVolume;
    bool isRunning;
    double fuelLevel;

public:
    Car(string b, string m, int y, double engine) 
        : brand(b), model(m), year(y), engineVolume(engine), isRunning(false), fuelLevel(100.0) {}
    
    void start() {
        if (fuelLevel > 0) {
            isRunning = true;
            cout << brand << " " << model << " завелась" << endl;
        } else {
            cout << "Недостаточно топлива!" << endl;
        }
    }
    
    void stop() {
        isRunning = false;
        cout << brand << " " << model << " остановилась" << endl;
    }
    
    void refuel(double amount) {
        fuelLevel += amount;
        if (fuelLevel > 100.0) fuelLevel = 100.0;
    }
    
    void displayInfo() const {
        cout << brand << " " << model << " " << year << " года" << endl;
        cout << "Состояние: " << (isRunning ? "заведена" : "заглушена") << endl;
        cout << "Топливо: " << fuelLevel << "%" << endl;
    }
};
```

### Почему ООП лучше?

**Без ООП проблемы:**
- Данные и функции разделены
- Легко нарушить целостность данных
- Сложно отследить, какие функции работают с какими структурами
- Нет контроля над изменением состояния

**С ООП преимущества:**
- Данные и методы объединены
- Контроль доступа к данным
- Легче тестировать и отлаживать
- Код более читаем и поддерживаем

## 2. Классы и объекты - Глубокое погружение

### Разница между class и struct

```cpp
// В C++ разница минимальна:
struct PersonStruct {      // по умолчанию все public
    string name;
    int age;
    
    void display() {
        cout << name << ", " << age << " лет" << endl;
    }
};

class PersonClass {        // по умолчанию все private
    string name;
    int age;
    
public:
    void display() {
        cout << name << ", " << age << " лет" << endl;
    }
    
    void setName(string n) {
        if (!n.empty()) {
            name = n;
        }
    }
};
```

### Конвенции именования

```cpp
class BankAccount {
private:
    string m_accountNumber;    // префикс m_ для членов класса
    double m_balance;          // ясность в методах
    
public:
    // Геттеры
    string getAccountNumber() const { return m_accountNumber; }
    double getBalance() const { return m_balance; }
    
    // Сеттеры
    void setAccountNumber(const string& accNum) {
        if (accNum.length() == 20) {  // проверка IBAN
            m_accountNumber = accNum;
        }
    }
};
```

### Вопрос: Когда использовать struct vs class?

**Ответ:**
- **struct** для простых контейнеров данных (Point, Rectangle, Color)
- **class** для сложных объектов с поведением и инвариантами

```cpp
// struct - пассивные данные
struct Point {
    double x, y;
    // нет сложного поведения, только данные
};

struct RGB {
    uint8_t red, green, blue;
};

// class - активные объекты с поведением
class FileHandler {
private:
    FILE* m_file;
    string m_fileName;
    bool m_isOpen;
    
public:
    bool open(const string& filename);
    bool close();
    string readLine();
    bool write(const string& data);
    // сложное управление ресурсами
};
```

## 3. Инкапсуляция - Защита данных

### Почему приватные поля так важны?

```cpp
class Temperature {
private:
    double m_celsius;  // внутреннее представление

public:
    // Конструктор с валидацией
    Temperature(double celsius) {
        setCelsius(celsius);
    }
    
    // Геттер
    double getCelsius() const { 
        return m_celsius; 
    }
    
    // Сеттер с валидацией
    void setCelsius(double celsius) {
        // Абсолютный ноль - минимальная возможная температура
        if (celsius >= -273.15) {
            m_celsius = celsius;
        } else {
            throw invalid_argument("Температура не может быть ниже абсолютного нуля!");
        }
    }
    
    // Другие представления температуры
    double getFahrenheit() const {
        return m_celsius * 9.0/5.0 + 32.0;
    }
    
    void setFahrenheit(double fahrenheit) {
        setCelsius((fahrenheit - 32.0) * 5.0/9.0);
    }
    
    double getKelvin() const {
        return m_celsius + 273.15;
    }
    
    void setKelvin(double kelvin) {
        setCelsius(kelvin - 273.15);
    }
};
```

### Инварианты класса

**Инвариант** - условие, которое всегда истинно для объекта.

```cpp
class Date {
private:
    int m_day, m_month, m_year;
    
    // Приватный метод для проверки валидности даты
    bool isValidDate(int day, int month, int year) const {
        if (year < 1 || month < 1 || month > 12 || day < 1) 
            return false;
            
        int daysInMonth[] = {31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31};
        
        // Проверка високосного года
        if (month == 2 && isLeapYear(year)) {
            return day <= 29;
        }
        
        return day <= daysInMonth[month - 1];
    }
    
    bool isLeapYear(int year) const {
        return (year % 4 == 0 && year % 100 != 0) || (year % 400 == 0);
    }

public:
    Date(int day, int month, int year) {
        // Поддерживаем инвариант: дата всегда валидна
        if (!isValidDate(day, month, year)) {
            throw invalid_argument("Некорректная дата!");
        }
        m_day = day;
        m_month = month;
        m_year = year;
    }
    
    void setDate(int day, int month, int year) {
        if (!isValidDate(day, month, year)) {
            throw invalid_argument("Некорректная дата!");
        }
        m_day = day;
        m_month = month;
        m_year = year;
    }
    
    // Инвариант гарантирует, что дата всегда корректна
};
```

## 4. Конструкторы и деструкторы - Управление жизненным циклом

### Подробно о конструкторах

```cpp
class SmartArray {
private:
    int* m_data;
    size_t m_size;
    size_t m_capacity;

public:
    // 1. Конструктор по умолчанию
    SmartArray() : m_data(nullptr), m_size(0), m_capacity(0) {
        cout << "Конструктор по умолчанию" << endl;
    }
    
    // 2. Параметризованный конструктор
    SmartArray(size_t capacity) : m_size(0), m_capacity(capacity) {
        m_data = new int[capacity];
        cout << "Параметризованный конструктор, capacity: " << capacity << endl;
    }
    
    // 3. Конструктор копирования (ГЛУБОКОЕ копирование)
    SmartArray(const SmartArray& other) 
        : m_size(other.m_size), m_capacity(other.m_capacity) {
        
        m_data = new int[m_capacity];
        for (size_t i = 0; i < m_size; ++i) {
            m_data[i] = other.m_data[i];
        }
        cout << "Конструктор копирования" << endl;
    }
    
    // 4. Конструктор перемещения (C++11)
    SmartArray(SmartArray&& other) noexcept 
        : m_data(other.m_data), m_size(other.m_size), m_capacity(other.m_capacity) {
        
        // "Обкрадываем" другой объект
        other.m_data = nullptr;
        other.m_size = 0;
        other.m_capacity = 0;
        cout << "Конструктор перемещения" << endl;
    }
    
    // 5. Деструктор
    ~SmartArray() {
        delete[] m_data;  // безопасно для nullptr
        cout << "Деструктор, size: " << m_size << endl;
    }
    
    // 6. Оператор присваивания копированием
    SmartArray& operator=(const SmartArray& other) {
        cout << "Оператор присваивания копированием" << endl;
        
        // Защита от самоприсваивания: a = a;
        if (this == &other) {
            return *this;
        }
        
        // Освобождаем старые ресурсы
        delete[] m_data;
        
        // Копируем новые ресурсы
        m_size = other.m_size;
        m_capacity = other.m_capacity;
        m_data = new int[m_capacity];
        for (size_t i = 0; i < m_size; ++i) {
            m_data[i] = other.m_data[i];
        }
        
        return *this;
    }
    
    // 7. Оператор присваивания перемещением
    SmartArray& operator=(SmartArray&& other) noexcept {
        cout << "Оператор присваивания перемещением" << endl;
        
        // Защита от самоприсваивания
        if (this == &other) {
            return *this;
        }
        
        // Освобождаем старые ресурсы
        delete[] m_data;
        
        // Перемещаем ресурсы
        m_data = other.m_data;
        m_size = other.m_size;
        m_capacity = other.m_capacity;
        
        // Обнуляем другой объект
        other.m_data = nullptr;
        other.m_size = 0;
        other.m_capacity = 0;
        
        return *this;
    }
};
```

### Почему список инициализации важен?

```cpp
class Example {
private:
    const int m_constValue;    // должен быть инициализирован при создании
    int& m_reference;          // должен быть инициализирован при создании
    string m_name;
    vector<int> m_data;

public:
    // ПРАВИЛЬНО - список инициализации
    Example(int value, int& ref, string name) 
        : m_constValue(value)    // инициализация константы
        , m_reference(ref)       // инициализация ссылки
        , m_name(move(name))     // эффективная инициализация строки
        , m_data(100, 0)         // инициализация вектора
    {
        // Тело конструктора
    }
    
    // НЕПРАВИЛЬНО - присваивание в теле
    /*
    Example(int value, int& ref, string name) {
        // ОШИБКА: m_constValue и m_reference должны быть инициализированы
        m_constValue = value;    // ОШИБКА компиляции!
        m_reference = ref;       // ОШИБКА компиляции!
        
        m_name = name;           // Неэффективно: вызов конструктора + оператор присваивания
        m_data = vector<int>(100, 0); // Неэффективно: временный объект + присваивание
    }
    */
};
```

### Порядок инициализации

**Важно:** Поля инициализируются в порядке объявления в классе, а не в порядке списка инициализации!

```cpp
class OrderExample {
private:
    int a;    // 1
    int b;    // 2
    int c;    // 3

public:
    // ОПАСНО: порядок инициализации a, b, c (по объявлению)
    // но в списке написано c, b, a
    OrderExample(int value) : c(value), b(c + 1), a(b + 1) {
        // Проблема: b инициализируется ДО c, но использует c!
        // a инициализируется ДО b, но использует b!
    }
    
    // ПРАВИЛЬНО:
    OrderExample(int value) : a(value + 2), b(value + 1), c(value) {
        // Теперь порядок правильный
    }
};
```

## 5. Наследование - Отношения "is-a"

### Типы наследования на практике

```cpp
class Animal {
protected:
    string m_name;
    int m_age;

public:
    Animal(string name, int age) : m_name(name), m_age(age) {}
    
    virtual void speak() const {
        cout << m_name << " издает звук" << endl;
    }
    
    virtual void eat() const {
        cout << m_name << " ест" << endl;
    }
    
    virtual ~Animal() = default;
};

// Public наследование - "является"
class Dog : public Animal {
private:
    string m_breed;

public:
    Dog(string name, int age, string breed) 
        : Animal(name, age), m_breed(breed) {}
    
    void speak() const override {
        cout << m_name << " гавкает: Гав-гав!" << endl;
    }
    
    void fetch() const {
        cout << m_name << " приносит палку!" << endl;
    }
};

// Protected наследование - редко используется
class Base {
public:
    int publicVar;
protected:
    int protectedVar;
private:
    int privateVar;
};

class DerivedProtected : protected Base {
    // publicVar становится protected
    // protectedVar остается protected  
    // privateVar недоступна
public:
    void test() {
        publicVar = 1;      // OK - теперь protected
        protectedVar = 2;   // OK
        // privateVar = 3;  // ERROR
    }
};

// Private наследование - "реализовано посредством"
class Stack : private vector<int> {
    // Все public/protected члены vector становятся private
public:
    void push(int value) {
        vector<int>::push_back(value);  // используем реализацию вектора
    }
    
    int pop() {
        if (!empty()) {
            int value = back();
            pop_back();
            return value;
        }
        throw runtime_error("Стек пуст");
    }
    
    bool empty() const {
        return vector<int>::empty();
    }
    
    // НЕ предоставляем доступ к другим методам вектора
    // Это НЕ Stack "is-a" Vector, а Stack "implemented-in-terms-of" Vector
};
```

### Множественное наследование

```cpp
// Интерфейсы (абстрактные классы только с чисто виртуальными функциями)
class Drawable {
public:
    virtual void draw() const = 0;
    virtual ~Drawable() = default;
};

class Clickable {
public:
    virtual void onClick() = 0;
    virtual bool contains(int x, int y) const = 0;
    virtual ~Clickable() = default;
};

class Movable {
public:
    virtual void move(int dx, int dy) = 0;
    virtual ~Movable() = default;
};

// Множественное наследование
class Button : public Drawable, public Clickable {
private:
    string m_text;
    int m_x, m_y, m_width, m_height;

public:
    Button(string text, int x, int y, int w, int h)
        : m_text(text), m_x(x), m_y(y), m_width(w), m_height(h) {}
    
    void draw() const override {
        cout << "Рисую кнопку: " << m_text 
             << " [" << m_x << "," << m_y << "," << m_width << "," << m_height << "]" << endl;
    }
    
    void onClick() override {
        cout << "Кнопка '" << m_text << "' нажата!" << endl;
    }
    
    bool contains(int x, int y) const override {
        return x >= m_x && x <= m_x + m_width &&
               y >= m_y && y <= m_y + m_height;
    }
};

class DraggableButton : public Button, public Movable {
public:
    DraggableButton(string text, int x, int y, int w, int h)
        : Button(text, x, y, w, h) {}
    
    void move(int dx, int dy) override {
        // Реализация перемещения
        cout << "Перемещаю кнопку на (" << dx << "," << dy << ")" << endl;
    }
};
```

### Проблема ромбовидного наследования

```cpp
class Animal {
protected:
    string m_name;
public:
    Animal(string name) : m_name(name) {}
    virtual void speak() const = 0;
};

class Mammal : virtual public Animal {  // виртуальное наследование
protected:
    int m_gestationPeriod;
public:
    Mammal(string name, int gestation) 
        : Animal(name), m_gestationPeriod(gestation) {}
};

class WingedAnimal : virtual public Animal {  // виртуальное наследование
protected:
    double m_wingspan;
public:
    WingedAnimal(string name, double wingspan) 
        : Animal(name), m_wingspan(wingspan) {}
};

// Без виртуального наследования было бы две копии Animal
class Bat : public Mammal, public WingedAnimal {
public:
    Bat(string name, int gestation, double wingspan)
        : Animal(name),                // напрямую инициализируем Animal
          Mammal(name, gestation), 
          WingedAnimal(name, wingspan) {}
    
    void speak() const override {
        cout << m_name << " пищит" << endl;  // теперь m_name одна
    }
};
```

## 6. Полиморфизм - Один интерфейс, много реализаций

### Виртуальные функции под капотом

```cpp
class Shape {
protected:
    string m_color;

public:
    Shape(string color) : m_color(color) {}
    
    // Виртуальная таблица (vtable) создается для классов с виртуальными функциями
    virtual double area() const = 0;
    virtual double perimeter() const = 0;
    virtual void draw() const {
        cout << "Рисую фигуру цвета " << m_color << endl;
    }
    
    // Виртуальный деструктор ОБЯЗАТЕЛЕН
    virtual ~Shape() {
        cout << "Уничтожаем Shape" << endl;
    }
};

class Circle : public Shape {
private:
    double m_radius;

public:
    Circle(string color, double radius) 
        : Shape(color), m_radius(radius) {}
    
    // override гарантирует, что метод переопределяет виртуальный метод
    double area() const override {
        return 3.14159 * m_radius * m_radius;
    }
    
    double perimeter() const override {
        return 2 * 3.14159 * m_radius;
    }
    
    void draw() const override {
        cout << "Рисую круг радиуса " << m_radius 
             << " цвета " << m_color << endl;
    }
    
    ~Circle() override {
        cout << "Уничтожаем Circle" << endl;
    }
};

class Rectangle : public Shape {
private:
    double m_width, m_height;

public:
    Rectangle(string color, double w, double h)
        : Shape(color), m_width(w), m_height(h) {}
    
    double area() const override {
        return m_width * m_height;
    }
    
    double perimeter() const override {
        return 2 * (m_width + m_height);
    }
    
    void draw() const override {
        cout << "Рисую прямоугольник " << m_width << "x" << m_height 
             << " цвета " << m_color << endl;
    }
    
    // final запрещает дальнейшее переопределение
    void scale(double factor) final {
        m_width *= factor;
        m_height *= factor;
    }
    
    ~Rectangle() override {
        cout << "Уничтожаем Rectangle" << endl;
    }
};

// final класс - нельзя наследовать
class Square final : public Rectangle {
public:
    Square(string color, double side)
        : Rectangle(color, side, side) {}
    
    // ОШИБКА: метод scale final в Rectangle
    // void scale(double factor) override { }
};
```

### Как работает полиморфизм?

```cpp
void demonstratePolymorphism() {
    vector<unique_ptr<Shape>> shapes;
    
    shapes.push_back(make_unique<Circle>("Красный", 5.0));
    shapes.push_back(make_unique<Rectangle>("Синий", 4.0, 6.0));
    shapes.push_back(make_unique<Square>("Зеленый", 3.0));
    
    // Полиморфное поведение во время выполнения
    for (const auto& shape : shapes) {
        shape->draw();                    // Вызовется правильная версия
        cout << "Площадь: " << shape->area() << endl;      // Полиморфно
        cout << "Периметр: " << shape->perimeter() << endl; // Полиморфно
        cout << "---" << endl;
    }
    
    // Автоматическое освобождение памяти через умные указатели
}
```

### Вопрос: Когда не использовать виртуальные функции?

**Ответ:**
- В классах, которые не предназначены для наследования
- В performance-critical коде (виртуальные вызовы дороже)
- Когда нужен контроль над layout памяти

```cpp
// Класс не для наследования - нет виртуальных функций
class Point3D final {
private:
    double x, y, z;

public:
    Point3D(double x, double y, double z) : x(x), y(y), z(z) {}
    
    // Нет виртуальных функций - класс final
    double length() const {
        return sqrt(x*x + y*y + z*z);
    }
    
    // Статический полиморфизм через перегрузку
    Point3D operator+(const Point3D& other) const {
        return Point3D(x + other.x, y + other.y, z + other.z);
    }
};
```

## 7. Абстрактные классы и интерфейсы

### Чисто виртуальные функции

```cpp
// Интерфейс - только чисто виртуальные функции
class Serializable {
public:
    virtual string toJson() const = 0;
    virtual void fromJson(const string& json) = 0;
    virtual ~Serializable() = default;
};

class Clonable {
public:
    virtual unique_ptr<Clonable> clone() const = 0;
    virtual ~Clonable() = default;
};

// Абстрактный класс - может иметь реализацию
class Vehicle {
protected:
    string m_brand;
    string m_model;
    int m_year;
    double m_speed;

public:
    Vehicle(string brand, string model, int year) 
        : m_brand(brand), m_model(model), m_year(year), m_speed(0) {}
    
    // Чисто виртуальные функции - интерфейс
    virtual void start() = 0;
    virtual void stop() = 0;
    virtual double getMaxSpeed() const = 0;
    
    // Виртуальные функции с реализацией по умолчанию
    virtual void accelerate(double amount) {
        m_speed += amount;
        double maxSpeed = getMaxSpeed();
        if (m_speed > maxSpeed) {
            m_speed = maxSpeed;
        }
        cout << m_brand << " " << m_model << " ускорилась до " << m_speed << " км/ч" << endl;
    }
    
    virtual void brake(double amount) {
        m_speed -= amount;
        if (m_speed < 0) m_speed = 0;
        cout << m_brand << " " << m_model << " замедлилась до " << m_speed << " км/ч" << endl;
    }
    
    // Невиртуальная функция
    void displayInfo() const {
        cout << m_brand << " " << m_model << " " << m_year << " года" << endl;
        cout << "Текущая скорость: " << m_speed << " км/ч" << endl;
    }
    
    virtual ~Vehicle() = default;
};

// Конкретная реализация
class Car : public Vehicle, public Serializable, public Clonable {
private:
    int m_doors;
    string m_fuelType;
    double m_engineVolume;

public:
    Car(string brand, string model, int year, int doors, string fuel, double engine)
        : Vehicle(brand, model, year), m_doors(doors), m_fuelType(fuel), m_engineVolume(engine) {}
    
    // Реализация чисто виртуальных функций Vehicle
    void start() override {
        cout << "Автомобиль " << m_brand << " " << m_model << " заводится с ключа" << endl;
        m_speed = 0;
    }
    
    void stop() override {
        cout << "Автомобиль " << m_brand << " " << m_model << " останавливается" << endl;
        m_speed = 0;
    }
    
    double getMaxSpeed() const override {
        return 200.0;  // Упрощенно
    }
    
    // Реализация Serializable
    string toJson() const override {
        return "{ \"brand\": \"" + m_brand + "\", \"model\": \"" + m_model + 
               "\", \"year\": " + to_string(m_year) + ", \"doors\": " + 
               to_string(m_doors) + " }";
    }
    
    void fromJson(const string& json) override {
        // Упрощенная реализация
        cout << "Загружаем данные автомобиля из JSON: " << json << endl;
    }
    
    // Реализация Clonable
    unique_ptr<Clonable> clone() const override {
        return make_unique<Car>(*this);
    }
    
    // Собственные методы
    void honk() const {
        cout << "Би-бип!" << endl;
    }
};
```

## 8. Дружественные функции - когда нарушать инкапсуляцию

### Обоснованное использование friend

```cpp
class Matrix {
private:
    vector<vector<double>> m_data;
    size_t m_rows, m_cols;

    // Проверка индексов
    bool isValidIndex(size_t row, size_t col) const {
        return row < m_rows && col < m_cols;
    }

public:
    Matrix(size_t rows, size_t cols) 
        : m_rows(rows), m_cols(cols), m_data(rows, vector<double>(cols, 0)) {}
    
    // Геттеры
    size_t getRows() const { return m_rows; }
    size_t getCols() const { return m_cols; }
    
    // Доступ к элементам с проверкой
    double& at(size_t row, size_t col) {
        if (!isValidIndex(row, col)) {
            throw out_of_range("Индекс вне диапазона");
        }
        return m_data[row][col];
    }
    
    const double& at(size_t row, size_t col) const {
        if (!isValidIndex(row, col)) {
            throw out_of_range("Индекс вне диапазона");
        }
        return m_data[row][col];
    }
    
    // Дружественные функции для операторов
    friend Matrix operator+(const Matrix& lhs, const Matrix& rhs);
    friend Matrix operator*(const Matrix& lhs, const Matrix& rhs);
    friend ostream& operator<<(ostream& os, const Matrix& matrix);
    
    // Дружественный класс
    friend class MatrixCalculator;
};

// Дружественные функции имеют доступ к приватным членам
Matrix operator+(const Matrix& lhs, const Matrix& rhs) {
    if (lhs.m_rows != rhs.m_rows || lhs.m_cols != rhs.m_cols) {
        throw invalid_argument("Размеры матриц не совпадают");
    }
    
    Matrix result(lhs.m_rows, lhs.m_cols);
    for (size_t i = 0; i < lhs.m_rows; ++i) {
        for (size_t j = 0; j < lhs.m_cols; ++j) {
            result.m_data[i][j] = lhs.m_data[i][j] + rhs.m_data[i][j];
        }
    }
    return result;
}

ostream& operator<<(ostream& os, const Matrix& matrix) {
    for (size_t i = 0; i < matrix.m_rows; ++i) {
        for (size_t j = 0; j < matrix.m_cols; ++j) {
            os << matrix.m_data[i][j] << " ";
        }
        os << endl;
    }
    return os;
}

// Дружественный класс
class MatrixCalculator {
public:
    static Matrix transpose(const Matrix& matrix) {
        Matrix result(matrix.m_cols, matrix.m_rows);
        for (size_t i = 0; i < matrix.m_rows; ++i) {
            for (size_t j = 0; j < matrix.m_cols; ++j) {
                result.m_data[j][i] = matrix.m_data[i][j];
            }
        }
        return result;
    }
    
    static double determinant(const Matrix& matrix) {
        if (matrix.m_rows != matrix.m_cols) {
            throw invalid_argument("Матрица должна быть квадратной");
        }
        // Упрощенная реализация для 2x2
        if (matrix.m_rows == 2) {
            return matrix.m_data[0][0] * matrix.m_data[1][1] - 
                   matrix.m_data[0][1] * matrix.m_data[1][0];
        }
        // Для больших матриц - рекурсивное вычисление
        return 0; // заглушка
    }
};
```

### Когда НЕ использовать friend?

```cpp
// ПЛОХО: избыточное использование friend
class BankAccount {
private:
    double balance;
    string accountNumber;

public:
    // НЕПРАВИЛЬНО - вместо friend лучше сделать метод
    friend void printBalance(BankAccount& account) {
        cout << "Баланс: " << account.balance << endl;
    }
    
    // ПРАВИЛЬНО - метод класса
    void printBalance() const {
        cout << "Баланс: " << balance << endl;
    }
};

// ПРАВИЛЬНОЕ использование friend - операторы
class Complex {
private:
    double real, imag;

public:
    Complex(double r = 0, double i = 0) : real(r), imag(i) {}
    
    // Операторы как дружественные функции
    friend Complex operator+(const Complex& lhs, const Complex& rhs);
    friend bool operator==(const Complex& lhs, const Complex& rhs);
    
    // Методы для геттеров
    double getReal() const { return real; }
    double getImag() const { return imag; }
};

Complex operator+(const Complex& lhs, const Complex& rhs) {
    return Complex(lhs.real + rhs.real, lhs.imag + rhs.imag);
}

bool operator==(const Complex& lhs, const Complex& rhs) {
    return lhs.real == rhs.real && lhs.imag == rhs.imag;
}
```

## 9. Статические члены - данные класса, а не объекта

### Подробно о статических членах

```cpp
class DatabaseConnection {
private:
    string m_connectionString;
    bool m_isConnected;
    
    // Статические члены - общие для всех объектов
    static int s_connectionCount;       // количество соединений
    static int s_maxConnections;        // максимальное количество
    static mutex s_connectionMutex;     // для потокобезопасности

public:
    DatabaseConnection(const string& connStr) 
        : m_connectionString(connStr), m_isConnected(false) {
        
        // Потокобезопасное увеличение счетчика
        lock_guard<mutex> lock(s_connectionMutex);
        if (s_connectionCount < s_maxConnections) {
            s_connectionCount++;
            m_isConnected = true;
            cout << "Соединение установлено. Всего соединений: " 
                 << s_connectionCount << endl;
        } else {
            cout << "Достигнут лимит соединений!" << endl;
        }
    }
    
    ~DatabaseConnection() {
        if (m_isConnected) {
            lock_guard<mutex> lock(s_connectionMutex);
            s_connectionCount--;
            cout << "Соединение закрыто. Осталось соединений: " 
                 << s_connectionCount << endl;
        }
    }
    
    // Статические методы - работают только со статическими членами
    static int getConnectionCount() {
        return s_connectionCount;
    }
    
    static int getMaxConnections() {
        return s_maxConnections;
    }
    
    static void setMaxConnections(int max) {
        lock_guard<mutex> lock(s_connectionMutex);
        s_maxConnections = max;
    }
    
    static bool canCreateConnection() {
        return s_connectionCount < s_maxConnections;
    }
    
    // Нестатический метод
    bool executeQuery(const string& query) {
        if (!m_isConnected) {
            cout << "Нет соединения с базой данных!" << endl;
            return false;
        }
        cout << "Выполняем запрос: " << query << endl;
        return true;
    }
};

// Инициализация статических членов ВНЕ класса
int DatabaseConnection::s_connectionCount = 0;
int DatabaseConnection::s_maxConnections = 5;
mutex DatabaseConnection::s_connectionMutex;
```

### Статические константы

```cpp
class MathConstants {
public:
    // Статические константы можно инициализировать в классе (C++11)
    static constexpr double PI = 3.14159265358979323846;
    static constexpr double E = 2.71828182845904523536;
    static constexpr double SQRT2 = 1.41421356237309504880;
    
    // Для сложных типов нужно определение вне класса
    static const string APP_NAME;
    
    // Статические методы
    static double toRadians(double degrees) {
        return degrees * PI / 180.0;
    }
    
    static double toDegrees(double radians) {
        return radians * 180.0 / PI;
    }
};

// Определение статической константы
const string MathConstants::APP_NAME = "Math Calculator";

// Использование
void useMathConstants() {
    cout
