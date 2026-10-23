# Structural Patterns

Structural patterns deal with **object composition** — how classes and objects are combined to form larger structures while keeping these structures flexible and efficient.

## Table of Contents
- [Adapter](#adapter)
- [Bridge](#bridge)
- [Composite](#composite)
- [Decorator](#decorator)
- [Facade](#facade)
- [Flyweight](#flyweight)
- [Proxy](#proxy)
- [Pattern Comparison](#pattern-comparison)
- [Interview Questions](#interview-questions)
- [⚡ Flashcard Summary](#-flashcard-summary)

---

## Adapter

**Intent:** Allow objects with incompatible interfaces to collaborate. Wraps an existing class with a new interface.

**Real-world analogy:** Power plug adapter — your laptop has a 3-pin plug, the wall has 2-pin socket. Adapter translates.

```java
// Legacy payment system — old interface
class LegacyPaymentProcessor {
    public void processPayment(String accountNumber, double amount) {
        System.out.println("Processing $" + amount + " for account: " + accountNumber);
    }
}

// New payment interface that our system expects
interface PaymentGateway {
    void pay(String userId, double amount, String currency);
}

// Adapter — wraps legacy, exposes new interface
class PaymentAdapter implements PaymentGateway {
    private LegacyPaymentProcessor legacy;
    
    public PaymentAdapter(LegacyPaymentProcessor legacy) {
        this.legacy = legacy;
    }
    
    @Override
    public void pay(String userId, double amount, String currency) {
        // Translate: userId → account number, convert currency if needed
        double convertedAmount = convertCurrency(amount, currency);
        String accountNumber = resolveAccount(userId);
        legacy.processPayment(accountNumber, convertedAmount);
    }
    
    private double convertCurrency(double amount, String currency) {
        // Currency conversion logic
        return "USD".equals(currency) ? amount : amount * 83.0; // INR example
    }
    
    private String resolveAccount(String userId) {
        return "ACC-" + userId; // lookup logic
    }
}

// Usage — code works with PaymentGateway interface; doesn't know about legacy system
PaymentGateway gateway = new PaymentAdapter(new LegacyPaymentProcessor());
gateway.pay("user123", 99.99, "USD");

// Another adapter example: List ↔ Array
// Arrays.asList() — adapts array to List interface (fixed size)
List<String> list = Arrays.asList("a", "b", "c");
```

**Two forms:**
- **Object Adapter** (shown above): Adapter holds instance of adaptee — more flexible (can adapt multiple classes)
- **Class Adapter**: Adapter extends adaptee — only works in languages with multiple inheritance

---

## Bridge

**Intent:** Decouple an abstraction from its implementation so that the two can vary independently.

**Real-world analogy:** A TV remote (abstraction) and a TV (implementation). You can swap the remote (universal remote) or swap the TV — independently.

```java
// Without Bridge: Explosion of subclasses
// Shape + Color → RedCircle, BlueCircle, RedSquare, BlueSquare... (2×2 = 4 classes)
// With Bridge: Shape HAS-A Color → 2 + 2 = 4 classes for same result, but extensible

// Implementation interface (Color)
interface Color {
    String fill();
}

class Red implements Color {
    public String fill() { return "Red color"; }
}
class Blue implements Color {
    public String fill() { return "Blue color"; }
}

// Abstraction (Shape) — holds reference to implementation
abstract class Shape {
    protected Color color; // Bridge — holds implementation
    
    public Shape(Color color) { this.color = color; }
    public abstract void draw();
}

// Refined abstraction
class Circle extends Shape {
    private double radius;
    
    public Circle(double radius, Color color) {
        super(color);
        this.radius = radius;
    }
    
    @Override
    public void draw() {
        System.out.println("Drawing Circle[radius=" + radius + "] with " + color.fill());
    }
}

class Square extends Shape {
    private double side;
    
    public Square(double side, Color color) {
        super(color);
        this.side = side;
    }
    
    @Override
    public void draw() {
        System.out.println("Drawing Square[side=" + side + "] with " + color.fill());
    }
}

// Usage — shapes and colors vary independently
Shape redCircle = new Circle(5.0, new Red());
Shape blueSquare = new Square(4.0, new Blue());
redCircle.draw();  // Drawing Circle[radius=5.0] with Red color
blueSquare.draw(); // Drawing Square[side=4.0] with Blue color
// Add new color (Green): no Shape changes needed
// Add new shape (Triangle): no Color changes needed
```

---

## Composite

**Intent:** Compose objects into **tree structures** to represent part-whole hierarchies. Clients treat individual objects and compositions uniformly.

**Real-world analogy:** File system — files and folders both support `open()`, `delete()`, `getSize()`. A folder's `getSize()` sums its children.

```java
// Component interface — uniform interface for both leaf and composite
interface FileSystemItem {
    String getName();
    long getSize();
    void print(String indent);
}

// Leaf — no children
class File implements FileSystemItem {
    private String name;
    private long size;
    
    public File(String name, long size) {
        this.name = name;
        this.size = size;
    }
    
    public String getName() { return name; }
    public long getSize() { return size; }
    public void print(String indent) {
        System.out.println(indent + "📄 " + name + " (" + size + " bytes)");
    }
}

// Composite — has children, delegates to them
class Folder implements FileSystemItem {
    private String name;
    private List<FileSystemItem> children = new ArrayList<>();
    
    public Folder(String name) { this.name = name; }
    
    public void add(FileSystemItem item) { children.add(item); }
    public void remove(FileSystemItem item) { children.remove(item); }
    
    public String getName() { return name; }
    
    // Composite operation: sum of all children
    public long getSize() {
        return children.stream().mapToLong(FileSystemItem::getSize).sum();
    }
    
    public void print(String indent) {
        System.out.println(indent + "📁 " + name + " (" + getSize() + " bytes)");
        for (FileSystemItem child : children) {
            child.print(indent + "  "); // recurse with indentation
        }
    }
}

// Usage — client code treats File and Folder the same way
Folder root = new Folder("root");
Folder src = new Folder("src");
src.add(new File("Main.java", 2048));
src.add(new File("Utils.java", 1024));
root.add(src);
root.add(new File("README.md", 512));

root.print("");
// 📁 root (3584 bytes)
//   📁 src (3072 bytes)
//     📄 Main.java (2048 bytes)
//     📄 Utils.java (1024 bytes)
//   📄 README.md (512 bytes)
System.out.println("Total: " + root.getSize()); // 3584
```

---

## Decorator

**Intent:** Attach additional responsibilities to an object **dynamically**. Decorators provide a flexible alternative to subclassing for extending functionality.

**Real-world analogy:** Coffee + milk + sugar. Each addition wraps the base coffee with new behavior.

```java
// Component interface
interface CoffeeOrder {
    String getDescription();
    double getCost();
}

// Concrete component
class SimpleCoffee implements CoffeeOrder {
    public String getDescription() { return "Simple Coffee"; }
    public double getCost() { return 1.00; }
}

// Base decorator — implements same interface, wraps a component
abstract class CoffeeDecorator implements CoffeeOrder {
    protected CoffeeOrder wrapped;
    public CoffeeDecorator(CoffeeOrder coffee) { this.wrapped = coffee; }
}

// Concrete decorators
class MilkDecorator extends CoffeeDecorator {
    public MilkDecorator(CoffeeOrder coffee) { super(coffee); }
    public String getDescription() { return wrapped.getDescription() + ", Milk"; }
    public double getCost() { return wrapped.getCost() + 0.25; }
}

class SugarDecorator extends CoffeeDecorator {
    public SugarDecorator(CoffeeOrder coffee) { super(coffee); }
    public String getDescription() { return wrapped.getDescription() + ", Sugar"; }
    public double getCost() { return wrapped.getCost() + 0.10; }
}

class WhipDecorator extends CoffeeDecorator {
    public WhipDecorator(CoffeeOrder coffee) { super(coffee); }
    public String getDescription() { return wrapped.getDescription() + ", Whip"; }
    public double getCost() { return wrapped.getCost() + 0.50; }
}

// Usage — compose at runtime
CoffeeOrder order = new SimpleCoffee();
order = new MilkDecorator(order);
order = new SugarDecorator(order);
order = new WhipDecorator(order);
System.out.println(order.getDescription()); // Simple Coffee, Milk, Sugar, Whip
System.out.println("$" + order.getCost());   // $1.85

// Real Java examples:
// InputStream → FileInputStream → BufferedInputStream → GZIPInputStream
// (each wraps the previous — classic Decorator chain)
InputStream in = new GZIPInputStream(new BufferedInputStream(new FileInputStream("file.gz")));
```

---

## Facade

**Intent:** Provide a **simplified interface** to a complex subsystem. Reduces dependencies on internals.

**Real-world analogy:** Your TV's remote is a facade — one button for "Watch Movie" sets input, dims lights, starts streaming — you don't configure each subsystem separately.

```java
// Complex subsystem classes
class VideoEncoder {
    public void encode(String video, String format) {
        System.out.println("Encoding " + video + " to " + format);
    }
}

class AudioMixer {
    public void mix(String audio) {
        System.out.println("Mixing audio: " + audio);
    }
}

class SubtitleProcessor {
    public void process(String subtitles) {
        System.out.println("Processing subtitles: " + subtitles);
    }
}

class CDNUploader {
    public String upload(String file) {
        System.out.println("Uploading " + file + " to CDN");
        return "https://cdn.example.com/" + file;
    }
}

// Facade — provides one simple method for complex multi-step process
class VideoPublisher {
    private VideoEncoder encoder = new VideoEncoder();
    private AudioMixer mixer = new AudioMixer();
    private SubtitleProcessor subtitleProcessor = new SubtitleProcessor();
    private CDNUploader uploader = new CDNUploader();
    
    // Single simplified interface
    public String publish(String videoFile, String audioFile, String subtitles, String format) {
        encoder.encode(videoFile, format);
        mixer.mix(audioFile);
        subtitleProcessor.process(subtitles);
        String processedFile = videoFile.replace(".raw", "." + format);
        return uploader.upload(processedFile);
    }
}

// Client — only knows VideoPublisher, not the subsystem internals
VideoPublisher publisher = new VideoPublisher();
String url = publisher.publish("movie.raw", "audio.wav", "subs.srt", "mp4");
```

**Facade vs Adapter:**
- **Adapter:** Changes the interface of an existing object (1-to-1 translation)
- **Facade:** Simplifies a complex system into a single interface (many-to-1 simplification)

---

## Flyweight

**Intent:** Use sharing to support a large number of fine-grained objects efficiently. Separates **intrinsic state** (shared) from **extrinsic state** (context-specific).

**Use cases:** Text rendering (character objects), game particles, icons in file system.

```java
// Flyweight — shared intrinsic state
class CharacterGlyph {
    private final char character;    // intrinsic — same for all 'A' characters
    private final String font;       // intrinsic — shared
    private final int size;          // intrinsic — shared
    
    public CharacterGlyph(char character, String font, int size) {
        this.character = character;
        this.font = font;
        this.size = size;
        System.out.println("Creating glyph for '" + character + "'"); // only once per unique char+font+size
    }
    
    // Render uses extrinsic state (position) passed from outside
    public void render(int x, int y, String color) {
        System.out.printf("Rendering '%c' at (%d,%d) in %s font, size %d, color %s%n",
            character, x, y, font, size, color);
    }
}

// Flyweight Factory — returns shared instances
class GlyphFactory {
    private Map<String, CharacterGlyph> cache = new HashMap<>();
    
    public CharacterGlyph getGlyph(char c, String font, int size) {
        String key = c + "_" + font + "_" + size;
        return cache.computeIfAbsent(key, k -> new CharacterGlyph(c, font, size));
    }
}

// Context — stores extrinsic state (position, color) per character instance
class TextCharacter {
    private CharacterGlyph glyph; // shared flyweight
    private int x, y;              // extrinsic — unique per position
    private String color;          // extrinsic
    
    public TextCharacter(CharacterGlyph glyph, int x, int y, String color) {
        this.glyph = glyph; this.x = x; this.y = y; this.color = color;
    }
    
    public void render() { glyph.render(x, y, color); }
}

// Usage — 1000 'A' characters share one CharacterGlyph object
GlyphFactory factory = new GlyphFactory();
List<TextCharacter> text = new ArrayList<>();

for (int i = 0; i < 1000; i++) {
    CharacterGlyph glyph = factory.getGlyph('A', "Arial", 12); // reused
    text.add(new TextCharacter(glyph, i * 10, 100, "black"));   // 1000 positions
}
// "Creating glyph for 'A'" printed ONCE, not 1000 times
// Memory: 1 CharacterGlyph + 1000 lightweight TextCharacter objects
```

**Java's own Flyweight examples:**
```java
Integer.valueOf(127)  // -128 to 127 cached — same object returned
String.intern()       // string pool — shared string instances
```

---

## Proxy

**Intent:** Provide a substitute or placeholder for another object. Controls access to the original.

**Types:**
- **Virtual Proxy:** Lazy initialization (defer creation until needed)
- **Protection Proxy:** Access control
- **Remote Proxy:** Represents object in another JVM/server
- **Caching Proxy:** Caches results of expensive operations

```java
interface DataService {
    String fetchData(String query);
}

// Real subject — expensive operation
class DatabaseDataService implements DataService {
    public String fetchData(String query) {
        System.out.println("Executing expensive DB query: " + query);
        // Simulate DB call
        return "Result for: " + query;
    }
}

// Caching Proxy
class CachedDataService implements DataService {
    private DataService realService;
    private Map<String, String> cache = new HashMap<>();
    
    public CachedDataService(DataService realService) {
        this.realService = realService;
    }
    
    @Override
    public String fetchData(String query) {
        if (cache.containsKey(query)) {
            System.out.println("Cache HIT: " + query);
            return cache.get(query);
        }
        System.out.println("Cache MISS: " + query);
        String result = realService.fetchData(query);
        cache.put(query, result);
        return result;
    }
}

// Protection Proxy — access control
class SecureDataService implements DataService {
    private DataService realService;
    private String currentRole;
    
    public SecureDataService(DataService realService, String role) {
        this.realService = realService;
        this.currentRole = role;
    }
    
    @Override
    public String fetchData(String query) {
        if (!"ADMIN".equals(currentRole) && query.contains("sensitive")) {
            throw new SecurityException("Access denied for role: " + currentRole);
        }
        return realService.fetchData(query);
    }
}

// Usage
DataService service = new CachedDataService(new DatabaseDataService());
service.fetchData("SELECT * FROM users"); // Cache MISS → DB call
service.fetchData("SELECT * FROM users"); // Cache HIT → no DB call

// Real Java Proxy example: JDK Dynamic Proxy (used by Spring AOP)
DataService dynamicProxy = (DataService) Proxy.newProxyInstance(
    DataService.class.getClassLoader(),
    new Class[]{DataService.class},
    (proxy, method, args) -> {
        System.out.println("Before: " + method.getName());
        Object result = method.invoke(new DatabaseDataService(), args);
        System.out.println("After: " + method.getName());
        return result;
    }
);
```

---

## Pattern Comparison

| Pattern | Wraps? | Changes interface? | Adds behavior? | Use case |
|---------|--------|--------------------|----------------|----------|
| Adapter | Yes | Yes | No | Interface compatibility |
| Bridge | No | No | No | Vary independently |
| Composite | Children | No | No | Tree structure |
| Decorator | Yes | No | Yes | Add features dynamically |
| Facade | Yes (subsystem) | Yes (simplifies) | No | Simplify complex API |
| Flyweight | Partial | No | No | Memory efficiency |
| Proxy | Yes | No | Access control/cache | Control access |

---

## Interview Questions

### Q: Decorator vs Proxy — what's the difference?

Both wrap another object. The key difference is **intent**:
- **Decorator**: Adds new behavior/responsibility. Client knows they're using a decorator.
- **Proxy**: Controls access to the original. Client often doesn't know about the proxy (transparency).

Also:
- Decorator is typically created by the client (stacked at runtime)
- Proxy often controls its own subject's lifecycle (creates it internally for lazy init)

### Q: Java I/O is a classic example of which pattern?

**Decorator.**
```java
InputStream in = new FileInputStream("data.txt");  // concrete component
in = new BufferedInputStream(in);                  // decorator: adds buffering
in = new GZIPInputStream(in);                      // decorator: adds decompression
```
Each layer wraps the previous with new behavior. You can combine any number of decorators in any order.

---

## ⚡ Flashcard Summary

| # | Pattern | One-liner |
|---|---------|-----------|
| 1 | Adapter | Translates one interface to another |
| 2 | Object Adapter | Composition (holds adaptee) |
| 3 | Bridge | Abstraction HAS-A Implementation; both vary independently |
| 4 | Bridge vs inheritance | Bridge: 2+2=4 classes; Inheritance: 2×2=4 classes |
| 5 | Composite | Tree: leaf and composite both implement Component |
| 6 | Composite key | Composite delegates to children; client code uniform |
| 7 | Decorator | Same interface; wraps and extends behavior |
| 8 | Java I/O + Decorator | FileInputStream → BufferedInputStream → GZIPInputStream |
| 9 | Facade | Hides complexity behind simplified interface |
| 10 | Facade vs Adapter | Facade: many-to-1 simplification; Adapter: 1-to-1 translation |
| 11 | Flyweight | Intrinsic (shared) vs Extrinsic (context) state |
| 12 | Flyweight Java example | Integer.valueOf() caches -128 to 127 |
| 13 | Proxy types | Virtual (lazy), Protection (auth), Remote, Caching |
| 14 | Proxy vs Decorator | Proxy: controls access; Decorator: adds behavior |
| 15 | Spring AOP | Uses JDK Dynamic Proxy or CGLIB for method interception |

---

*Next: [Behavioral Patterns](03-behavioral-patterns.md)*
