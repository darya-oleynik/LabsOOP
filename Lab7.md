# ЛАБОРАТОРНАЯ РАБОТА №7

**Тема:** Файловый ввод-вывод: чтение и запись CSV-файлов в Java.

**Индивидуальное задание:** вариант 8 — «Счета».

## 1. Условие задания

Разработать программу для обработки CSV-файла со сведениями о банковских счетах.

Формат записи:

id,accountNumber,ownerName,balance,currency,openedDate

Необходимо:
- Прочитать файл двумя способами: с помощью `BufferedReader` и `Files.readAllLines()` совместно со Stream API.
- Проверить корректность данных: баланс неотрицательный, номер счёта не пустой, валюта — только `USD`, `EUR` или `RUB`.
- Пропускать некорректные записи и учитывать типы ошибок.
- Обнаруживать дубликаты номеров счетов.
- Конвертировать баланс в рубли по заданным учебным курсам.
- Записать корректные уникальные счета в отдельный CSV-файл.

## 2. Теоретические сведения

Для чтения текстовых файлов в Java применяются символьные потоки. `BufferedReader` обеспечивает буферизированное чтение и позволяет получать файл построчно с помощью `readLine()`.

Класс `Files` из пакета `java.nio.file` предоставляет методы для работы с файлами: `readAllLines()` загружает все строки в список, а `newBufferedReader()` открывает буферизированный поток чтения.

Stream API позволяет обрабатывать коллекции декларативно с помощью операций `filter`, `map`, `collect` и других. При использовании `Files.readAllLines()` весь файл сначала загружается в память, поэтому такой способ удобен для небольших файлов, а построчное чтение лучше подходит для больших файлов.

CSV — текстовый формат табличных данных. Поля обычно разделяются запятыми. Если внутри поля есть запятая или кавычка, поле заключается в двойные кавычки, а кавычка внутри поля записывается удвоением (`""`). Поэтому простой вызов `split(",")` не обеспечивает корректный разбор всех CSV-записей.

Конструкция `try-with-resources` автоматически закрывает потоки после завершения работы. Для корректной обработки текста рекомендуется явно указывать кодировку, например `UTF-8`.

## 3. Код программы

### 3.1. Файл `src/main/java/Main.java`

```java
import model.Account;
import parser.AccountCsvParser;
import parser.AccountCsvParser.ParseResult;
import util.CurrencyConverter;

import java.io.IOException;
import java.nio.file.Path;
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

public class Main {
    private static final Path INPUT_FILE = Path.of("src/main/resources/accounts.csv");
    private static final Path OUTPUT_FILE = Path.of("src/main/resources/accounts_clean.csv");

    public static void main(String[] args) {
        try {
            System.out.println("=== Лабораторная работа №7. Вариант 8: Счета ===\n");
            System.out.println("Курсы для учебной конвертации: 1 USD = " + CurrencyConverter.getRate("USD")
                    + " RUB; 1 EUR = " + CurrencyConverter.getRate("EUR") + " RUB; 1 RUB = 1 RUB.\n");

            long start = System.nanoTime();
            ParseResult buffered = AccountCsvParser.parseBufferedReader(INPUT_FILE);
            long bufferedTime = (System.nanoTime() - start) / 1_000_000;
            System.out.println("\n1. Чтение через BufferedReader:");
            System.out.println(buffered.getStatistics());
            System.out.println("Время выполнения: " + bufferedTime + " мс");

            start = System.nanoTime();
            ParseResult stream = AccountCsvParser.parseStream(INPUT_FILE);
            long streamTime = (System.nanoTime() - start) / 1_000_000;
            System.out.println("\n2. Чтение через Files.readAllLines() + Stream API:");
            System.out.println(stream.getStatistics());
            System.out.println("Время выполнения: " + streamTime + " мс");

            List<Account> accounts = buffered.getAccounts();
            AccountCsvParser.writeCleanCsv(accounts, OUTPUT_FILE);
            System.out.println("\n3. Конвертация валют и запись результата:");
            System.out.printf(java.util.Locale.US, "Общий баланс всех корректных счетов в RUB: %.2f RUB%n",
                    AccountCsvParser.totalBalanceInRub(accounts));
            Map<String, Double> totals = accounts.stream().collect(Collectors.groupingBy(
                    Account::getCurrency, Collectors.summingDouble(Account::getBalance)));
            System.out.println("Сумма по исходным валютам: " + totals);
            System.out.println("Результат сохранён в: " + OUTPUT_FILE);

            System.out.println("\n4. Сравнение подходов:");
            System.out.println("BufferedReader — построчное чтение, не требует загружать весь файл в память.");
            System.out.println("Files.readAllLines() + Stream API — компактная обработка коллекции строк, "
                    + "но весь файл сначала загружается в память.");
        } catch (IOException e) {
            System.err.println("Ошибка при работе с файлом: " + e.getMessage());
        }
    }
}
````

### **3.2. Файл&#x20;**`src/main/java/model/Account.java`

```
package model;

