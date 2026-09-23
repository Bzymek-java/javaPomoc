# 📋 Cheat Sheet - Kolokwium Java OOP

> Analiza kolokwiów 2021–2024. Wzorce powtarzają się co roku.

---

## 🏗️ Struktura projektu (Maven)

```
src/main/java/    ← kod główny
src/test/java/    ← testy JUnit
pom.xml           ← zależności
```

**Kompilacja i uruchomienie:**
```bash
mvn compile
mvn test
mvn exec:java -Dexec.mainClass="Main"
```

---

## 🔁 Wzorzec powtarzający się w każdym roku

Każde kolokwium ma:
1. **Klasę abstrakcyjną** z fabryką `fromCsv(Path)`
2. **Dwie podklasy** rozszerzające klasę abstrakcyjną
3. **Własny wyjątek** (checked lub unchecked)
4. **Parsowanie CSV** z danymi
5. **Sortowanie** listy obiektów
6. **Zapis do pliku** (tekstowego lub SVG)
7. **Testy JUnit** (od 2023)

---

## 📦 Klasa abstrakcyjna + podklasy

```java
public abstract class Base {
    private final String name;          // prywatne, final

    protected Base(String name) {       // konstruktor protected
        this.name = name;
    }

    public String getName() { return name; }

    // Statyczna fabryka
    public static Base fromCsv(Path path) throws IOException {
        // ...
    }

    // Metoda abstrakcyjna
    public abstract double getValue(int year, int month);
}

public class ConcreteA extends Base {
    public ConcreteA(String name) {
        super(name);
    }
    @Override
    public double getValue(int year, int month) { /* ... */ }
}

public class ConcreteB extends Base {
    private final ConcreteA[] children;
    public ConcreteB(String name, ConcreteA[] children) {
        super(name);
        this.children = children;
    }
    @Override
    public double getValue(int year, int month) {
        // suma rekurencyjna po dzieciach
        int sum = 0;
        for (var c : children) sum += c.getValue(year, month);
        return sum;
    }
}
```

---

## 📄 Parsowanie CSV

```java
// Wczytaj wszystkie linie
List<String> lines = Files.readAllLines(Path.of("plik.csv"));

// Podziel linię
String[] cols = line.split(";", -1);   // semicolon
String[] cols = line.split(",", -1);   // comma

// Polska liczba (przecinek → kropka)
double v = Double.parseDouble(s.trim().replace(",", "."));

// Pomiń nagłówek
for (int i = 2; i < lines.size(); i++) { /* ... */ }

// Parsowanie daty
LocalDate d = LocalDate.parse(s.trim(),
    DateTimeFormatter.ofPattern("M/d/yy"));   // format US: 1/5/21
LocalDate d = LocalDate.parse(s.trim(),
    DateTimeFormatter.ofPattern("d.MM.yy"));  // format PL: 05.01.21

// Scanner (linia po linii, mniej pamięci)
try (Scanner sc = new Scanner(path)) {
    String name = sc.nextLine();   // pierwsza linia
    sc.nextLine();                 // pomiń nagłówek
    while (sc.hasNextLine()) { String line = sc.nextLine(); }
}
```

---

## ⚠️ Wyjątki

```java
// Checked (kompilator wymusza przechwycenie)
class NotFoundException extends Exception {
    public NotFoundException(String name) { super(name); }
}
// Rzucanie: throw new NotFoundException("Polska");
// Deklaracja: throws NotFoundException

// Unchecked (nie wymusza przechwycenia)
class DomainException extends RuntimeException {
    public DomainException(String msg) { super(msg); }
}

// FileNotFoundException (wbudowany checked)
throw new FileNotFoundException(pathStr);

// IllegalArgumentException (wbudowany unchecked) - 2024
throw new IllegalArgumentException("Godzina poza zakresem: " + hour);

// IndexOutOfBoundsException (wbudowany unchecked) - 2022
throw new IndexOutOfBoundsException("Data poza zakresem");
```

---

## 🔢 Sortowanie

```java
// Malejąco po liczbie
list.sort(Comparator.comparingInt(Obj::getValue).reversed());

// Rosnąco po stringu
list.sort(Comparator.comparing(Obj::getName));

// Własny komparator (metoda statyczna, 2024)
public static int compare(City a, City b) {
    return Double.compare(Math.abs(b.diff()), Math.abs(a.diff()));
}
list.sort(City::compare);

// Przez Stream
List<Obj> sorted = list.stream()
    .sorted(Comparator.comparing(Obj::getName))
    .collect(Collectors.toList());
```

