# Продвинутые структуры данных в C++

## 1. Контейнеры стандартной библиотеки (STL)

### 1.1. Вектор (vector) - динамический массив

```cpp
#include <iostream>
#include <vector>
using namespace std;

int main() {
    // Создание вектора
    vector<int> numbers = {1, 2, 3, 4, 5};
    
    // Добавление элементов
    numbers.push_back(6);  // В конец
    numbers.insert(numbers.begin() + 2, 10);  // В позицию
    
    // Доступ к элементам
    cout << "Первый элемент: " << numbers[0] << endl;       // Быстрый доступ
    cout << "Второй элемент: " << numbers.at(1) << endl;    // С проверкой границ
    
    // Размер и емкость
    cout << "Размер: " << numbers.size() << endl;
    cout << "Емкость: " << numbers.capacity() << endl;
    
    // Удаление элементов
    numbers.pop_back();  // Удалить последний
    numbers.erase(numbers.begin() + 1);  // Удалить по позиции
    
    // Итерация
    for (int num : numbers) {
        cout << num << " ";
    }
    cout << endl;
    
    return 0;
}
```

**Преимущества вектора:**
- Динамическое изменение размера
- Автоматическое управление памятью
- Быстрый доступ по индексу O(1)

### 1.2. Список (list) - двусвязный список

```cpp
#include <iostream>
#include <list>
using namespace std;

int main() {
    list<int> myList = {1, 2, 3};
    
    // Добавление элементов
    myList.push_front(0);    // В начало O(1)
    myList.push_back(4);     // В конец O(1)
    
    // Вставка в середину
    auto it = myList.begin();
    advance(it, 2);  // Перемещаем итератор на 3-ю позицию
    myList.insert(it, 10);
    
    // Удаление
    myList.pop_front();
    myList.remove(2);  // Удалить все элементы со значением 2
    
    // Итерация
    for (int num : myList) {
        cout << num << " ";
    }
    cout << endl;
    
    return 0;
}
```

**Особенности списка:**
- Быстрая вставка/удаление в любом месте O(1)
- Медленный доступ по индексу O(n)
- Требует больше памяти из-за хранения указателей

### 1.3. Дек (deque) - двусторонняя очередь

```cpp
#include <iostream>
#include <deque>
using namespace std;

int main() {
    deque<int> dq = {2, 3, 4};
    
    // Добавление с обеих сторон
    dq.push_front(1);  // В начало O(1)
    dq.push_back(5);   // В конец O(1)
    
    // Доступ к элементам
    cout << "Первый: " << dq.front() << endl;
    cout << "Последний: " << dq.back() << endl;
    cout << "Элемент по индексу 2: " << dq[2] << endl;
    
    // Удаление
    dq.pop_front();
    dq.pop_back();
    
    return 0;
}
```

### 1.4. Стек (stack) - LIFO структура

```cpp
#include <iostream>
#include <stack>
using namespace std;

int main() {
    stack<int> st;
    
    // Добавление элементов
    st.push(1);
    st.push(2);
    st.push(3);
    
    // Просмотр верхнего элемента
    cout << "Верхний элемент: " << st.top() << endl;
    
    // Удаление
    st.pop();
    cout << "После удаления: " << st.top() << endl;
    
    // Размер
    cout << "Размер стека: " << st.size() << endl;
    
    return 0;
}
```

### 1.5. Очередь (queue) - FIFO структура

```cpp
#include <iostream>
#include <queue>
using namespace std;

int main() {
    queue<int> q;
    
    // Добавление в конец
    q.push(1);
    q.push(2);
    q.push(3);
    
    // Доступ к первому и последнему
    cout << "Первый элемент: " << q.front() << endl;
    cout << "Последний элемент: " << q.back() << endl;
    
    // Удаление из начала
    q.pop();
    cout << "После удаления первого: " << q.front() << endl;
    
    return 0;
}
```

### 1.6. Очередь с приоритетом (priority_queue)