import java.time.LocalDate;

public class Account {
    private final long id;
    private final String accountNumber;
    private final String ownerName;
    private final double balance;
    private final String currency;
    private final LocalDate openedDate;

    public Account(long id, String accountNumber, String ownerName,
                   double balance, String currency, LocalDate openedDate) {
        this.id = id;
        this.accountNumber = accountNumber;
        this.ownerName = ownerName;
        this.balance = balance;
        this.currency = currency;
        this.openedDate = openedDate;
    }

    public long getId() { return id; }
    public String getAccountNumber() { return accountNumber; }
    public String getOwnerName() { return ownerName; }
    public double getBalance() { return balance; }
    public String getCurrency() { return currency; }
    public LocalDate getOpenedDate() { return openedDate; }

    @Override
    public String toString() {
        return "Account{id=" + id + ", accountNumber='" + accountNumber
                + "', ownerName='" + ownerName + "', balance=" + balance
                + " " + currency + ", openedDate=" + openedDate + "}";
    }
}
```

### **3.3. Файл&#x20;**`src/main/java/parser/CsvParseException.java`

```
package parser;

public class CsvParseException extends Exception {
    private final String type;

    public CsvParseException(String type, String message) {
        super(message);
        this.type = type;
    }

    public String getType() { return type; }
}
```

### **3.4. Файл&#x20;**`src/main/java/parser/AccountCsvParser.java`

```
package parser;

import model.Account;
import statistics.CsvStatistics;
import util.CurrencyConverter;

import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.nio.file.Files;
import java.nio.file.Path;
import java.time.LocalDate;
import java.time.format.DateTimeParseException;
import java.util.ArrayList;
import java.util.HashSet;
import java.util.List;
import java.util.Set;

public final class AccountCsvParser {
    public static final String HEADER = "id,accountNumber,ownerName,balance,currency,openedDate";

    private AccountCsvParser() { }

    public static class ParseResult {
        private final List<Account> accounts;
        private final CsvStatistics statistics;

        public ParseResult(List<Account> accounts, CsvStatistics statistics) {
            this.accounts = accounts;
            this.statistics = statistics;
        }

        public List<Account> getAccounts() { return accounts; }
        public CsvStatistics getStatistics() { return statistics; }
    }

    public static ParseResult parseBufferedReader(Path path) throws IOException {
        List<Account> accounts = new ArrayList<>();
        CsvStatistics stats = new CsvStatistics();
        Set<String> accountNumbers = new HashSet<>();
        int totalLines = 0;

        try (BufferedReader reader = Files.newBufferedReader(path, StandardCharsets.UTF_8)) {
            String header = reader.readLine();
            if (header == null) throw new IOException("CSV-файл пустой");
            totalLines++;

            if (!HEADER.equals(header.trim()))
                throw new IOException("Неверный заголовок CSV: " + header);

            String line;
            while ((line = reader.readLine()) != null) {
                totalLines++;
                processLine(line, totalLines, accounts, accountNumbers, stats);
            }
        }

        stats.setTotalLines(totalLines);
        return new ParseResult(accounts, stats);
    }

    public static ParseResult parseStream(Path path) throws IOException {
        List<String> lines = Files.readAllLines(path, StandardCharsets.UTF_8);

        if (lines.isEmpty()) throw new IOException("CSV-файл пустой");
        if (!HEADER.equals(lines.get(0).trim()))
            throw new IOException("Неверный заголовок CSV: " + lines.get(0));

        CsvStatistics stats = new CsvStatistics();
        Set<String> accountNumbers = new HashSet<>();
        List<Account> accounts = new ArrayList<>();

        lines.stream().skip(1).forEachOrdered(line ->
                processLine(line, 0, accounts, accountNumbers, stats));

        stats.setTotalLines(lines.size());
        return new ParseResult(accounts, stats);
    }