---

## 📁 Zapis do pliku

```java
// Zapis z tabulatorami (2021)
try (PrintWriter pw = new PrintWriter(Files.newBufferedWriter(Path.of("out.txt")))) {
    pw.printf("%s\t%d\t%d%n", date.format(DateTimeFormatter.ofPattern("d.MM.yy")), v1, v2);
}

// Zapis Stringa (SVG, 2024)
Files.writeString(Path.of("plik.svg"), svgContent);

// Stwórz katalog + plik
Files.createDirectories(Path.of("katalog"));
Files.writeString(Path.of("katalog/miasto.svg"), svg);

// Sprawdź istnienie pliku
if (!Files.exists(path) || !Files.isReadable(path))
    throw new FileNotFoundException(path.toString());
```

---

## ⏰ Czas (2024)

```java
LocalTime now = LocalTime.now();
LocalTime t = LocalTime.of(13, 30, 0);
int h = t.getHour(); int m = t.getMinute(); int s = t.getSecond();
LocalTime shifted = t.plusHours(2).minusHours(1);

// Format 24h
t.format(DateTimeFormatter.ofPattern("HH:mm:ss"));

// Format 12h (własna implementacja)
String ampm = h < 12 ? "AM" : "PM";
int h12 = h % 12; if (h12 == 0) h12 = 12;
String result = String.format("%d:%02d:%02d %s", h12, m, s, ampm);

// Local Mean Time: longitude → offset sekund
long offsetSec = Math.round(longitude / 180.0 * 12 * 3600);
LocalTime lmt = utc.plusSeconds(offsetSec);
```

---

## 📐 Geometria (2023)

```java
// Record Point
record Point(double x, double y) {
    double distanceTo(Point o) {
        return Math.sqrt(Math.pow(x-o.x,2) + Math.pow(y-o.y,2));
    }
}

// Algorytm ray casting (punkt w wielokącie)
boolean inside(Point p) {
    int counter = 0;
    int n = points.size();
    for (int i = 0; i < n; i++) {
        Point pa = points.get(i), pb = points.get((i+1)%n);
        if (pa.y() > pb.y()) { Point t=pa; pa=pb; pb=t; }
        if (pa.y() < p.y() && p.y() < pb.y()) {
            double d = pb.x()-pa.x();
            double x = (d==0) ? pa.x() : (p.y() - (pa.y() - (pb.y()-pa.y())/d*pa.x())) / ((pb.y()-pa.y())/d);
            if (x < p.x()) counter++;
        }
    }
    return counter % 2 == 1;
}

// Kwadrat z centrum i bokiem
List<Point> square(Point c, double side) {
    double h = side/2;
    return List.of(new Point(c.x()-h,c.y()-h), new Point(c.x()+h,c.y()-h),
                   new Point(c.x()+h,c.y()+h), new Point(c.x()-h,c.y()+h));
}
```

---

## 🧪 JUnit 5 (od 2023)

```java
// Zależność w pom.xml (już dodana)

import org.junit.jupiter.api.*;
import org.junit.jupiter.params.*;
import org.junit.jupiter.params.provider.*;
import static org.junit.jupiter.api.Assertions.*;

class MyTest {
    @Test
    void simpleTest() {
        assertTrue(polygon.inside(new Point(5,5)));
        assertFalse(polygon.inside(new Point(100,100)));
    }

    @Test
    void exceptionTest() {
        RuntimeException ex = assertThrows(RuntimeException.class, () -> {
            land.addCity(cityOnWater);
        });
        assertEquals("NazwaMiasta", ex.getMessage());
    }

    @ParameterizedTest
    @MethodSource("cases")
    void paramTest(String input, boolean expected) {
        assertEquals(expected, someMethod(input));
    }

    static Stream<Arguments> cases() {
        return Stream.of(
            Arguments.of("A", true),
            Arguments.of("B", false)
        );
    }
}
```

---

## 🧩 Interfejsy funkcyjne (2022)