```cpp
#include <iostream>
#include <queue>
#include <vector>
using namespace std;

int main() {
    // Максимальная куча (по умолчанию)
    priority_queue<int> maxHeap;
    
    // Минимальная куча
    priority_queue<int, vector<int>, greater<int>> minHeap;
    
    maxHeap.push(3);
    maxHeap.push(1);
    maxHeap.push(4);
    maxHeap.push(2);
    
    cout << "Максимальная куча (вывод в порядке убывания):" << endl;
    while (!maxHeap.empty()) {
        cout << maxHeap.top() << " ";
        maxHeap.pop();
    }
    cout << endl;
    
    minHeap.push(3);
    minHeap.push(1);
    minHeap.push(4);
    minHeap.push(2);
    
    cout << "Минимальная куча (вывод в порядке возрастания):" << endl;
    while (!minHeap.empty()) {
        cout << minHeap.top() << " ";
        minHeap.pop();
    }
    cout << endl;
    
    return 0;
}
```

## 2. Ассоциативные контейнеры

### 2.1. Множество (set) - уникальные отсортированные элементы

```cpp
#include <iostream>
#include <set>
using namespace std;

int main() {
    set<int> mySet = {3, 1, 4, 1, 5, 9};  // Дубликаты игнорируются
    
    // Вставка
    mySet.insert(2);
    mySet.insert(6);
    
    // Поиск
    auto it = mySet.find(4);
    if (it != mySet.end()) {
        cout << "Элемент 4 найден" << endl;
    }
    
    // Удаление
    mySet.erase(1);
    
    // Итерация (элементы отсортированы)
    for (int num : mySet) {
        cout << num << " ";
    }
    cout << endl;
    
    // Проверка наличия элемента
    if (mySet.count(5) > 0) {
        cout << "5 присутствует в множестве" << endl;
    }
    
    return 0;
}
```

### 2.2. Мультимножество (multiset) - разрешены дубликаты

```cpp
#include <iostream>
#include <set>
using namespace std;

int main() {
    multiset<int> ms = {1, 2, 2, 3, 3, 3};
    
    // Вставка дубликатов разрешена
    ms.insert(2);
    ms.insert(3);
    
    // Подсчет количества вхождений
    cout << "Количество двоек: " << ms.count(2) << endl;
    cout << "Количество троек: " << ms.count(3) << endl;
    
    // Удаление всех вхождений
    ms.erase(3);  // Удалит все тройки
    
    for (int num : ms) {
        cout << num << " ";
    }
    cout << endl;
    
    return 0;
}
```

### 2.3. Словарь (map) - пары ключ-значение

```cpp
#include <iostream>
#include <map>
#include <string>
using namespace std;

int main() {
    map<string, int> ageMap;
    
    // Вставка элементов
    ageMap["Alice"] = 25;
    ageMap["Bob"] = 30;
    ageMap["Charlie"] = 35;
    
    // Или так
    ageMap.insert({"David", 40});
    
    // Доступ к элементам
    cout << "Возраст Alice: " << ageMap["Alice"] << endl;
    
    // Проверка существования ключа
    if (ageMap.find("Eve") == ageMap.end()) {
        cout << "Eve не найден в словаре" << endl;
    }
    
    // Итерация (ключи отсортированы)
    for (const auto& pair : ageMap) {
        cout << pair.first << ": " << pair.second << endl;
    }
    
    // Удаление
    ageMap.erase("Bob");
    
    return 0;
}
```

### 2.4. Мультисловарь (multimap) - несколько значений для одного ключа

```cpp
#include <iostream>
#include <map>
#include <string>
using namespace std;

int main() {
    multimap<string, string> phonebook;
    
    // Один ключ может иметь несколько значений
    phonebook.insert({"Alice", "123-4567"});
    phonebook.insert({"Alice", "765-4321"});
    phonebook.insert({"Bob", "555-1234"});
    phonebook.insert({"Alice", "999-8888"});
    
    // Поиск всех значений для ключа
    auto range = phonebook.equal_range("Alice");
    
    cout << "Телефоны Alice:" << endl;
    for (auto it = range.first; it != range.second; ++it) {
        cout << it->second << endl;
    }
    
    return 0;
}
```

### 2.5. Неупорядоченные контейнеры (C++11)

```cpp
#include <iostream>
#include <unordered_set>
#include <unordered_map>
#include <string>
using namespace std;

int main() {
    // Хэш-таблица вместо бинарного дерева
    unordered_set<string> us = {"apple", "banana", "cherry"};
    unordered_map<string, int> um = {{"one", 1}, {"two", 2}, {"three", 3}};
    
    // Быстрый доступ O(1) в среднем случае
    us.insert("date");
    um["four"] = 4;
    
    cout << "Элементы unordered_set:" << endl;
    for (const string& fruit : us) {
        cout << fruit << " ";
    }
    cout << endl;
    
    cout << "Элементы unordered_map:" << endl;
    for (const auto& pair : um) {
        cout << pair.first << ": " << pair.second << endl;
    }
    
    return 0;
}
```