    private static void processLine(String line, int lineNumber, List<Account> accounts,
                                    Set<String> accountNumbers, CsvStatistics stats) {
        try {
            List<String> fields = parseCsvLine(line);
            Account account = createAccount(fields);

            if (!accountNumbers.add(account.getAccountNumber())) {
                stats.incrementDuplicate();
                System.out.println("Дубликат номера счёта: " + account.getAccountNumber()
                        + (lineNumber > 0 ? " (строка " + lineNumber + ")" : ""));
                return;
            }

            accounts.add(account);
            stats.incrementSuccess();

        } catch (CsvParseException e) {
            stats.incrementError(e.getType());
            System.out.println("Строка " + (lineNumber > 0 ? lineNumber : "Stream")
                    + " пропущена: " + e.getMessage());
        }
    }

    public static List<String> parseCsvLine(String line) throws CsvParseException {
        List<String> fields = new ArrayList<>();
        StringBuilder field = new StringBuilder();
        boolean inQuotes = false;
        boolean quoteClosed = false;

        for (int i = 0; i < line.length(); i++) {
            char c = line.charAt(i);

            if (inQuotes) {
                if (c == '"') {
                    if (i + 1 < line.length() && line.charAt(i + 1) == '"') {
                        field.append('"');
                        i++;
                    } else {
                        inQuotes = false;
                        quoteClosed = true;
                    }
                } else {
                    field.append(c);
                }
            } else if (c == ',') {
                fields.add(field.toString().trim());
                field.setLength(0);
                quoteClosed = false;
            } else if (c == '"' && field.toString().trim().isEmpty() && !quoteClosed) {
                field.setLength(0);
                inQuotes = true;
            } else {
                if (quoteClosed && !Character.isWhitespace(c)) {
                    throw new CsvParseException("Некорректный CSV",
                            "символ после закрывающей кавычки");
                }
                field.append(c);
            }
        }

        if (inQuotes)
            throw new CsvParseException("Некорректный CSV", "незакрытая кавычка");

        fields.add(field.toString().trim());
        return fields;
    }

    private static Account createAccount(List<String> fields) throws CsvParseException {
        if (fields.size() != 6) {
            throw new CsvParseException("Количество полей",
                    "ожидалось 6 полей, получено " + fields.size());
        }

        try {
            long id = Long.parseLong(fields.get(0));
            String accountNumber = fields.get(1);
            String ownerName = fields.get(2);
            double balance = Double.parseDouble(fields.get(3));
            String currency = fields.get(4).toUpperCase();
            LocalDate openedDate = LocalDate.parse(fields.get(5));

            if (accountNumber.isBlank())
                throw new CsvParseException("Пустой номер счёта",
                        "accountNumber не должен быть пустым");

            if (balance < 0 || !Double.isFinite(balance))
                throw new CsvParseException("Некорректный баланс",
                        "balance должен быть >= 0");

            if (!List.of("USD", "EUR", "RUB").contains(currency))
                throw new CsvParseException("Некорректная валюта",
                        "допустимы только USD, EUR, RUB");

            if (id <= 0)
                throw new CsvParseException("Некорректный ID",
                        "id должен быть положительным");

            if (ownerName.isBlank())
                throw new CsvParseException("Пустое имя владельца",
                        "ownerName не должно быть пустым");

            CurrencyConverter.getRate(currency);

            return new Account(id, accountNumber, ownerName,
                    balance, currency, openedDate);

        } catch (NumberFormatException e) {
            throw new CsvParseException("Ошибка числа",
                    "id или balance имеет неверный формат");
        } catch (DateTimeParseException e) {
            throw new CsvParseException("Ошибка даты",
                    "openedDate должна иметь формат yyyy-MM-dd");
        }
    }

