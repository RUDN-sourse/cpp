# Лекция: Работа с базами данных в C++

## Оглавление
1. [Введение в базы данных и интерфейсы доступа](#введение)
2. [Выбор библиотеки для работы с БД](#выбор-библиотеки)
3. [Подключение к базе данных](#подключение)
4. [Выполнение запросов](#выполнение-запросов)
5. [Работа с транзакциями](#транзакции)
6. [Обработка ошибок](#обработка-ошибок)
7. [Безопасность и SQL-инъекции](#безопасность)
8. [Оптимизация производительности](#оптимизация)
9. [Паттерны проектирования](#паттерны)
10. [Практические примеры](#примеры)

## 1. Введение в базы данных и интерфейсы доступа <a name="введение"></a>

### Что такое API доступа к базам данных?
API (Application Programming Interface) для работы с базами данных - это набор функций и классов, которые позволяют приложению взаимодействовать с СУБД.

### Основные подходы:

**Нативные драйверы**:
- Прямое взаимодействие с API СУБД
- Высокая производительность
- Специфичны для каждой базы данных

**Универсальные интерфейсы**:
- ODBC (Open Database Connectivity)
- JDBC (для Java)
- Кроссплатформенные решения

## 2. Выбор библиотеки для работы с БД <a name="выбор-библиотеки"></a>

### Популярные библиотеки для C++:

**1. SQLite** - встраиваемая БД
```cpp
#include <sqlite3.h>
```
- **Плюсы**: нулевая конфигурация, сервер не требуется, легковесная
- **Минусы**: нет сетевого доступа, ограниченная параллельная запись

**2. MySQL Connector/C++**
```cpp
#include <mysql_driver.h>
#include <mysql_connection.h>
```
- **Плюсы**: высокая производительность, богатый функционал
- **Минусы**: требуется установка сервера

**3. PostgreSQL libpq**
```cpp
#include <libpq-fe.h>
```
- **Плюсы**: продвинутые функции, надежность
- **Минусы**: сложнее в настройке

**4. ODBC**
```cpp
#include <sql.h>
#include <sqlext.h>
```
- **Плюсы**: универсальность, поддержка многих СУБД
- **Минусы**: дополнительный уровень абстракции

## 3. Подключение к базе данных <a name="подключение"></a>

### Подключение к SQLite

```cpp
#include <sqlite3.h>
#include <iostream>
#include <string>

class Database {
private:
    sqlite3* db;
    std::string lastError;
    
public:
    Database() : db(nullptr) {}
    
    bool connect(const std::string& filename) {
        int result = sqlite3_open(filename.c_str(), &db);
        if (result != SQLITE_OK) {
            lastError = sqlite3_errmsg(db);
            sqlite3_close(db);
            db = nullptr;
            return false;
        }
        
        // Включение поддержки внешних ключей
        execute("PRAGMA foreign_keys = ON;");
        return true;
    }
    
    ~Database() {
        if (db) {
            sqlite3_close(db);
        }
    }
    
    std::string getLastError() const { return lastError; }
    sqlite3* getHandle() const { return db; }
};
```

**Критические моменты:**
- Всегда проверяйте результат открытия базы данных
- Закрывайте соединение в деструкторе
- Обрабатывайте ошибки немедленно

### Подключение к MySQL

```cpp
#include <mysql_driver.h>
#include <mysql_connection.h>
#include <cppconn/statement.h>
#include <cppconn/prepared_statement.h>
#include <cppconn/resultset.h>
#include <cppconn/exception.h>

class MySQLDatabase {
private:
    sql::mysql::MySQL_Driver* driver;
    sql::Connection* connection;
    
public:
    MySQLDatabase() : driver(nullptr), connection(nullptr) {
        driver = sql::mysql::get_mysql_driver_instance();
    }
    
    bool connect(const std::string& host, const std::string& user,
                const std::string& password, const std::string& database) {
        try {
            connection = driver->connect(host, user, password);
            connection->setSchema(database);
            return true;
        } catch (sql::SQLException& e) {
            std::cerr << "MySQL Error: " << e.what() << std::endl;
            std::cerr << "MySQL error code: " << e.getErrorCode() << std::endl;
            std::cerr << "SQLState: " << e.getSQLState() << std::endl;
            return false;
        }
    }
    
    ~MySQLDatabase() {
        if (connection) {
            delete connection;
        }
    }
};
```

## 4. Выполнение запросов <a name="выполнение-запросов"></a>

### Создание таблиц

```cpp
bool createTables() {
    const char* sql = R"(
        CREATE TABLE IF NOT EXISTS users (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            username TEXT UNIQUE NOT NULL,
            email TEXT UNIQUE NOT NULL,
            created_at DATETIME DEFAULT CURRENT_TIMESTAMP
        );
        
        CREATE TABLE IF NOT EXISTS posts (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            user_id INTEGER NOT NULL,
            title TEXT NOT NULL,
            content TEXT,
            created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
            FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
        );
    )";
    
    char* errorMessage = nullptr;
    int result = sqlite3_exec(db, sql, nullptr, nullptr, &errorMessage);
    
    if (result != SQLITE_OK) {
        lastError = errorMessage;
        sqlite3_free(errorMessage);
        return false;
    }
    
    return true;
}
```

### Выполнение простых запросов

```cpp
bool execute(const std::string& sql) {
    char* errorMessage = nullptr;
    int result = sqlite3_exec(db, sql.c_str(), nullptr, nullptr, &errorMessage);
    
    if (result != SQLITE_OK) {
        lastError = errorMessage;
        sqlite3_free(errorMessage);
        return false;
    }
    
    return true;
}
```

### Подготовленные выражения (Prepared Statements)

**Почему это важно:**
- Безопасность от SQL-инъекций
- Повторное использование запросов
- Лучшая производительность

```cpp
class PreparedStatement {
private:
    sqlite3_stmt* stmt;
    
public:
    PreparedStatement(sqlite3* db, const std::string& sql) : stmt(nullptr) {
        int result = sqlite3_prepare_v2(db, sql.c_str(), -1, &stmt, nullptr);
        if (result != SQLITE_OK) {
            throw std::runtime_error("Failed to prepare statement");
        }
    }
    
    ~PreparedStatement() {
        if (stmt) {
            sqlite3_finalize(stmt);
        }
    }
    
    // Привязка параметров
    void bindInt(int index, int value) {
        sqlite3_bind_int(stmt, index, value);
    }
    
    void bindText(int index, const std::string& value) {
        sqlite3_bind_text(stmt, index, value.c_str(), -1, SQLITE_TRANSIENT);
    }
    
    void bindDouble(int index, double value) {
        sqlite3_bind_double(stmt, index, value);
    }
    
    void bindNull(int index) {
        sqlite3_bind_null(stmt, index);
    }
    
    // Выполнение
    bool execute() {
        int result = sqlite3_step(stmt);
        sqlite3_reset(stmt); // Сбрасываем для повторного использования
        
        return result == SQLITE_DONE;
    }
    
    // Для SELECT запросов
    bool next() {
        int result = sqlite3_step(stmt);
        return result == SQLITE_ROW;
    }
    
    // Получение данных
    int getInt(int column) {
        return sqlite3_column_int(stmt, column);
    }
    
    std::string getText(int column) {
        const unsigned char* text = sqlite3_column_text(stmt, column);
        return text ? reinterpret_cast<const char*>(text) : "";
    }
    
    void reset() {
        sqlite3_reset(stmt);
        sqlite3_clear_bindings(stmt);
    }
};
```

### Пример использования подготовленных выражений

```cpp
class UserManager {
private:
    sqlite3* db;
    
public:
    UserManager(sqlite3* database) : db(database) {}
    
    bool addUser(const std::string& username, const std::string& email) {
        const std::string sql = "INSERT INTO users (username, email) VALUES (?, ?)";
        
        try {
            PreparedStatement stmt(db, sql);
            stmt.bindText(1, username);
            stmt.bindText(2, email);
            return stmt.execute();
        } catch (const std::exception& e) {
            std::cerr << "Error adding user: " << e.what() << std::endl;
            return false;
        }
    }
    
    User getUserById(int id) {
        const std::string sql = "SELECT id, username, email, created_at FROM users WHERE id = ?";
        User user;
        
        try {
            PreparedStatement stmt(db, sql);
            stmt.bindInt(1, id);
            
            if (stmt.next()) {
                user.id = stmt.getInt(0);
                user.username = stmt.getText(1);
                user.email = stmt.getText(2);
                user.createdAt = stmt.getText(3);
            }
        } catch (const std::exception& e) {
            std::cerr << "Error getting user: " << e.what() << std::endl;
        }
        
        return user;
    }
    
    std::vector<User> getAllUsers() {
        const std::string sql = "SELECT id, username, email, created_at FROM users";
        std::vector<User> users;
        
        try {
            PreparedStatement stmt(db, sql);
            
            while (stmt.next()) {
                User user;
                user.id = stmt.getInt(0);
                user.username = stmt.getText(1);
                user.email = stmt.getText(2);
                user.createdAt = stmt.getText(3);
                users.push_back(user);
            }
        } catch (const std::exception& e) {
            std::cerr << "Error getting users: " << e.what() << std::endl;
        }
        
        return users;
    }
};
```

## 5. Работа с транзакциями <a name="транзакции"></a>

### Зачем нужны транзакции?
- **Atomicity** (Атомарность) - все или ничего
- **Consistency** (Согласованность) - база переходит из одного согласованного состояния в другое
- **Isolation** (Изолированность) - параллельные транзакции не мешают друг другу
- **Durability** (Долговечность) - результаты фиксируются навсегда

### Реализация транзакций

```cpp
class Transaction {
private:
    sqlite3* db;
    bool committed;
    
public:
    Transaction(sqlite3* database) : db(database), committed(false) {
        execute("BEGIN TRANSACTION;");
    }
    
    ~Transaction() {
        if (!committed) {
            execute("ROLLBACK;");
        }
    }
    
    bool commit() {
        if (execute("COMMIT;")) {
            committed = true;
            return true;
        }
        return false;
    }
    
    void rollback() {
        execute("ROLLBACK;");
        committed = true; // Чтобы деструктор не откатывал повторно
    }
    
private:
    bool execute(const std::string& sql) {
        char* errorMessage = nullptr;
        int result = sqlite3_exec(db, sql.c_str(), nullptr, nullptr, &errorMessage);
        
        if (result != SQLITE_OK) {
            if (errorMessage) {
                std::cerr << "Transaction error: " << errorMessage << std::endl;
                sqlite3_free(errorMessage);
            }
            return false;
        }
        
        return true;
    }
};
```

### Пример использования транзакций

```cpp
bool transferMoney(int fromAccount, int toAccount, double amount) {
    Transaction transaction(db);
    
    try {
        // Снимаем деньги с первого счета
        PreparedStatement withdraw(db, "UPDATE accounts SET balance = balance - ? WHERE id = ? AND balance >= ?");
        withdraw.bindDouble(1, amount);
        withdraw.bindInt(2, fromAccount);
        withdraw.bindDouble(3, amount);
        
        if (!withdraw.execute()) {
            std::cerr << "Insufficient funds or account not found" << std::endl;
            return false;
        }
        
        // Добавляем деньги на второй счет
        PreparedStatement deposit(db, "UPDATE accounts SET balance = balance + ? WHERE id = ?");
        deposit.bindDouble(1, amount);
        deposit.bindInt(2, toAccount);
        
        if (!deposit.execute()) {
            std::cerr << "Target account not found" << std::endl;
            return false;
        }
        
        // Фиксируем транзакцию
        return transaction.commit();
        
    } catch (const std::exception& e) {
        std::cerr << "Transfer failed: " << e.what() << std::endl;
        return false;
    }
}
```

## 6. Обработка ошибок <a name="обработка-ошибок"></a>

### Стратегии обработки ошибок

```cpp
class DatabaseException : public std::exception {
private:
    std::string message;
    int errorCode;
    
public:
    DatabaseException(const std::string& msg, int code = 0) 
        : message(msg), errorCode(code) {}
    
    const char* what() const noexcept override {
        return message.c_str();
    }
    
    int getErrorCode() const { return errorCode; }
};

class SafeDatabase {
private:
    sqlite3* db;
    
    void checkError(int result, const std::string& operation) {
        if (result != SQLITE_OK) {
            std::string errorMsg = sqlite3_errmsg(db);
            throw DatabaseException(operation + " failed: " + errorMsg, result);
        }
    }
    
public:
    void connect(const std::string& filename) {
        checkError(sqlite3_open(filename.c_str(), &db), "Database connection");
        
        // Устанавливаем обработчик ошибок
        sqlite3_extended_result_codes(db, 1);
    }
    
    std::unique_ptr<PreparedStatement> prepare(const std::string& sql) {
        sqlite3_stmt* stmt;
        checkError(sqlite3_prepare_v2(db, sql.c_str(), -1, &stmt, nullptr), 
                   "Prepare statement");
        return std::unique_ptr<PreparedStatement>(new PreparedStatement(stmt));
    }
};
```

### Обработка специфических ошибок

```cpp
bool handleDatabaseError(const std::exception& e) {
    try {
        const DatabaseException* dbEx = dynamic_cast<const DatabaseException*>(&e);
        if (dbEx) {
            switch (dbEx->getErrorCode()) {
                case SQLITE_CONSTRAINT_UNIQUE:
                    std::cerr << "Duplicate entry" << std::endl;
                    return false;
                case SQLITE_CONSTRAINT_FOREIGNKEY:
                    std::cerr << "Foreign key constraint failed" << std::endl;
                    return false;
                case SQLITE_BUSY:
                    std::cerr << "Database is busy, retrying..." << std::endl;
                    return true; // Можно повторить
                default:
                    std::cerr << "Database error: " << dbEx->what() << std::endl;
                    return false;
            }
        }
    } catch (...) {
        // Обработка динамического приведения
    }
    
    std::cerr << "Unknown error: " << e.what() << std::endl;
    return false;
}
```

## 7. Безопасность и SQL-инъекции <a name="безопасность"></a>

### ❌ ОПАСНЫЙ КОД (уязвимый к SQL-инъекциям):

```cpp
// НИКОГДА ТАК НЕ ДЕЛАЙТЕ!
bool loginUser(const std::string& username, const std::string& password) {
    std::string sql = "SELECT * FROM users WHERE username = '" + username + 
                     "' AND password = '" + password + "'";
    
    // Злоумышленник может ввести: 
    // username: admin' --
    // password: anything
    // Получится: SELECT * FROM users WHERE username = 'admin' -- AND password = 'anything'
    // Комментарий -- закомментирует проверку пароля!
}
```

### ✅ БЕЗОПАСНЫЙ КОД (использование подготовленных выражений):

```cpp
bool loginUser(const std::string& username, const std::string& password) {
    const std::string sql = "SELECT id FROM users WHERE username = ? AND password = ?";
    
    try {
        PreparedStatement stmt(db, sql);
        stmt.bindText(1, username);
        stmt.bindText(2, password);
        
        return stmt.next(); // Если есть результат - пользователь аутентифицирован
    } catch (const std::exception& e) {
        std::cerr << "Login error: " << e.what() << std::endl;
        return false;
    }
}
```

### Дополнительные меры безопасности:

```cpp
class SecureDatabase {
private:
    sqlite3* db;
    
public:
    void enableSecurityFeatures() {
        // Включаем расширенные коды ошибок
        sqlite3_extended_result_codes(db, 1);
        
        // Устанавливаем лимит для предотвращения DoS-атак
        sqlite3_limit(db, SQLITE_LIMIT_LENGTH, 1000000); // Максимальный размер строки
        sqlite3_limit(db, SQLITE_LIMIT_SQL_LENGTH, 100000); // Максимальная длина SQL
        sqlite3_limit(db, SQLITE_LIMIT_COLUMN, 100); // Максимальное число колонок
        sqlite3_limit(db, SQLITE_LIMIT_EXPR_DEPTH, 100); // Максимальная глубина выражений
    }
    
    // Валидация ввода
    bool isValidInput(const std::string& input) {
        // Проверка на потенциально опасные шаблоны
        if (input.find("--") != std::string::npos ||
            input.find(";") != std::string::npos ||
            input.find("/*") != std::string::npos) {
            return false;
        }
        
        // Ограничение длины
        return input.length() <= 255;
    }
};
```

## 8. Оптимизация производительности <a name="оптимизация"></a>

### Настройки SQLite для лучшей производительности:

```cpp
void optimizeSQLite() {
    // Включаем WAL (Write-Ahead Logging) для лучшей параллельности
    execute("PRAGMA journal_mode = WAL;");
    
    // Увеличиваем размер кэша
    execute("PRAGMA cache_size = -64000;"); // 64MB
    
    // Включаем синхронизацию только в критических секциях
    execute("PRAGMA synchronous = NORMAL;");
    
    // Включаем внешние ключи
    execute("PRAGMA foreign_keys = ON;");
    
    // Увеличиваем таймаут для занятой базы
    sqlite3_busy_timeout(db, 5000); // 5 секунд
}
```

### Пакетная вставка данных:

```cpp
class BatchInserter {
private:
    sqlite3* db;
    std::unique_ptr<PreparedStatement> stmt;
    int batchSize;
    int currentCount;
    
public:
    BatchInserter(sqlite3* database, const std::string& sql, int batchSize = 1000) 
        : db(database), batchSize(batchSize), currentCount(0) {
        stmt = std::make_unique<PreparedStatement>(db, sql);
    }
    
    void insert(const std::string& data) {
        stmt->bindText(1, data);
        stmt->execute();
        stmt->reset();
        
        currentCount++;
        if (currentCount >= batchSize) {
            commit();
        }
    }
    
    void commit() {
        if (currentCount > 0) {
            // Можно добавить транзакцию для пакетной вставки
            Transaction transaction(db);
            // Выполняем финальные операции
            transaction.commit();
            currentCount = 0;
        }
    }
    
    ~BatchInserter() {
        commit();
    }
};
```

### Использование индексов:

```cpp
void createIndexes() {
    // Создаем индексы для часто используемых запросов
    execute("CREATE INDEX IF NOT EXISTS idx_users_username ON users(username);");
    execute("CREATE INDEX IF NOT EXISTS idx_users_email ON users(email);");
    execute("CREATE INDEX IF NOT EXISTS idx_posts_user_id ON posts(user_id);");
    execute("CREATE INDEX IF NOT EXISTS idx_posts_created_at ON posts(created_at);");
}
```

## 9. Паттерны проектирования <a name="паттерны"></a>

### Repository Pattern

```cpp
class UserRepository {
private:
    sqlite3* db;
    
public:
    UserRepository(sqlite3* database) : db(database) {}
    
    std::optional<User> findById(int id) {
        const std::string sql = "SELECT id, username, email, created_at FROM users WHERE id = ?";
        
        try {
            PreparedStatement stmt(db, sql);
            stmt.bindInt(1, id);
            
            if (stmt.next()) {
                User user;
                user.id = stmt.getInt(0);
                user.username = stmt.getText(1);
                user.email = stmt.getText(2);
                user.createdAt = stmt.getText(3);
                return user;
            }
        } catch (const std::exception& e) {
            std::cerr << "Error finding user: " << e.what() << std::endl;
        }
        
        return std::nullopt;
    }
    
    bool save(const User& user) {
        if (user.id == 0) {
            return create(user);
        } else {
            return update(user);
        }
    }
    
private:
    bool create(const User& user) {
        const std::string sql = "INSERT INTO users (username, email) VALUES (?, ?)";
        
        try {
            PreparedStatement stmt(db, sql);
            stmt.bindText(1, user.username);
            stmt.bindText(2, user.email);
            return stmt.execute();
        } catch (const std::exception& e) {
            std::cerr << "Error creating user: " << e.what() << std::endl;
            return false;
        }
    }
    
    bool update(const User& user) {
        const std::string sql = "UPDATE users SET username = ?, email = ? WHERE id = ?";
        
        try {
            PreparedStatement stmt(db, sql);
            stmt.bindText(1, user.username);
            stmt.bindText(2, user.email);
            stmt.bindInt(3, user.id);
            return stmt.execute();
        } catch (const std::exception& e) {
            std::cerr << "Error updating user: " << e.what() << std::endl;
            return false;
        }
    }
};
```

### Unit of Work Pattern

```cpp
class UnitOfWork {
private:
    sqlite3* db;
    std::vector<std::function<void()>> operations;
    
public:
    UnitOfWork(sqlite3* database) : db(database) {}
    
    template<typename T>
    void addOperation(std::function<T()> operation) {
        operations.push_back([operation]() { operation(); });
    }
    
    bool commit() {
        Transaction transaction(db);
        
        try {
            for (auto& operation : operations) {
                operation();
            }
            
            return transaction.commit();
        } catch (const std::exception& e) {
            std::cerr << "Unit of Work commit failed: " << e.what() << std::endl;
            return false;
        }
    }
    
    void rollback() {
        operations.clear();
    }
};
```

## 10. Практические примеры <a name="примеры"></a>

### Полный пример приложения

```cpp
#include <sqlite3.h>
#include <iostream>
#include <string>
#include <vector>
#include <memory>
#include <optional>

struct User {
    int id;
    std::string username;
    std::string email;
    std::string createdAt;
};

class DatabaseManager {
private:
    sqlite3* db;
    
public:
    DatabaseManager() : db(nullptr) {}
    
    ~DatabaseManager() {
        if (db) {
            sqlite3_close(db);
        }
    }
    
    bool initialize(const std::string& filename) {
        if (sqlite3_open(filename.c_str(), &db) != SQLITE_OK) {
            std::cerr << "Cannot open database: " << sqlite3_errmsg(db) << std::endl;
            return false;
        }
        
        // Настройка базы данных
        optimizeDatabase();
        createTables();
        createIndexes();
        
        return true;
    }
    
    void optimizeDatabase() {
        execute("PRAGMA journal_mode = WAL;");
        execute("PRAGMA foreign_keys = ON;");
        execute("PRAGMA cache_size = -64000;");
        sqlite3_busy_timeout(db, 5000);
    }
    
    void createTables() {
        const char* sql = R"(
            CREATE TABLE IF NOT EXISTS users (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                username TEXT UNIQUE NOT NULL,
                email TEXT UNIQUE NOT NULL,
                created_at DATETIME DEFAULT CURRENT_TIMESTAMP
            );
        )";
        
        execute(sql);
    }
    
    void createIndexes() {
        execute("CREATE INDEX IF NOT EXISTS idx_users_username ON users(username);");
        execute("CREATE INDEX IF NOT EXISTS idx_users_email ON users(email);");
    }
    
    bool execute(const std::string& sql) {
        char* errorMessage = nullptr;
        int result = sqlite3_exec(db, sql.c_str(), nullptr, nullptr, &errorMessage);
        
        if (result != SQLITE_OK) {
            std::cerr << "SQL error: " << errorMessage << std::endl;
            sqlite3_free(errorMessage);
            return false;
        }
        
        return true;
    }
    
    sqlite3* getHandle() const { return db; }
};

class UserService {
private:
    DatabaseManager& dbManager;
    
public:
    UserService(DatabaseManager& manager) : dbManager(manager) {}
    
    std::optional<User> createUser(const std::string& username, const std::string& email) {
        const std::string sql = "INSERT INTO users (username, email) VALUES (?, ?)";
        sqlite3* db = dbManager.getHandle();
        
        try {
            sqlite3_stmt* stmt;
            if (sqlite3_prepare_v2(db, sql.c_str(), -1, &stmt, nullptr) != SQLITE_OK) {
                return std::nullopt;
            }
            
            sqlite3_bind_text(stmt, 1, username.c_str(), -1, SQLITE_TRANSIENT);
            sqlite3_bind_text(stmt, 2, email.c_str(), -1, SQLITE_TRANSIENT);
            
            if (sqlite3_step(stmt) != SQLITE_DONE) {
                sqlite3_finalize(stmt);
                return std::nullopt;
            }
            
            int userId = sqlite3_last_insert_rowid(db);
            sqlite3_finalize(stmt);
            
            return getUserById(userId);
            
        } catch (...) {
            return std::nullopt;
        }
    }
    
    std::optional<User> getUserById(int id) {
        const std::string sql = "SELECT id, username, email, created_at FROM users WHERE id = ?";
        sqlite3* db = dbManager.getHandle();
        
        try {
            sqlite3_stmt* stmt;
            if (sqlite3_prepare_v2(db, sql.c_str(), -1, &stmt, nullptr) != SQLITE_OK) {
                return std::nullopt;
            }
            
            sqlite3_bind_int(stmt, 1, id);
            
            if (sqlite3_step(stmt) == SQLITE_ROW) {
                User user;
                user.id = sqlite3_column_int(stmt, 0);
                user.username = reinterpret_cast<const char*>(sqlite3_column_text(stmt, 1));
                user.email = reinterpret_cast<const char*>(sqlite3_column_text(stmt, 2));
                user.createdAt = reinterpret_cast<const char*>(sqlite3_column_text(stmt, 3));
                
                sqlite3_finalize(stmt);
                return user;
            }
            
            sqlite3_finalize(stmt);
            return std::nullopt;
            
        } catch (...) {
            return std::nullopt;
        }
    }
    
    std::vector<User> getAllUsers() {
        const std::string sql = "SELECT id, username, email, created_at FROM users";
        sqlite3* db = dbManager.getHandle();
        std::vector<User> users;
        
        try {
            sqlite3_stmt* stmt;
            if (sqlite3_prepare_v2(db, sql.c_str(), -1, &stmt, nullptr) != SQLITE_OK) {
                return users;
            }
            
            while (sqlite3_step(stmt) == SQLITE_ROW) {
                User user;
                user.id = sqlite3_column_int(stmt, 0);
                user.username = reinterpret_cast<const char*>(sqlite3_column_text(stmt, 1));
                user.email = reinterpret_cast<const char*>(sqlite3_column_text(stmt, 2));
                user.createdAt = reinterpret_cast<const char*>(sqlite3_column_text(stmt, 3));
                users.push_back(user);
            }
            
            sqlite3_finalize(stmt);
            
        } catch (...) {
            // Логирование ошибки
        }
        
        return users;
    }
};

int main() {
    DatabaseManager dbManager;
    if (!dbManager.initialize("test.db")) {
        std::cerr << "Failed to initialize database" << std::endl;
        return 1;
    }
    
    UserService userService(dbManager);
    
    // Создаем пользователя
    auto user = userService.createUser("john_doe", "john@example.com");
    if (user) {
        std::cout << "Created user: " << user->username << std::endl;
    }
    
    // Получаем всех пользователей
    auto users = userService.getAllUsers();
    std::cout << "Total users: " << users.size() << std::endl;
    
    return 0;
}
```

## Заключение

### Ключевые моменты для запоминания:

1. **Всегда используйте подготовленные выражения** для предотвращения SQL-инъекций
2. **Обрабатывайте ошибки** на каждом этапе работы с базой данных
3. **Используйте транзакции** для группировки связанных операций
4. **Закрывайте соединения** и освобождайте ресурсы в деструкторах
5. **Оптимизируйте запросы** с помощью индексов и правильной настройки БД
6. **Разрабатывайте с учетом масштабирования** - используйте паттерны проектирования
7. **Тестируйте работу с базой данных** в различных сценариях

### Дальнейшее изучение:

- Изучите конкретную СУБД, которую планируете использовать
- Освойте ORM системы (например, ODB, SOCI)
- Изучите продвинутые темы: репликация, шардирование, бэкапы
- Практикуйтесь на реальных проектах