## 3. Строки (string) - специализированный контейнер

```cpp
#include <iostream>
#include <string>
#include <algorithm>
using namespace std;

int main() {
    string str = "Hello, World!";
    
    // Доступ к символам
    cout << "Первый символ: " << str[0] << endl;
    cout << "Длина строки: " << str.length() << endl;
    
    // Поиск подстроки
    size_t pos = str.find("World");
    if (pos != string::npos) {
        cout << "Подстрока найдена в позиции: " << pos << endl;
    }
    
    // Изменение строки
    str.replace(7, 5, "C++");  // Замена "World" на "C++"
    cout << "После замены: " << str << endl;
    
    // Подстрока
    string sub = str.substr(0, 5);  // "Hello"
    cout << "Подстрока: " << sub << endl;
    
    // Конкатенация
    string newStr = str + " Welcome!";
    cout << "После конкатенации: " << newStr << endl;
    
    return 0;
}
```

## 4. Кортежи (tuple) - C++11

```cpp
#include <iostream>
#include <tuple>
#include <string>
using namespace std;

int main() {
    // Создание кортежа с разными типами
    tuple<string, int, double> person("Alice", 25, 165.5);
    
    // Доступ к элементам
    cout << "Имя: " << get<0>(person) << endl;
    cout << "Возраст: " << get<1>(person) << endl;
    cout << "Рост: " << get<2>(person) << endl;
    
    // Изменение элементов
    get<1>(person) = 26;
    
    // Structured binding (C++17)
    auto [name, age, height] = person;
    cout << name << " теперь " << age << " лет" << endl;
    
    // Сравнение кортежей
    tuple<string, int> t1("Alice", 25);
    tuple<string, int> t2("Bob", 30);
    
    if (t1 < t2) {
        cout << "t1 меньше t2" << endl;
    }
    
    return 0;
}
```

## 5. Пара (pair) - частный случай кортежа

```cpp
#include <iostream>
#include <utility>
#include <map>
using namespace std;

int main() {
    // Создание пары
    pair<string, int> student("Alice", 95);
    
    // Доступ к элементам
    cout << "Имя: " << student.first << endl;
    cout << "Оценка: " << student.second << endl;
    
    // Создание с помощью make_pair
    auto teacher = make_pair("Mr. Smith", "Mathematics");
    
    // Пары часто используются в map
    map<pair<string, string>, int> classGrades;
    classGrades[{"Alice", "Math"}] = 95;
    classGrades[{"Bob", "Science"}] = 88;
    
    return 0;
}
```

## 6. Пользовательские структуры данных

### 6.1. Связный список (реализация)

```cpp
#include <iostream>
using namespace std;

template<typename T>
class LinkedList {
private:
    struct Node {
        T data;
        Node* next;
        Node(const T& value) : data(value), next(nullptr) {}
    };
    
    Node* head;
    Node* tail;
    int size;

public:
    LinkedList() : head(nullptr), tail(nullptr), size(0) {}
    
    ~LinkedList() {
        while (head != nullptr) {
            Node* temp = head;
            head = head->next;
            delete temp;
        }
    }
    
    void pushFront(const T& value) {
        Node* newNode = new Node(value);
        if (head == nullptr) {
            head = tail = newNode;
        } else {
            newNode->next = head;
            head = newNode;
        }
        size++;
    }
    
    void pushBack(const T& value) {
        Node* newNode = new Node(value);
        if (tail == nullptr) {
            head = tail = newNode;
        } else {
            tail->next = newNode;
            tail = newNode;
        }
        size++;
    }
    
    void print() const {
        Node* current = head;
        while (current != nullptr) {
            cout << current->data << " -> ";
            current = current->next;
        }
        cout << "nullptr" << endl;
    }
    
    int getSize() const { return size; }
};

int main() {
    LinkedList<int> list;
    list.pushBack(1);
    list.pushBack(2);
    list.pushFront(0);
    list.pushBack(3);
    
    list.print();  // 0 -> 1 -> 2 -> 3 -> nullptr
    cout << "Размер: " << list.getSize() << endl;
    
    return 0;
}
```

### 6.2. Бинарное дерево поиска