```java
// Użycie Function<Path, T> jako fabryki
@FunctionalInterface
interface CsvFactory<T> {
    T create(Path path) throws IOException;
}

// Wywołanie metody referencją
loadAll(NonFoodProduct::fromCsv, Path.of("data/nonfood"));
loadAll(FoodProduct::fromCsv,    Path.of("data/food"));

// Wzorzec w metodzie
static <T> List<T> loadAll(CsvFactory<T> factory, Path dir) throws IOException {
    try (var stream = Files.list(dir)) {
        return stream.filter(p -> p.toString().endsWith(".csv"))
                     .map(p -> { try { return factory.create(p); }
                                 catch (IOException e) { throw new RuntimeException(e); }})
                     .collect(Collectors.toList());
    }
}
```

---

## 🔧 SVG (2024)

```java
// Budowanie SVG przez StringBuilder
StringBuilder sb = new StringBuilder();
sb.append("<svg xmlns=\"http://www.w3.org/2000/svg\" width=\"400\" height=\"400\">\n");
sb.append("<circle cx=\"200\" cy=\"200\" r=\"190\" fill=\"white\" stroke=\"black\" stroke-width=\"3\"/>\n");
// Wskazówka (rotate wokół centrum)
sb.append("<line x1=\"200\" y1=\"200\" x2=\"200\" y2=\"50\" stroke=\"black\" stroke-width=\"3\" ");
sb.append("transform=\"rotate(%.2f 200 200)\"/>\n".formatted(angleDeg));
sb.append("</svg>");

// Kąty wskazówek
double secAngle  = second * 6.0;                          // 360/60
double minAngle  = minute * 6.0 + second * 0.1;           // płynny
double hourAngle = (hour % 12) * 30.0 + minute * 0.5 + second / 120.0;

// Zapis
Files.writeString(path, sb.toString());
```

---

## 📌 Enum

```java
// Wewnątrz klasy (2023: Resource.Type, 2024: DigitalClock.Mode)
public class Resource {
    public enum Type { Coal, Wood, Fish }
    public final Type type;
    // ...
}

// Użycie
Resource.Type t = Resource.Type.Coal;
Set<Resource.Type> resources = new HashSet<>();
resources.contains(Resource.Type.Fish);
```

---

## 📊 Wzorzec INFLACJA / koszyk (2022)

```java
// Cart.getInflation (krok 5)
double getInflation(int y1, int m1, int y2, int m2) {
    double price1 = getPrice(y1, m1);
    double price2 = getPrice(y2, m2);
    int months = (y2 - y1) * 12 + (m2 - m1);
    return (price2 - price1) / price1 * 100.0 / months * 12;
}
```

---

## 🗺️ Jackson XML (2023)

```xml
<!-- pom.xml - już dodane -->
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-xml</artifactId>
    <version>2.17.1</version>
</dependency>
```

```java
import com.fasterxml.jackson.annotation.JsonProperty;
import com.fasterxml.jackson.dataformat.xml.XmlMapper;
import com.fasterxml.jackson.dataformat.xml.annotation.JacksonXmlElementWrapper;

// Parsowanie SVG jako XML
XmlMapper mapper = new XmlMapper();
Svg svg = mapper.readValue(new File("map.svg"), Svg.class);

// W klasie Svg:
@JacksonXmlElementWrapper(useWrapping = false)
@JsonProperty("polygon")
private List<Map<String, String>> polygons;

// Atrybuty: "points" → "10,20 30,40 50,60"
// Parsuj: pair.split(",") → x, y
```

---

## 💡 Przydatne one-linery

```java
// Suma tablicy
Arrays.stream(arr).mapToDouble(Double::doubleValue).sum()

// Średnia tablicy
Arrays.stream(arr).mapToDouble(Double::doubleValue).average().orElse(0)

// Lista → tablica
list.toArray(new String[0])

// Tablica → lista
Arrays.asList(arr)    // stała długość
new ArrayList<>(Arrays.asList(arr))  // modyfikowalna

// String.format z zerowaniem
String.format("%02d", 5)  // "05"

// Sprawdź zakres LocalDate
!date.isBefore(from) && !date.isAfter(to)

// Odległość między punktami
Math.sqrt(Math.pow(x2-x1,2) + Math.pow(y2-y1,2))

// Math.abs różnicy czasu
Math.abs(Duration.between(t1, t2).toSeconds())
```