    public static void writeCleanCsv(List<Account> accounts, Path output) throws IOException {
        try (BufferedWriter writer = Files.newBufferedWriter(output, StandardCharsets.UTF_8)) {
            writer.write(HEADER + ",balanceInRUB");
            writer.newLine();

            for (Account account : accounts) {
                writer.write(account.getId() + ","
                        + csv(account.getAccountNumber()) + ","
                        + csv(account.getOwnerName()) + ","
                        + account.getBalance() + ","
                        + account.getCurrency() + ","
                        + account.getOpenedDate() + ","
                        + String.format(java.util.Locale.US, "%.2f",
                        CurrencyConverter.toRub(account.getBalance(), account.getCurrency())));
                writer.newLine();
            }
        }
    }

    private static String csv(String value) {
        String escaped = value.replace("\"", "\"\"");
        return "\"" + escaped + "\"";
    }

    public static double totalBalanceInRub(List<Account> accounts) {
        return accounts.stream()
                .mapToDouble(a -> CurrencyConverter.toRub(a.getBalance(), a.getCurrency()))
                .sum();
    }
}
```

### **3.5. Файл&#x20;**`src/main/java/statistics/CsvStatistics.java`

```
package statistics;

import java.util.LinkedHashMap;
import java.util.Map;

public class CsvStatistics {
    private int totalLines;
    private int successCount;
    private int errorCount;
    private int duplicateCount;
    private final Map<String, Integer> errorTypes = new LinkedHashMap<>();

    public void setTotalLines(int totalLines) {
        this.totalLines = totalLines;
    }

    public void incrementSuccess() {
        successCount++;
    }

    public void incrementDuplicate() {
        duplicateCount++;
    }

    public void incrementError(String type) {
        errorCount++;
        errorTypes.put(type, errorTypes.getOrDefault(type, 0) + 1);
    }

    public int getTotalLines() { return totalLines; }
    public int getSuccessCount() { return successCount; }
    public int getErrorCount() { return errorCount; }
    public int getDuplicateCount() { return duplicateCount; }
    public Map<String, Integer> getErrorTypes() { return errorTypes; }

    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder("=== Статистика обработки ===\n");
        sb.append("Всего строк (включая заголовок): ").append(totalLines).append('\n');
        sb.append("Успешно прочитано: ").append(successCount).append('\n');
        sb.append("Пропущено с ошибками: ").append(errorCount).append('\n');
        sb.append("Дубликатов по номеру счёта: ").append(duplicateCount).append('\n');

        if (!errorTypes.isEmpty()) {
            sb.append("Типы ошибок:\n");
            errorTypes.forEach((type, count) ->
                    sb.append("  - ").append(type).append(": ").append(count).append('\n'));
        }

        return sb.toString();
    }
}
```

### **3.6. Файл&#x20;**`src/main/java/util/CurrencyConverter.java`

```
package util;

import java.util.Map;

public final class CurrencyConverter {
    private static final Map<String, Double> RUB_RATES = Map.of(
            "RUB", 1.0,
            "USD", 90.0,
            "EUR", 98.0
    );

    private CurrencyConverter() { }

    public static double toRub(double amount, String currency) {
        Double rate = RUB_RATES.get(currency);

        if (rate == null)
            throw new IllegalArgumentException("Неизвестная валюта: " + currency);

        return amount * rate;
    }