```cpp
#include <iostream>
using namespace std;

template<typename T>
class BinarySearchTree {
private:
    struct Node {
        T data;
        Node* left;
        Node* right;
        Node(const T& value) : data(value), left(nullptr), right(nullptr) {}
    };
    
    Node* root;
    
    Node* insert(Node* node, const T& value) {
        if (node == nullptr) {
            return new Node(value);
        }
        
        if (value < node->data) {
            node->left = insert(node->left, value);
        } else if (value > node->data) {
            node->right = insert(node->right, value);
        }
        
        return node;
    }
    
    void inOrder(Node* node) const {
        if (node != nullptr) {
            inOrder(node->left);
            cout << node->data << " ";
            inOrder(node->right);
        }
    }
    
    bool contains(Node* node, const T& value) const {
        if (node == nullptr) return false;
        if (value == node->data) return true;
        
        if (value < node->data) {
            return contains(node->left, value);
        } else {
            return contains(node->right, value);
        }
    }

public:
    BinarySearchTree() : root(nullptr) {}
    
    void insert(const T& value) {
        root = insert(root, value);
    }
    
    bool contains(const T& value) const {
        return contains(root, value);
    }
    
    void printInOrder() const {
        inOrder(root);
        cout << endl;
    }
};

int main() {
    BinarySearchTree<int> bst;
    bst.insert(5);
    bst.insert(3);
    bst.insert(7);
    bst.insert(2);
    bst.insert(4);
    bst.insert(6);
    bst.insert(8);
    
    cout << "Обход inorder: ";
    bst.printInOrder();  // 2 3 4 5 6 7 8
    
    cout << "Содержит 4: " << (bst.contains(4) ? "Да" : "Нет") << endl;
    cout << "Содержит 9: " << (bst.contains(9) ? "Да" : "Нет") << endl;
    
    return 0;
}
```

## 7. Графовые структуры

### 7.1. Представление графа через список смежности

```cpp
#include <iostream>
#include <vector>
#include <list>
using namespace std;

class Graph {
private:
    int vertices;
    vector<list<int>> adjList;

public:
    Graph(int V) : vertices(V), adjList(V) {}
    
    void addEdge(int src, int dest) {
        adjList[src].push_back(dest);
        adjList[dest].push_back(src);  // Для неориентированного графа
    }
    
    void printGraph() const {
        for (int i = 0; i < vertices; i++) {
            cout << "Вершина " << i << ": ";
            for (int neighbor : adjList[i]) {
                cout << neighbor << " ";
            }
            cout << endl;
        }
    }
    
    void BFS(int startVertex) {
        vector<bool> visited(vertices, false);
        list<int> queue;
        
        visited[startVertex] = true;
        queue.push_back(startVertex);
        
        cout << "BFS обход: ";
        while (!queue.empty()) {
            int current = queue.front();
            cout << current << " ";
            queue.pop_front();
            
            for (int neighbor : adjList[current]) {
                if (!visited[neighbor]) {
                    visited[neighbor] = true;
                    queue.push_back(neighbor);
                }
            }
        }
        cout << endl;
    }
};

int main() {
    Graph g(5);
    g.addEdge(0, 1);
    g.addEdge(0, 2);
    g.addEdge(1, 3);
    g.addEdge(2, 4);
    
    g.printGraph();
    g.BFS(0);
    
    return 0;
}
```

## Сравнение сложности операций

| Структура данных | Вставка | Удаление | Поиск | Доступ по индексу |
|------------------|---------|----------|--------|-------------------|
| vector | O(1) в конце, O(n) в середине | O(n) | O(n) | O(1) |
| list | O(1) | O(1) | O(n) | O(n) |
| set/map | O(log n) | O(log n) | O(log n) | - |
| unordered_set/map | O(1) | O(1) | O(1) | - |
| stack/queue | O(1) | O(1) | - | - |

## Заключение

C++ предоставляет богатый набор структур данных:

1. **Последовательные**: vector, list, deque, array
2. **Ассоциативные**: set, map, multiset, multimap
3. **Неупорядоченные**: unordered_set, unordered_map
4. **Адаптеры**: stack, queue, priority_queue
5. **Специализированные**: string, bitset
6. **Кортежи**: tuple, pair

**Рекомендации по выбору:**
- Используйте **vector** по умолчанию
- **list** для частых вставок/удалений в середине
- **set/map** когда нужна сортировка и уникальность
- **unordered_set/map** для максимальной скорости
- **stack/queue** для конкретных алгоритмов (LIFO/FIFO)
