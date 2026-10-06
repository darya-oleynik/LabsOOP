# Вариант 8. Result<T, E> — результат операции (успех или ошибка)

## Описание

Обобщённый класс `Result<T, E>` представляет результат операции, которая может
завершиться **успехом** (`Ok`) или **ошибкой** (`Err`).

- `T` — тип успешного значения.
- `E` — тип ошибки.

Класс гарантирует, что объект всегда находится **ровно в одном** из двух состояний:
либо есть значение, либо есть ошибка — но не оба одновременно.

---

## Структура проекта

| Файл | Назначение |
|---|---|
| `Result.java` | Базовый класс `Result<T, E>` |
| `ComparableResult.java` | Наследник с ограничением `Comparable` и поддержкой сортировки |
| `ArrayUtils.java` | Обобщённый метод `swap` для массива любого типа |
| `ResultRepository.java` | Хранилище результатов: `save` / `findById` / `delete` / `findAll` |
| `Main.java` | Демонстрация работы всех классов |

---

## 1. Result.java

```java
public class Result<T, E> {

    private final T value;      // успешное значение
    private final E error;      // ошибка
    private final boolean success;

    protected Result(T value, E error, boolean success) {
        this.value = value;
        this.error = error;
        this.success = success;
    }

    public static <T, E> Result<T, E> ok(T value) {
        return new Result<>(value, null, true);
    }

    public static <T, E> Result<T, E> err(E error) {
        return new Result<>(null, error, false);
    }

    public boolean isSuccess() {
        return success;
    }

    public boolean isError() {
        return !success;
    }

    public T getValue() {
        return value;
    }

    public E getError() {
        return error;
    }

    @Override
    public String toString() {
        return success ? "Ok(" + value + ")" : "Err(" + error + ")";
    }
}
```

---

## 2. ComparableResult.java

```java
public class ComparableResult<T extends Comparable<T>, E extends Comparable<E>>
        extends Result<T, E>
        implements Comparable<ComparableResult<T, E>> {

    private ComparableResult(T value, E error, boolean success) {
        super(value, error, success);
    }

    public static <T extends Comparable<T>, E extends Comparable<E>> ComparableResult<T, E> ok(T value) {
        return new ComparableResult<>(value, null, true);
    }

    public static <T extends Comparable<T>, E extends Comparable<E>> ComparableResult<T, E> err(E error) {
        return new ComparableResult<>(null, error, false);
    }

    @Override
    public int compareTo(ComparableResult<T, E> other) {
        // успех < ошибка
        if (this.isSuccess() && other.isError()) return -1;
        if (this.isError() && other.isSuccess()) return  1;

        // оба успешные — сравниваем значения
        if (this.isSuccess()) {
            return this.getValue().compareTo(other.getValue());
        }

        // оба ошибочные — сравниваем ошибки
        return this.getError().compareTo(other.getError());
    }
}
```

---

## 3. ArrayUtils.java

```java
public class ArrayUtils {

    public static <A> void swap(A[] array, int i, int j) {
        A temp = array[i];
        array[i] = array[j];
        array[j] = temp;
    }
}
```

---

## 4. ResultRepository.java

```java
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class ResultRepository<T, E> {

    private final Map<Long, Result<T, E>> storage = new HashMap<>();
    private long nextId = 0;

    public long save(Result<T, E> result) {
        nextId++;
        storage.put(nextId, result);
        return nextId;
    }

    public Result<T, E> findById(long id) {
        return storage.get(id);
    }

    public boolean delete(long id) {
        return storage.remove(id) != null;
    }

    public List<Result<T, E>> findAll() {
        return new ArrayList<>(storage.values());
    }
}
```

---

## 5. Main.java

