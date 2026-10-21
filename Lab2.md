# Лабораторная работа №2. Конструкторы, перегрузка, static

## 1. Задание

Каждый вариант основан на классе из лабораторной работы №1. Необходимо:

- Добавить статическое поле-счётчик и метод `getCounter()`.
- Реализовать не менее 3 конструкторов с вызовом через `this(...)`.
- Реализовать статический фабричный метод.
- Реализовать не менее 2 перегруженных методов.

---

## 2. Класс Film

Класс `Film` описывает фильм и содержит:

- поля: `id`, `title`, `director`, `year`, `rating`;
- статический счётчик `counter` для подсчёта созданных объектов;
- приватный мастер-конструктор и три публичных конструктора, делегирующих ему через `this(...)`;
- статический фабричный метод `createFilm(...)`;
- перегруженные методы `getDescription(...)` и `calculateAge(...)`.

---

## 3. Код Film.java

```java
public class Film {
    private Long id;
    private String title;
    private String director;
    private Integer year;
    private Double rating;

    private static Long counter = 0L;


    private Film(Long id, String title, String director, Integer year, Double rating) {
        this.id = id;
        setTitle(title);
        setDirector(director);
        setYear(year);
        setRating(rating);
    }

    public Film() {
        this("Без названия", "Неизвестен", 1895, 0.0);
    }

    public Film(String title, String director) {
        this(title, director, 1895, 0.0);
    }

    public Film(String title, String director, Integer year, Double rating) {
        this(++counter, title, director, year, rating);
    }


    public static Film createFilm(String title, String director, Integer year, Double rating) {
        return new Film(title, director, year, rating);
    }

    public String getTitle() {
        return title;
    }

    public String getDirector() {
        return director;
    }

    public Long getId() {
        return id;
    }

    public Integer getYear() {
        return year;
    }

    public Double getRating() {
        return rating;
    }

    public static Long getCounter() {
        return counter;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public void setTitle(String title) {
        if (title == null || title.isBlank()) {
            throw new IllegalArgumentException("Название не может быть пустым");
        }
        this.title = title;
    }

    public void setDirector(String director) {
        if (director == null || director.isBlank()) {
            throw new IllegalArgumentException("Поле 'режиссер' не может быть пустым");
        }
        this.director = director;
    }

    public void setYear(Integer year) {
        if (year < 1895 || year > java.time.Year.now().getValue()) {
            throw new IllegalArgumentException("Год должен быть от 1895 до текущего");
        }
        this.year = year;
    }

    public void setRating(Double rating) {
        if (rating < 0.0 || rating > 10.0) {
            throw new IllegalArgumentException("Рейтинг должен быть от 0.0 до 10.0");
        }
        this.rating = rating;
    }

    public String getDescription() {
        return "\"" + title + "\" (реж. " + director + ", " + year + ") — рейтинг " + rating;
    }

    public String getDescription(boolean shortFormat) {
        if (shortFormat) {
            return title + " (" + rating + ")";
        }
        return getDescription();
    }

    public int calculateAge() {
        return java.time.Year.now().getValue() - this.year;
    }

    public int calculateAge(int targetYear) {
        if (targetYear < this.year) {
            throw new IllegalArgumentException("Целевой год (" + targetYear + ") не может быть раньше года выпуска фильма (" + this.year + ")");
        }
        return targetYear - this.year;
    }
}
```

---

## 4. Код Main

```java
public class Main {
    public static void main(String[] args) {

        System.out.println("=== Создание фильмов разными способами ===\n");

        Film film1 = new Film();
        System.out.println("1. По умолчанию:      " + film1.getDescription());

        Film film2 = new Film("Дэдпул и Росомаха", "Шон Леви");
        System.out.println("2. Только название:   " + film2.getDescription());

        Film film3 = new Film("Мстители: Финал", "Энтони и Джо Руссо", 2019, 8.4);
        System.out.println("3. Полный конструктор:" + film3.getDescription());

        Film film4 = Film.createFilm("Одиссея", "Кристофер Нолан", 2026, 8.4);
        System.out.println("4. Фабричный метод:   " + film4.getDescription());

        System.out.println("\n=== Перегруженный getDescription ===\n");
        System.out.println("Полное описание:  " + film3.getDescription());
        System.out.println("Краткое описание: " + film3.getDescription(true));
        System.out.println("Краткое (film4):  " + film4.getDescription(true));

        System.out.println("\n=== Перегруженный calculateAge ===\n");
        System.out.println("Возраст film3 сейчас:                 " + film3.calculateAge() + " лет");
        System.out.println("Возраст film3 на 2030 год:            " + film3.calculateAge(2030) + " лет");
        System.out.println("Возраст film4 на 2030 год:            " + film4.calculateAge(2030) + " лет");

        System.out.println("\n=== Статистика ===\n");
        System.out.println("Всего создано фильмов: " + Film.getCounter());
    }
}
```

---

## 5. Вывод программы

```
=== Создание фильмов разными способами ===

1. По умолчанию:      "Без названия" (реж. Неизвестен, 1895) — рейтинг 0.0
2. Только название:   "Дэдпул и Росомаха" (реж. Шон Леви, 1895) — рейтинг 0.0
3. Полный конструктор:"Мстители: Финал" (реж. Энтони и Джо Руссо, 2019) — рейтинг 8.4
4. Фабричный метод:   "Одиссея" (реж. Кристофер Нолан, 2026) — рейтинг 8.4

=== Перегруженный getDescription ===

Полное описание:  "Мстители: Финал" (реж. Энтони и Джо Руссо, 2019) — рейтинг 8.4
Краткое описание: Мстители: Финал (8.4)
Краткое (film4):  Одиссея (8.4)

=== Перегруженный calculateAge ===

Возраст film3 сейчас:                 7 лет
Возраст film3 на 2030 год:            11 лет
Возраст film4 на 2030 год:            4 лет

=== Статистика ===

Всего создано фильмов: 4
```

---

## 6. Ответы на контрольные вопросы

**1. Перегрузка конструкторов** — это несколько конструкторов в одном классе с одинаковым именем, но разными параметрами. Компилятор сам выбирает подходящий по аргументам.

**2. Вызов конструктора из конструктора** — через `this(...)`, причём первой инструкцией в теле. Так все конструкторы делегируют инициализацию одному «мастер-конструктору».

**3. Статическое поле** принадлежит классу, а не объекту, и существует в единственном экземпляре. Нестатическое — своё у каждого объекта.

**4. Статический метод** принадлежит классу и вызывается через его имя. К нестатическим полям он обращаться не может, потому что не привязан к конкретному объекту.

**5. Фабричный метод** — статический метод, создающий и возвращающий объект. Его преимущества: осмысленное имя, возможность вернуть подкласс или закэшированный объект.

**6. Перегрузка** — одинаковое имя, но разные параметры в одном классе. **Переопределение** — изменение реализации метода родителя в подклассе с той же сигнатурой.

**7. К статическому методу** обращаются через имя класса: `Film.getCounter()`. Через объект тоже можно, но это плохой стиль.

**8. `this`** — ссылка на текущий объект: различает поля и параметры, вызывает другой конструктор, передаёт и возвращает сам объект. В статическом контексте недоступно.
