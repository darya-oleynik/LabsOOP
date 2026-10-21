# Лабораторная работа №4

## Задание

1. Создать `List` из 10 объектов, выполнить 3 вида сортировки.
2. Создать `Set`, переопределить `equals()` и `hashCode()`.
3. Создать `Map` для группировки по одному из полей.
4. Применить Stream API для фильтрации и агрегации.
5. Замерить производительность `ArrayList` vs `HashSet`.

## Вариант 8. Класс «Фильм»

**Поля:** `id`, `title`, `director`, `year`, `rating`.

**Сортировка:** по названию, по году, по рейтингу.

**Группировка:** по режиссёру.

**Фильтрация:** фильмы после 2000 года.

**Агрегация:** средний рейтинг по режиссёрам.

---

## Код

### Movie.java

```java
import java.util.Objects;

public class Movie {
    private long id;
    private String title;
    private String director;
    private int year;
    private double rating;

    public Movie(long id, String title, String director, int year, double rating) {
        this.id = id;
        this.title = title;
        this.director = director;
        this.year = year;
        this.rating = rating;
    }

    public long getId() {
        return id;
    }

    public String getTitle() {
        return title;
    }

    public String getDirector() {
        return director;
    }

    public int getYear() {
        return year;
    }

    public double getRating() {
        return rating;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Movie movie = (Movie) o;
        return year == movie.year &&
                Double.compare(rating, movie.rating) == 0 &&
                Objects.equals(title, movie.title) &&
                Objects.equals(director, movie.director);
    }

    @Override
    public int hashCode() {
        return Objects.hash(title, director, year, rating);
    }

    @Override
    public String toString() {
        return id + ": " +
                "title='" + title +
                ", director='" + director +
                ", year=" + year +
                ", rating=" + rating;
    }
}