    public static double getRate(String currency) {
        Double rate = RUB_RATES.get(currency);

        if (rate == null)
            throw new IllegalArgumentException("Неизвестная валюта: " + currency);

        return rate;
    }
}
```

### **3.7. Файл&#x20;**`accounts.csv`

Расположение: `src/main/resources/accounts.csv`

Данный файл содержит исходные данные о банковских счетах. В нём присутствуют как корректные записи, так и записи с ошибками для проверки обработки некорректных данных.

```
id,accountNumber,ownerName,balance,currency,openedDate
1,ACCT-100001,Иванов Иван Иванович,1517.73,USD,2020-01-01
2,ACCT-100002,Петрова Анна Сергеевна,1834.96,EUR,2021-02-02
3,ACCT-100003,Сидоров Алексей Павлович,18752.25,RUB,2022-03-03
4,ACCT-100004,Смирнова Мария Олеговна,2469.42,USD,2023-04-04
5,ACCT-100005,Кузнецов Дмитрий Игоревич,2786.65,EUR,2024-05-05
6,ACCT-100006,Попова Елена Андреевна,22504.50,RUB,2025-06-06
7,ACCT-100007,"Иванов, Сергей Петрович",3421.11,USD,2020-07-07
8,ACCT-100008,Соколова Ольга Викторовна,3738.34,EUR,2021-08-08
9,ACCT-100009,Михайлов Артём Денисович,26256.75,RUB,2022-09-09
10,ACCT-100010,Фёдорова Наталья Петровна,4372.80,USD,2023-10-10
11,ACCT-100011,Иванов Иван Иванович,4690.03,EUR,2024-11-11
12,ACCT-100012,"Петрова ""Анна"" Сергеевна",30009.00,RUB,2025-12-12
13,ACCT-100013,Сидоров Алексей Павлович,5324.49,USD,2020-01-13
14,ACCT-100014,Смирнова Мария Олеговна,5641.72,EUR,2021-02-14
15,ACCT-100015,Кузнецов Дмитрий Игоревич,33761.25,RUB,2022-03-15
16,ACCT-100016,Попова Елена Андреевна,6276.18,USD,2023-04-16
17,ACCT-100017,Васильев Максим Романович,6593.41,EUR,2024-05-17
18,ACCT-100018,Соколова Ольга Викторовна,37513.50,RUB,2025-06-18
19,ACCT-100019,Михайлов Артём Денисович,7227.87,USD,2020-07-19
20,ACCT-100020,Фёдорова Наталья Петровна,7545.10,EUR,2021-08-20
21,ACCT-100021,Иванов Иван Иванович,41265.75,RUB,2022-09-21
22,ACCT-100022,Петрова Анна Сергеевна,8179.56,USD,2023-10-22
23,ACCT-100023,Сидоров Алексей Павлович,8496.79,EUR,2024-11-23
24,ACCT-100024,Смирнова Мария Олеговна,45018.00,RUB,2025-12-24
25,ACCT-100025,Кузнецов Дмитрий Игоревич,9131.25,USD,2020-01-25
26,ACCT-100026,Попова Елена Андреевна,9448.48,EUR,2021-02-26
27,ACCT-100027,Васильев Максим Романович,48770.25,RUB,2022-03-27
28,ACCT-100028,Соколова Ольга Викторовна,10082.94,USD,2023-04-01
29,ACCT-100029,Михайлов Артём Денисович,10400.17,EUR,2024-05-02
30,ACCT-100030,Фёдорова Наталья Петровна,52522.50,RUB,2025-06-03
31,ACCT-100031,Иванов Иван Иванович,11034.63,USD,2020-07-04
32,ACCT-100032,Петрова Анна Сергеевна,11351.86,EUR,2021-08-05
33,ACCT-100033,Сидоров Алексей Павлович,56274.75,RUB,2022-09-06
34,ACCT-100034,Смирнова Мария Олеговна,11986.32,USD,2023-10-07
35,ACCT-100035,Кузнецов Дмитрий Игоревич,12303.55,EUR,2024-11-08
36,ACCT-100036,Попова Елена Андреевна,60027.00,RUB,2025-12-09
37,ACCT-100037,Васильев Максим Романович,12938.01,USD,2020-01-10
38,ACCT-100038,Соколова Ольга Викторовна,13255.24,EUR,2021-02-11
39,ACCT-100039,Михайлов Артём Денисович,63779.25,RUB,2022-03-12
40,ACCT-100040,Фёдорова Наталья Петровна,13889.70,USD,2023-04-13
41,ACCT-100041,Иванов Иван Иванович,14206.93,EUR,2024-05-14
42,ACCT-100042,Петрова Анна Сергеевна,67531.50,RUB,2025-06-15
43,ACCT-100043,Сидоров Алексей Павлович,14841.39,USD,2020-07-16
44,ACCT-100044,Смирнова Мария Олеговна,15158.62,EUR,2021-08-17
45,ACCT-100045,Кузнецов Дмитрий Игоревич,71283.75,RUB,2022-09-18
46,ACCT-100046,Попова Елена Андреевна,15793.08,USD,2023-10-19
47,ACCT-100047,Васильев Максим Романович,16110.31,EUR,2024-11-20
48,ACCT-100048,Соколова Ольга Викторовна,75036.00,RUB,2025-12-21
49,ACCT-100049,Михайлов Артём Денисович,16744.77,USD,2020-01-22
50,ACCT-100050,Фёдорова Наталья Петровна,17062.00,EUR,2021-02-23
51,ACCT-100051,Иванов Иван Иванович,78788.25,RUB,2022-03-24
52,ACCT-100052,Петрова Анна Сергеевна,17696.46,USD,2023-04-25
53,ACCT-100053,Сидоров Алексей Павлович,18013.69,EUR,2024-05-26
54,ACCT-100054,Смирнова Мария Олеговна,82540.50,RUB,2025-06-27
55,ACCT-100055,Кузнецов Дмитрий Игоревич,18648.15,USD,2020-07-01
56,ACCT-100056,"Некорректный баланс",-150.00,RUB,2024-02-10
57,,"Пустой номер",500.00,USD,2024-03-11
58,ACCT-100058,"Неизвестная валюта",100.00,GBP,2024-04-12
59,ACCT-100059,"Неверное число",abc,EUR,2024-05-13
60,ACCT-100060,"Недостаточно полей",100.00,RUB
61,ACCT-100001,"Дубликат первого счёта",999.00,USD,2024-06-14
```

### **3.8. Файл&#x20;**`accounts_clean.csv`

Расположение: `src/main/resources/accounts_clean.csv`

Данный файл содержит результаты обработки исходных данных. В нём оставлены корректные записи, добавлен столбец `balanceInRUB` с балансом, пересчитанным в рубли.

```
id,accountNumber,ownerName,balance,currency,openedDate,balanceInRUB
1,"ACCT-100001","Иванов Иван Иванович",1517.73,USD,2020-01-01,136595.70
2,"ACCT-100002","Петрова Анна Сергеевна",1834.96,EUR,2021-02-02,179826.08
3,"ACCT-100003","Сидоров Алексей Павлович",18752.25,RUB,2022-03-03,18752.25
4,"ACCT-100004","Смирнова Мария Олеговна",2469.42,USD,2023-04-04,222247.80
5,"ACCT-100005","Кузнецов Дмитрий Игоревич",2786.65,EUR,2024-05-05,273091.70
6,"ACCT-100006","Попова Елена Андреевна",22504.5,RUB,2025-06-06,22504.50
7,"ACCT-100007","Иванов, Сергей Петрович",3421.11,USD,2020-07-07,307899.90
8,"ACCT-100008","Соколова Ольга Викторовна",3738.34,EUR,2021-08-08,366357.32
9,"ACCT-100009","Михайлов Артём Денисович",26256.75,RUB,2022-09-09,26256.75
10,"ACCT-100010","Фёдорова Наталья Петровна",4372.8,USD,2023-10-10,393552.00
11,"ACCT-100011","Иванов Иван Иванович",4690.03,EUR,2024-11-11,459622.94
12,"ACCT-100012","Петрова ""Анна"" Сергеевна",30009.0,RUB,2025-12-12,30009.00
13,"ACCT-100013","Сидоров Алексей Павлович",5324.49,USD,2020-01-13,479204.10
14,"ACCT-100014","Смирнова Мария Олеговна",5641.72,EUR,2021-02-14,552888.56
15,"ACCT-100015","Кузнецов Дмитрий Игоревич",33761.25,RUB,2022-03-15,33761.25
16,"ACCT-100016","Попова Елена Андреевна",6276.18,USD,2023-04-16,564856.20
17,"ACCT-100017","Васильев Максим Романович",6593.41,EUR,2024-05-17,646154.18
18,"ACCT-100018","Соколова Ольга Викторовна",37513.5,RUB,2025-06-18,37513.50
19,"ACCT-100019","Михайлов Артём Денисович",7227.87,USD,2020-07-19,650508.30
20,"ACCT-100020","Фёдорова Наталья Петровна",7545.1,EUR,2021-08-20,739419.80
21,"ACCT-100021","Иванов Иван Иванович",41265.75,RUB,2022-09-21,41265.75
22,"ACCT-100022","Петрова Анна Сергеевна",8179.56,USD,2023-10-22,736160.40
23,"ACCT-100023","Сидоров Алексей Павлович",8496.79,EUR,2024-11-23,832685.42
24,"ACCT-100024","Смирнова Мария Олеговна",45018.0,RUB,2025-12-24,45018.00
25,"ACCT-100025","Кузнецов Дмитрий Игоревич",9131.25,USD,2020-01-25,821812.50
26,"ACCT-100026","Попова Елена Андреевна",9448.48,EUR,2021-02-26,925951.04
27,"ACCT-100027","Васильев Максим Романович",48770.25,RUB,2022-03-27,48770.25
28,"ACCT-100028","Соколова Ольга Викторовна",10082.94,USD,2023-04-01,907464.60
29,"ACCT-100029","Михайлов Артём Денисович",10400.17,EUR,2024-05-02,1019216.66
30,"ACCT-100030","Фёдорова Наталья Петровна",52522.5,RUB,2025-06-03,52522.50
31,"ACCT-100031","Иванов Иван Иванович",11034.63,USD,2020-07-04,993116.70
32,"ACCT-100032","Петрова Анна Сергеевна",11351.86,EUR,2021-08-05,1112482.28
33,"ACCT-100033","Сидоров Алексей Павлович",56274.75,RUB,2022-09-06,56274.75
34,"ACCT-100034","Смирнова Мария Олеговна",11986.32,USD,2023-10-07,1078768.80
35,"ACCT-100035","Кузнецов Дмитрий Игоревич",12303.55,EUR,2024-11-08,1205747.90
36,"ACCT-100036","Попова Елена Андреевна",60027.0,RUB,2025-12-09,60027.00
37,"ACCT-100037","Васильев Максим Романович",12938.01,USD,2020-01-10,1164420.90
38,"ACCT-100038","Соколова Ольга Викторовна",13255.24,EUR,2021-02-11,1299013.52
39,"ACCT-100039","Михайлов Артём Денисович",63779.25,RUB,2022-03-12,63779.25
40,"ACCT-100040","Фёдорова Наталья Петровна",13889.7,USD,2023-04-13,1250073.00
41,"ACCT-100041","Иванов Иван Иванович",14206.93,EUR,2024-05-14,1392279.14
42,"ACCT-100042","Петрова Анна Сергеевна",67531.5,RUB,2025-06-15,67531.50
43,"ACCT-100043","Сидоров Алексей Павлович",14841.39,USD,2020-07-16,1335725.10
44,"ACCT-100044","Смирнова Мария Олеговна",15158.62,EUR,2021-08-17,1485544.76
45,"ACCT-100045","Кузнецов Дмитрий Игоревич",71283.75,RUB,2022-09-18,71283.75
46,"ACCT-100046","Попова Елена Андреевна",15793.08,USD,2023-10-19,1421377.20
47,"ACCT-100047","Васильев Максим Романович",16110.31,EUR,2024-11-20,1578810.38
48,"ACCT-100048","Соколова Ольга Викторовна",75036.0,RUB,2025-12-21,75036.00
49,"ACCT-100049","Михайлов Артём Денисович",16744.77,USD,2020-01-22,1507029.30
50,"ACCT-100050","Фёдорова Наталья Петровна",17062.0,EUR,2021-02-23,1672076.00
51,"ACCT-100051","Иванов Иван Иванович",78788.25,RUB,2022-03-24,78788.25
52,"ACCT-100052","Петрова Анна Сергеевна",17696.46,USD,2023-04-25,1592681.40
53,"ACCT-100053","Сидоров Алексей Павлович",18013.69,EUR,2024-05-26,1765341.62
54,"ACCT-100054","Смирнова Мария Олеговна",82540.5,RUB,2025-06-27,82540.50
55,"ACCT-100055","Кузнецов Дмитрий Игоревич",18648.15,USD,2020-07-01,1678333.50
```

### **4.1. Результат обработки CSV-файлов**

В ходе обработки исходного файла программа должна:

- Прочитать записи о банковских счетах из `accounts.csv`.
- Проверить корректность номера счёта, имени владельца, баланса, валюты и даты открытия.
- Исключить записи с отрицательным балансом, пустым номером счёта, неизвестной валютой и некорректным числовым значением.
- Обработать записи с недостаточным количеством полей.
- Исключить дубликат номера счёта.
- Пересчитать баланс каждого корректного счёта в рубли.
- Записать прошедшие проверку данные в `accounts_clean.csv`.

В результате исходный файл содержит 61 запись, а очищенный файл — 55 корректных записей.

## **4.2. Вывод программы**

```
=== Лабораторная работа №7. Вариант 8: Счета ===

