# Лабораторная работа №6. Исключения и логирование (Вариант 8: Счета)

## 1. Задание
**Вариант 8. Иерархия «Счета»**

**Сущность `Account`:** `id`, `accountNumber`, `ownerName`, `balance`, `currency`.

**Исключения:**
- `AccountNotFoundException` (checked)
- `InvalidAccountDataException` (unchecked)

**Валидация:** 
- `accountNumber` не null/пустой;
- `balance` >= 0;
- `currency` не null (и не пустая).

**Дополнительные требования:**
1. Валидация в сеттерах.
2. Репозиторий с выбрасыванием исключений и логированием (INFO, WARN, DEBUG, ERROR).
3. Демонстрация `try-catch-finally` и `try-with-resources`.
4. Чтение/запись в файл.
5. Подключение SLF4J + Logback с выводом в консоль и файлы (`application.log`, `errors.log`).

---

## 2. Структура проекта

```text
Lab6/
├── pom.xml
├── accounts.csv (создается программно)
├── logs/ (создается программно)
└── src/
    └── main/
        ├── java/
        │   └── ru/lab/
        │       ├── Main.java
        │       ├── exceptions/
        │       │   ├── AccountNotFoundException.java
        │       │   └── InvalidAccountDataException.java
        │       ├── model/
        │       │   └── Account.java
        │       └── repository/
        │           └── AccountRepository.java
        └── resources/
            └── logback.xml
```

---

## 3. Коды по файлам

### `pom.xml`
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>ru.lab</groupId>
    <artifactId>lab6</artifactId>
    <version>1.0</version>

    <properties>
        <maven.compiler.release>17</maven.compiler.release>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>2.0.9</version>
        </dependency>

        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>1.4.11</version>
        </dependency>
    </dependencies>
</project>
```

### `AccountNotFoundException.java`
```java
package ru.lab.exceptions;

public class AccountNotFoundException extends Exception {

    public AccountNotFoundException(String message) {
        super(message);
    }

    public AccountNotFoundException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

### `InvalidAccountDataException.java`
```java
package ru.lab.exceptions;

public class InvalidAccountDataException extends RuntimeException {

    public InvalidAccountDataException(String message) {
        super(message);
    }

    public InvalidAccountDataException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

### `Account.java`
```java
package ru.lab.model;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import ru.lab.exceptions.InvalidAccountDataException;
import java.math.BigDecimal;

public class Account {

    private static final Logger logger =
            LoggerFactory.getLogger(Account.class);

    private long id;
    private String accountNumber;
    private String ownerName;
    private BigDecimal balance;
    private String currency;

    // Конструктор без параметров — для фреймворков/десериализации
    public Account() {
    }

    public Account(String accountNumber, String ownerName,
                   BigDecimal balance, String currency) {
        setAccountNumber(accountNumber);
        setOwnerName(ownerName);
        setBalance(balance);
        setCurrency(currency);
    }

    public long getId() { return id; }
    public void setId(long id) { this.id = id; }

    public String getAccountNumber() { return accountNumber; }
    public String getOwnerName() { return ownerName; }
    public BigDecimal getBalance() { return balance; }
    public String getCurrency() { return currency; }

    public void setAccountNumber(String accountNumber) {
        if (accountNumber == null || accountNumber.isBlank()) {
            invalid("Номер счёта не может быть пустым");
        }
        this.accountNumber = accountNumber;
    }

    public void setOwnerName(String ownerName) {
        if (ownerName == null || ownerName.isBlank()) {
            invalid("Владелец счёта не может быть пустым");
        }
        this.ownerName = ownerName;
    }

    public void setBalance(BigDecimal balance) {
        if (balance == null || balance.compareTo(BigDecimal.ZERO) < 0) {
            invalid("Баланс должен быть >= 0");
        }
        this.balance = balance;
    }

    public void setCurrency(String currency) {
        if (currency == null || currency.isBlank()) {
            invalid("Валюта не может быть null или пустой");
        }
        this.currency = currency;
    }

    private void invalid(String message) {
        logger.error("Ошибка валидации: {}", message);
        throw new InvalidAccountDataException(message);
    }

    @Override
    public String toString() {
        return "Account{id=" + id +
                ", accountNumber='" + accountNumber +
                "', ownerName='" + ownerName +
                "', balance=" + balance +
                ", currency='" + currency + "'}";
    }
}
```

### `AccountRepository.java`
```java
package ru.lab.repository;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import ru.lab.exceptions.AccountNotFoundException;
import ru.lab.model.Account;

import java.util.*;

public class AccountRepository {

