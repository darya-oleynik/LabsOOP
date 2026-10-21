# Вариант 8. Иерархия «Мебель»

## Задание

Каждый вариант содержит:

- базовый класс с 3 полями и 1 методом;
- 2 подкласса с переопределением метода;
- интерфейс с 1 методом;
- реализацию интерфейса в подклассах.

**Вариант 8. Иерархия «Мебель»**

Базовый класс `Furniture`: `name`, `material`, `price`. Метод `assemble()`.  
Подкласс `Chair`: `legsCount`. Переопределение `assemble()`.  
Подкласс `Table`: `surfaceArea`. Переопределение `assemble()`.

---

## Код

### Main.java

```java
public class Main {
    public static void main(String[] args) {
        Furniture chair = new Chair("Офисный стул", "дерево", 8500, 4);
        Furniture table = new Table("Обеденный стол", "дуб", 25000, 2.5);

        System.out.println(chair);
        chair.assemble();

        System.out.println();

        System.out.println(table);
        table.assemble();
    }
}
```

### Furniture.java

```java
public abstract class Furniture {
    protected String name;
    protected String material;
    protected double price;

    public Furniture(String name, String material, double price) {
        this.name = name;
        this.material = material;
        this.price = price;
    }

    public abstract void assemble();

    @Override
    public String toString() {
        return "Furniture{" + "name='" + name + '\'' + ", material='" +
                material + '\'' + ", price=" + price + '}';
    }
}
```

### Deliverable.java

```java
public interface Deliverable {
    int deliveryDays();
}
```

### Chair.java

```java
public class Chair extends Furniture implements Deliverable {
    private Integer legsCount;

    public Chair(String name, String material, double price, Integer legsCount) {
        super(name, material, price);
        this.legsCount = legsCount;
    }

    @Override
    public void assemble() {
        System.out.println("Собираем стул: " + name + ". Прикручиваем " + legsCount + " ножек.");
    }

    @Override
    public int deliveryDays() {
        return 3;
    }

    @Override
    public String toString() {
        return "Chair{" + "legsCount=" + legsCount + "} " + super.toString();
    }
}
```

### Table.java

```java
public class Table extends Furniture implements Deliverable {
    private double surfaceArea;

    public Table(String name, String material, double price, double surfaceArea) {
        super(name, material, price);
        this.surfaceArea = surfaceArea;
    }

    @Override
    public void assemble() {
        System.out.println("Собираем стол: " + name + ". Площадь столешницы: " + surfaceArea + " м².");
    }

    @Override
    public int deliveryDays() {
        return 7;
    }

    @Override
    public String toString() {
        return "Table{" + "surfaceArea=" + surfaceArea + "} " + super.toString();
    }
}
```

---

## Вывод программы

```
Chair{legsCount=4} Furniture{name='Офисный стул', material='дерево', price=8500.0}
Собираем стул: Офисный стул. Прикручиваем 4 ножек.

Table{surfaceArea=2.5} Furniture{name='Обеденный стол', material='дуб', price=25000.0}
Собираем стол: Обеденный стол. Площадь столешницы: 2.5 м².
```

---

## Ответы на вопросы

**1. Что такое наследование? Какое отношение оно моделирует?**
Механизм, при котором один класс (потомок) получает поля и методы другого (родителя). Моделирует отношение «является» (is-a): стул — это мебель.

**2. Что такое переопределение методов? Чем отличается от перегрузки?**
Переопределение — замена метода родителя методом с той же сигнатурой в потомке. Перегрузка — несколько методов с одинаковым именем, но разными параметрами в одном классе.

**3. Что такое полиморфизм? Приведите пример.**
Способность объекта вести себя по-разному в зависимости от реального типа. Пример: переменная типа «Мебель» может ссылаться и на стул, и на стол, а при вызове метода «собрать» выполнится версия конкретного объекта.

**4. Что такое абстрактный класс? Можно ли создать его объект?**
Класс с ключевым словом `abstract`, может содержать абстрактные методы без реализации. Объект создать нельзя — только наследовать.

**5. Что такое интерфейс? Чем отличается от абстрактного класса?**
Контракт: набор методов без реализации (в основном). Отличия: класс реализует много интерфейсов, но наследует один класс; интерфейс не хранит состояние (полей экземпляра нет).

**6. Может ли класс наследовать несколько классов? Реализовывать несколько интерфейсов?**
Наследовать — нет (только один класс). Реализовывать — да, сколько угодно.

**7. Для чего нужна аннотация @Override?**
Компилятор проверяет, что метод действительно переопределяет/реализует метод родителя или интерфейса. Защита от опечаток.

**8. Что такое ключевое слово super?**
Ссылка на родительский класс: вызов его конструктора или его метода.

**9. Когда абстрактный класс, а когда интерфейс?**
Абстрактный класс — когда есть общее состояние (поля) и общая логика. Интерфейс — когда нужен только контракт и множественная реализация.

**10. Что такое модификатор final? Какие ограничения он накладывает?**
`final` запрещает изменение:
- переменная — нельзя переприсвоить;
- метод — нельзя переопределить;
- класс — нельзя наследовать.