Курсы для учебной конвертации: 1 USD = 90.0 RUB; 1 EUR = 98.0 RUB; 1 RUB = 1 RUB.

Строка 57 пропущена: balance должен быть >= 0
Строка 58 пропущена: accountNumber не должен быть пустым
Строка 59 пропущена: допустимы только USD, EUR, RUB
Строка 60 пропущена: id или balance имеет неверный формат
Строка 61 пропущена: ожидалось 6 полей, получено 5
Дубликат номера счёта: ACCT-100001 (строка 62)

1. Чтение через BufferedReader:
=== Статистика обработки ===
Всего строк (включая заголовок): 62
Успешно прочитано: 55
Пропущено с ошибками: 5
Дубликатов по номеру счёта: 1
Типы ошибок:
  - Некорректный баланс: 1
  - Пустой номер счёта: 1
  - Некорректная валюта: 1
  - Ошибка числа: 1
  - Количество полей: 1

Время выполнения: 69 мс

2. Чтение через Files.readAllLines() + Stream API:
=== Статистика обработки ===
Всего строк (включая заголовок): 62
Успешно прочитано: 55
Пропущено с ошибками: 5
Дубликатов по номеру счёта: 1
Типы ошибок:
  - Некорректный баланс: 1
  - Пустой номер счёта: 1
  - Некорректная валюта: 1
  - Ошибка числа: 1
  - Количество полей: 1