```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.Collections;
import java.util.List;

public class Main {
    public static void main(String[] args) {

        System.out.println("=== 1. Result: успех и ошибка ===");

        Result<Integer, String> ok  = Result.ok(42);
        Result<Integer, String> err = Result.err("not found");

        System.out.println(ok);
        System.out.println(err);
        System.out.println(ok.getValue());
        System.out.println(err.getError());

        System.out.println("\n=== 2. ArrayUtils.swap для Result<Integer, String> ===");

        Result<Integer, String>[] results = new Result[] {
                Result.ok(1),
                Result.err("boom"),
                Result.ok(3)
        };

        System.out.println("До:    " + Arrays.toString(results));

        ArrayUtils.swap(results, 0, 2);
        System.out.println("После: " + Arrays.toString(results));

        System.out.println("\n=== 3. ComparableResult: сортировка / min / max ===");

        List<ComparableResult<Integer, String>> list = new ArrayList<>(List.of(
                ComparableResult.ok(10),
                ComparableResult.err("boom"),
                ComparableResult.ok(3),
                ComparableResult.err("aaa")
        ));

        System.out.println("До сортировки: " + list);
        Collections.sort(list);
        System.out.println("После сортировки: " + list);

        System.out.println("\n=== 4. ResultRepository ===");

        ResultRepository<Integer, String> repo = new ResultRepository<>();

        long id1 = repo.save(Result.ok(100));
        long id2 = repo.save(Result.err("failure"));
        long id3 = repo.save(Result.ok(200));

        System.out.println("Сохранено: id1=" + id1 + ", id2=" + id2 + ", id3=" + id3);

        System.out.println("findById(id1) = " + repo.findById(id1));
        System.out.println("findById(id2) = " + repo.findById(id2));
        System.out.println("findById(999) = " + repo.findById(999));

        System.out.println("findAll = " + repo.findAll());

        System.out.println("delete(id1) = " + repo.delete(id1));
        System.out.println("delete(id1) = " + repo.delete(id1));

        System.out.println("findAll после удаления = " + repo.findAll());
    }
}
```

---

## Вывод программы

```
=== 1. Result: успех и ошибка ===
Ok(42)
Err(not found)
42
not found

=== 2. ArrayUtils.swap для Result<Integer, String> ===
До:    [Ok(1), Err(boom), Ok(3)]
После: [Ok(3), Err(boom), Ok(1)]

=== 3. ComparableResult: сортировка / min / max ===
До сортировки: [Ok(10), Err(boom), Ok(3), Err(aaa)]
После сортировки: [Ok(3), Ok(10), Err(aaa), Err(boom)]

=== 4. ResultRepository ===
Сохранено: id1=1, id2=2, id3=3
findById(id1) = Ok(100)
findById(id2) = Err(failure)
findById(999) = null
findAll = [Ok(100), Err(failure), Ok(200)]
delete(id1) = true
delete(id1) = false
findAll после удаления = [Err(failure), Ok(200)]
```

> Порядок `findAll` может отличаться: `HashMap` не гарантирует порядок.

---

## Ответы на вопросы

### 1. Что такое generics? Какую проблему они решают?

Generics — механизм параметризации типов. Решают проблему отсутствия
типобезопасности: ошибки типов ловятся на этапе компиляции, а не в runtime,
и не нужно явно приводить типы.

### 2. Что такое параметр типа? Какие соглашения об именовании существуют?

Параметр типа — это заполнитель для конкретного типа, объявляется в угловых
скобках. Соглашения об именовании: `T` (type), `E` (element), `K` (key),
`V` (value), `N` (number), `S`, `U`, `V` — дополнительные типы,
`R` — возвращаемый тип.

### 3. Что такое ограничители типов (extends)?

Ограничители типов (`extends`) сужают множество допустимых типов для параметра.
Записываются через `extends` (даже для интерфейсов), можно указать несколько
границ через `&`. Есть также wildcards: `? extends T`, `? super T`, `?`.

### 4. Что такое стирание типов (type erasure)?

Стирание типов (type erasure) — удаление информации о generic-типах
компилятором после проверки. В байт-коде `T` заменяется на верхнюю границу
или `Object`, при необходимости вставляются касты. Из-за этого generics
существуют только на этапе компиляции.

### 5. Можно ли создать экземпляр параметра типа (`new T()`)?

Нет, создать экземпляр параметра типа через `new T()` нельзя из-за стирания
типов. Обходные пути: передать `Class<T>` и использовать рефлексию, передать
`Supplier<T>` или фабрику, передать прототип для клонирования.