    private static final Logger logger =
            LoggerFactory.getLogger(AccountRepository.class);

    private final Map<Long, Account> storage = new HashMap<>();
    private long nextId = 1;

    public Account save(Account account) {
        logger.info("Сохранение счёта: number={}", account.getAccountNumber());
        account.setId(nextId++);
        storage.put(account.getId(), account);

        logger.info("Счёт сохранён: id={}, number={}",
                account.getId(), account.getAccountNumber());

        return account;
    }

    public Account findById(long id) throws AccountNotFoundException {
        Account account = storage.get(id);

        if (account == null) {
            logger.warn("Счёт с id {} не найден", id);
            throw new AccountNotFoundException(
                    "Счёт с id " + id + " не найден");
        }

        logger.debug("Найден счёт: id={}", id);
        return account;
    }

    public void delete(long id) throws AccountNotFoundException {
        Account account = storage.remove(id);

        if (account == null) {
            logger.warn("Попытка удалить отсутствующий счёт id={}", id);
            throw new AccountNotFoundException(
                    "Счёт с id " + id + " не найден");
        }

        logger.info("Счёт удалён: id={}", id);
    }

    public Collection<Account> findAll() {
        logger.debug("Запрос всех счетов. Количество: {}", storage.size());
        return new ArrayList<>(storage.values());
    }
}
```

### `Main.java`
```java
package ru.lab;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import ru.lab.exceptions.*;
import ru.lab.model.Account;
import ru.lab.repository.AccountRepository;

import java.io.*;
import java.math.BigDecimal;
import java.nio.file.*;

public class Main {

    private static final Logger logger =
            LoggerFactory.getLogger(Main.class);

    public static void main(String[] args) {

        AccountRepository repository = new AccountRepository();

        try {
            // 1. Создание и сохранение счёта
            System.out.println("=== Сохранение счёта ===");

            Account account = new Account(
                    "40817810000000000001",
                    "Иванов И.И.",
                    new BigDecimal("1500.50"),
                    "RUB"
            );

            repository.save(account);
            System.out.println(account);

            // 2. Поиск несуществующего счёта
            System.out.println("\n=== Поиск счёта ===");

            try {
                repository.findById(999);
            } catch (AccountNotFoundException e) {
                System.out.println(e.getMessage());
            }

            // 3. Проверки валидации — каждая отдельно
            System.out.println("\n=== Проверка валидации ===");

            // 3.1 пустой accountNumber
            try {
                new Account("", "Петров П.П.",
                        new BigDecimal("100"), "RUB");
            } catch (InvalidAccountDataException e) {
                logger.error("Ошибка валидации accountNumber: {}", e.getMessage());
            }

            // 3.2 пустой ownerName
            try {
                new Account("40817...0002", "",
                        new BigDecimal("100"), "RUB");
            } catch (InvalidAccountDataException e) {
                logger.error("Ошибка валидации ownerName: {}", e.getMessage());
            }

            // 3.3 отрицательный баланс
            try {
                new Account("40817...0003", "Сидоров С.С.",
                        new BigDecimal("-100"), "RUB");
            } catch (InvalidAccountDataException e) {
                logger.error("Ошибка валидации balance: {}", e.getMessage());
            }

            // 3.4 null валюта
            try {
                new Account("40817...0004", "Кузнецов К.К.",
                        new BigDecimal("50"), null);
            } catch (InvalidAccountDataException e) {
                logger.error("Ошибка валидации currency: {}", e.getMessage());
            }

            // 4. Запись в файл (try-with-resources)
            System.out.println("\n=== Запись в файл (try-with-resources) ===");
            writeAccounts("accounts.csv", repository);

            // 5. Чтение из файла (try-with-resources + try-catch-finally внутри)
            System.out.println("\n=== Чтение из файла ===");
            loadAccounts("accounts.csv", repository);

            // 6. Удаление счёта
            System.out.println("\n=== Удаление счёта ===");
            repository.delete(account.getId());

            // 7. Попытка удалить несуществующий счёт
            try {
                repository.delete(999);
            } catch (AccountNotFoundException e) {
                System.out.println(e.getMessage());
            }

        } catch (Exception e) {
            logger.error("Ошибка программы", e);
        } finally {
            System.out.println("\nПрограмма завершена.");
        }
    }