Время выполнения: 5 мс

3. Конвертация валют и запись результата:
Общий баланс всех корректных счетов в RUB: 35659971.45 RUB
Сумма по исходным валютам: {EUR=178637.85, USD=191575.86, RUB=911634.75}
Результат сохранён в: src\main\resources\accounts_clean.csv

4. Сравнение подходов:
BufferedReader — построчное чтение, не требует загружать весь файл в память.
Files.readAllLines() + Stream API — компактная обработка коллекции строк, но весь файл сначала загружается в память.
```

## **5. Ответы на контрольные вопросы**

**1. Для чего нужен&#x20;**`BufferedReader`**?**

`BufferedReader` читает текст с буферизацией и предоставляет метод `readLine()` для получения строк. Это удобно при последовательной обработке больших файлов.

**2. Чем&#x20;**`Files.readAllLines()`**&#x20;отличается от построчного чтения?**

`Files.readAllLines()` загружает все строки файла в память и возвращает список. `BufferedReader` позволяет обрабатывать строки по одной, не загружая весь файл целиком.

**3. Для чего применяется Stream API?**

Stream API позволяет выполнять операции над последовательностями данных, например фильтрацию, преобразование и сбор результатов. Это помогает записывать обработку коллекций в виде цепочки операций.

**4. Почему недостаточно использовать&#x20;**`split(",")`**&#x20;для любого CSV-файла?**

Запятая может находиться внутри поля, заключённого в кавычки, и в таком случае не является разделителем. Также CSV допускает удвоенные кавычки внутри поля, поэтому требуется учитывать состояние кавычек при разборе строки.

**5. Зачем нужна конструкция&#x20;**`try-with-resources`**?**

Она автоматически закрывает файлы и другие ресурсы после завершения блока, в том числе при возникновении исключения. Это снижает риск утечек ресурсов.

**6. Для чего проверяются дубликаты номеров счетов?**

Проверка не позволяет добавить один и тот же номер счёта в результат несколько раз. В данной работе повторный номер фиксируется в статистике и пропускается.

**7. Как выполняется конвертация валют?**

Баланс умножается на курс соответствующей валюты к рублю. В работе используются фиксированные учебные курсы: USD — 90 RUB, EUR — 98 RUB, RUB — 1 RUB.

## **6. Вывод**

В ходе лабораторной работы реализована обработка CSV-файла со сведениями о счетах.

Программа читает данные двумя способами, проверяет корректность записей, выявляет дубликаты, формирует статистику, конвертирует балансы в рубли и сохраняет очищенные данные в новый CSV-файл.

Сравнение показало, что `BufferedReader` удобен для построчной обработки, а `Files.readAllLines()` совместно со Stream API — для обработки уже загруженной в память коллекции строк.

В результате были закреплены навыки работы с файловым вводом-выводом, обработки CSV-данных, применения Stream API и обработки исключений в Java.

```
```