    /** Запись счетов в файл через try-with-resources. */
    private static void writeAccounts(
            String path, AccountRepository repository) {

        try (BufferedWriter writer =
                     Files.newBufferedWriter(Path.of(path))) {

            for (Account account : repository.findAll()) {
                writer.write(
                        account.getAccountNumber() + ";" +
                                account.getOwnerName() + ";" +
                                account.getBalance() + ";" +
                                account.getCurrency()
                );
                writer.newLine();
            }

            logger.info("Данные записаны в файл {}", path);

        } catch (IOException e) {
            logger.error("Ошибка записи файла", e);
        }
    }

    /** Чтение счетов из файла с классическим try-catch-finally для демонстрации. */
    private static void loadAccounts(
            String path, AccountRepository repository) {

        BufferedReader reader = null;

        try {
            reader = Files.newBufferedReader(Path.of(path));

            String line;
            int success = 0;
            int errors = 0;

            while ((line = reader.readLine()) != null) {

                try {
                    String[] data = line.split(";", -1);

                    if (data.length != 4) {
                        throw new InvalidAccountDataException(
                                "Неверный формат строки");
                    }

                    Account account = new Account(
                            data[0],
                            data[1],
                            new BigDecimal(data[2]),
                            data[3]
                    );

                    repository.save(account);
                    success++;

                } catch (RuntimeException e) {
                    errors++;
                    logger.error("Ошибка обработки строки: {}", line, e);
                }
            }

            logger.info("Чтение файла завершено: успешно={}, ошибок={}",
                    success, errors);

        } catch (IOException e) {
            logger.error("Ошибка чтения файла {}", path, e);
        } finally {
            // Ручное закрытие — демонстрация finally
            if (reader != null) {
                try {
                    reader.close();
                } catch (IOException e) {
                    logger.error("Ошибка закрытия reader", e);
                }
            }
            logger.info("Завершение обработки файла (finally)");
        }
    }
}
```

### `logback.xml`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>

    <!-- Вывод в консоль -->
    <appender name="CONSOLE"
              class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{HH:mm:ss} %-5level %logger{30} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- Запись в файл -->
    <appender name="FILE"
              class="ch.qos.logback.core.FileAppender">
        <file>logs/application.log</file>
        <append>true</append>
        <encoder>
            <pattern>%d{HH:mm:ss} %-5level %logger - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- Файл ошибок -->
    <appender name="ERROR_FILE"
              class="ch.qos.logback.core.FileAppender">
        <file>logs/errors.log</file>
        <append>true</append>

        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>ERROR</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>

        <encoder>
            <pattern>%d{HH:mm:ss} %-5level %logger - %msg%n</pattern>
        </encoder>
    </appender>

    <root level="DEBUG">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="FILE"/>
        <appender-ref ref="ERROR_FILE"/>
    </root>

</configuration>
```

---

## 4. Вывод программы

```text
=== Сохранение счёта ===
18:47:48 INFO  r.l.r.AccountRepository - Сохранение счёта: number=40817810000000000001
18:47:48 INFO  r.l.r.AccountRepository - Счёт сохранён: id=1, number=40817810000000000001
Account{id=1, accountNumber='40817810000000000001', ownerName='Иванов И.И.', balance=1500.50, currency='RUB'}

=== Поиск счёта ===
18:47:48 WARN  r.l.r.AccountRepository - Счёт с id 999 не найден
Счёт с id 999 не найден

=== Проверка валидации ===
18:47:48 ERROR ru.lab.model.Account - Ошибка валидации: Номер счёта не может быть пустым
18:47:48 ERROR ru.lab.Main - Ошибка валидации accountNumber: Номер счёта не может быть пустым
18:47:48 ERROR ru.lab.model.Account - Ошибка валидации: Владелец счёта не может быть пустым
18:47:48 ERROR ru.lab.Main - Ошибка валидации ownerName: Владелец счёта не может быть пустым
18:47:48 ERROR ru.lab.model.Account - Ошибка валидации: Баланс должен быть >= 0
18:47:48 ERROR ru.lab.Main - Ошибка валидации balance: Баланс должен быть >= 0
18:47:48 ERROR ru.lab.model.Account - Ошибка валидации: Валюта не может быть null или пустой
18:47:48 ERROR ru.lab.Main - Ошибка валидации currency: Валюта не может быть null или пустой

=== Запись в файл (try-with-resources) ===
18:47:48 DEBUG r.l.r.AccountRepository - Запрос всех счетов. Количество: 1
18:47:48 INFO  ru.lab.Main - Данные записаны в файл accounts.csv

=== Чтение из файла ===
18:47:48 INFO  r.l.r.AccountRepository - Сохранение счёта: number=40817810000000000001
18:47:48 INFO  r.l.r.AccountRepository - Счёт сохранён: id=2, number=40817810000000000001
18:47:48 INFO  ru.lab.Main - Чтение файла завершено: успешно=1, ошибок=0
18:47:48 INFO  ru.lab.Main - Завершение обработки файла (finally)

=== Удаление счёта ===
18:47:48 INFO  r.l.r.AccountRepository - Счёт удалён: id=1
18:47:48 WARN  r.l.r.AccountRepository - Попытка удалить отсутствующий счёт id=999
Счёт с id 999 не найден

Программа завершена.
```

---

## 5. Ответы на контрольные вопросы

**1. Что такое иерархия исключений в Java? Назовите основные классы.**
Все исключения наследуются от `Throwable`. Основные ветви: `Error` (критические ошибки JVM, например `OutOfMemoryError`) и `Exception` (обрабатываемые исключения, например `IOException`). Внутри `Exception` выделяют `RuntimeException` (unchecked-исключения, например `NullPointerException`).

**2. В чём разница между checked и unchecked исключениями?**
Checked (наследники `Exception`, кроме `RuntimeException`) проверяются компилятором: их нужно либо обработать (`try-catch`), либо пробросить (`throws`). Unchecked (наследники `RuntimeException`) не требуют обязательной обработки, обычно сигнализируют об ошибках программиста.

**3. Что такое try-with-resources и как он работает?**
Конструкция (Java 7+), которая автоматически закрывает ресурсы, реализующие интерфейс `AutoCloseable`. Ресурсы объявляются в круглых скобках после `try`. Закрытие происходит автоматически в обратном порядке, даже если возникло исключение.

**4. В чём отличие между ключевыми словами throw и throws?**
`throw` — оператор, который непосредственно выбрасывает объект-исключение. `throws` — модификатор в сигнатуре метода, который предупреждает компилятор и вызывающий код о том, что метод может выбросить указанные checked-исключения.

**5. Как создать собственное checked-исключение?**
Создать класс, который наследуется от `Exception`. В нем нужно объявить конструкторы, вызывающие `super(message)` и `super(message, cause)`.

**6. Как создать собственное unchecked-исключение?**
Создать класс, который наследуется от `RuntimeException`. Конструкторы аналогичны: вызывают `super(message)` и `super(message, cause)`.

**7. Что такое SLF4J и Logback?**
SLF4J — это фасад (абстрактный интерфейс) для логирования. Logback — это конкретная реализация (библиотека) логирования, которая подключается к SLF4J и выполняет фактическую запись логов.

**8. Перечислите уровни логирования и опишите, когда какой использовать.**
- ERROR: критические ошибки, требующие вмешательства.
- WARN: потенциальные проблемы (например, не найден объект).
- INFO: важные события (успешные операции).
- DEBUG: детальная информация для отладки.
- TRACE: сверхдетальная информация (вывод каждого шага).

**9. Почему важно логировать операции в приложении?**
Логирование позволяет отслеживать работу программы, диагностировать ошибки в процессе эксплуатации (когда нет доступа к отладчику), а также проводить аудит действий пользователей и системы.

**10. Что такое цепочка причин (cause) в исключениях?**
Это механизм сохранения исходного исключения при оборачивании его в новое. Позволяет не потерять информацию о первопричине ошибки (стек-трейс). Передается через конструктор `(String message, Throwable cause)` и извлекается методом `getCause()`.

**11. В чём отличие finally от try-with-resources?**
Блок `finally` выполняется всегда, но требует ручного закрытия ресурсов (что может привести к затиранию исходного исключения, если в `finally` тоже выбросить ошибку). `try-with-resources` закрывает ресурсы автоматически и безопасно, а исключения при закрытии добавляются к основному через `getSuppressed()`.

**12. Как настроить логирование в файл?**
В файле конфигурации `logback.xml` необходимо объявить `FileAppender` (или `RollingFileAppender`), указать в теге `<file>` путь к файлу, настроить `<encoder>` для формата вывода и подключить этот appender к тегу `<root>`.
