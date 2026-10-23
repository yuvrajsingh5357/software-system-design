# Behavioral Patterns

Behavioral patterns deal with **algorithms and responsibility assignment** between objects — how objects communicate, interact, and divide work.

## Table of Contents
- [Observer](#observer)
- [Strategy](#strategy)
- [Command](#command)
- [State](#state)
- [Template Method](#template-method)
- [Chain of Responsibility](#chain-of-responsibility)
- [Iterator](#iterator)
- [Mediator](#mediator)
- [Memento](#memento)
- [Visitor](#visitor)
- [Interview Questions](#interview-questions)
- [⚡ Flashcard Summary](#-flashcard-summary)

---

## Observer

**Intent:** Define a one-to-many dependency. When an object (Subject) changes state, all dependents (Observers) are notified automatically.

**Use cases:** Event systems, MVC (Model notifies Views), pub/sub systems, stock price feeds.

```java
// Observer interface
interface StockObserver {
    void update(String stockSymbol, double price);
}

// Subject
class StockMarket {
    private Map<String, Double> prices = new HashMap<>();
    private List<StockObserver> observers = new ArrayList<>();
    
    public void addObserver(StockObserver observer) { observers.add(observer); }
    public void removeObserver(StockObserver observer) { observers.remove(observer); }
    
    public void updatePrice(String symbol, double price) {
        prices.put(symbol, price);
        notifyObservers(symbol, price);
    }
    
    private void notifyObservers(String symbol, double price) {
        for (StockObserver observer : observers) {
            observer.update(symbol, price);
        }
    }
}

// Concrete observers
class StockAlertService implements StockObserver {
    private double threshold;
    public StockAlertService(double threshold) { this.threshold = threshold; }
    
    @Override
    public void update(String symbol, double price) {
        if (price < threshold) {
            System.out.println("ALERT: " + symbol + " dropped to $" + price);
        }
    }
}

class PortfolioTracker implements StockObserver {
    private Map<String, Integer> holdings = new HashMap<>();
    public void hold(String symbol, int qty) { holdings.put(symbol, qty); }
    
    @Override
    public void update(String symbol, double price) {
        if (holdings.containsKey(symbol)) {
            double value = holdings.get(symbol) * price;
            System.out.printf("Portfolio: %s × %d @ $%.2f = $%.2f%n",
                symbol, holdings.get(symbol), price, value);
        }
    }
}

// Usage
StockMarket market = new StockMarket();
market.addObserver(new StockAlertService(100.0));
PortfolioTracker tracker = new PortfolioTracker();
tracker.hold("AAPL", 10);
market.addObserver(tracker);

market.updatePrice("AAPL", 95.0); // Both observers notified

// Java built-in alternatives:
// java.util.Observable (deprecated in Java 9)
// PropertyChangeListener from java.beans
// EventListener pattern
// Modern: RxJava, Project Reactor (reactive streams)
```

---

## Strategy

**Intent:** Define a family of algorithms, encapsulate each, and make them interchangeable. Strategy lets the algorithm vary independently from clients that use it.

**Use cases:** Sorting algorithms, pricing strategies, routing algorithms, compression algorithms.

```java
// Strategy interface
interface SortStrategy {
    void sort(int[] data);
}

// Concrete strategies
class QuickSortStrategy implements SortStrategy {
    public void sort(int[] data) {
        System.out.println("QuickSort on " + data.length + " elements");
        Arrays.sort(data); // simplified
    }
}

class MergeSortStrategy implements SortStrategy {
    public void sort(int[] data) {
        System.out.println("MergeSort on " + data.length + " elements");
        // merge sort implementation
    }
}

class BucketSortStrategy implements SortStrategy {
    public void sort(int[] data) {
        System.out.println("BucketSort — best for floats [0,1)");
        // bucket sort implementation
    }
}

// Context
class DataSorter {
    private SortStrategy strategy;
    
    public DataSorter(SortStrategy strategy) { this.strategy = strategy; }
    
    // Strategy can be swapped at runtime
    public void setStrategy(SortStrategy strategy) { this.strategy = strategy; }
    
    public void sort(int[] data) {
        strategy.sort(data);
    }
}

// Pricing strategy example (more realistic)
interface PricingStrategy {
    double calculatePrice(double basePrice, int quantity);
}

class RegularPricing implements PricingStrategy {
    public double calculatePrice(double basePrice, int quantity) {
        return basePrice * quantity;
    }
}

class BulkPricing implements PricingStrategy {
    public double calculatePrice(double basePrice, int quantity) {
        if (quantity > 100) return basePrice * quantity * 0.8; // 20% discount
        return basePrice * quantity;
    }
}

class PremiumMemberPricing implements PricingStrategy {
    public double calculatePrice(double basePrice, int quantity) {
        return basePrice * quantity * 0.9; // 10% member discount
    }
}

// Usage
DataSorter sorter = new DataSorter(new QuickSortStrategy());
sorter.sort(largeDataset);

sorter.setStrategy(new MergeSortStrategy()); // swap at runtime
sorter.sort(almostSortedData);
```

---

## Command

**Intent:** Encapsulate a request as an object. Enables parameterization of clients with different requests, queuing, logging, and undoable operations.

**Use cases:** Undo/redo, transaction management, job queues, macro recording.

```java
// Command interface
interface Command {
    void execute();
    void undo(); // for undo support
}

// Receiver — the actual business logic
class TextEditor {
    private StringBuilder text = new StringBuilder();
    
    public void insert(int position, String content) {
        text.insert(position, content);
        System.out.println("Text: " + text);
    }
    
    public void delete(int position, int length) {
        text.delete(position, position + length);
        System.out.println("Text: " + text);
    }
    
    public String getText() { return text.toString(); }
}

// Concrete commands
class InsertCommand implements Command {
    private TextEditor editor;
    private int position;
    private String content;
    
    public InsertCommand(TextEditor editor, int position, String content) {
        this.editor = editor;
        this.position = position;
        this.content = content;
    }
    
    public void execute() { editor.insert(position, content); }
    public void undo() { editor.delete(position, content.length()); }
}

class DeleteCommand implements Command {
    private TextEditor editor;
    private int position, length;
    private String deletedText; // saved for undo
    
    public DeleteCommand(TextEditor editor, int position, int length) {
        this.editor = editor;
        this.position = position;
        this.length = length;
    }
    
    public void execute() {
        deletedText = editor.getText().substring(position, position + length);
        editor.delete(position, length);
    }
    
    public void undo() { editor.insert(position, deletedText); }
}

// Invoker — manages command history
class CommandHistory {
    private Deque<Command> history = new ArrayDeque<>();
    
    public void execute(Command command) {
        command.execute();
        history.push(command);
    }
    
    public void undo() {
        if (!history.isEmpty()) {
            history.pop().undo();
        }
    }
}

// Usage
TextEditor editor = new TextEditor();
CommandHistory history = new CommandHistory();

history.execute(new InsertCommand(editor, 0, "Hello"));
history.execute(new InsertCommand(editor, 5, " World"));
// Text: Hello World

history.undo(); // undo " World"
// Text: Hello

history.undo(); // undo "Hello"
// Text: (empty)
```

---

## State

**Intent:** Allow an object to alter its behavior when its internal state changes. The object will appear to change its class.

**Use cases:** Order status workflow, ATM machine, traffic lights, elevator, vending machine.

```java
// State interface
interface OrderState {
    void confirm(OrderContext order);
    void ship(OrderContext order);
    void deliver(OrderContext order);
    void cancel(OrderContext order);
    String getStatus();
}

// Context
class OrderContext {
    private OrderState state;
    private String orderId;
    
    public OrderContext(String orderId) {
        this.orderId = orderId;
        this.state = new PendingState(); // initial state
    }
    
    public void setState(OrderState state) { this.state = state; }
    
    // Delegate all operations to current state
    public void confirm() { state.confirm(this); }
    public void ship() { state.ship(this); }
    public void deliver() { state.deliver(this); }
    public void cancel() { state.cancel(this); }
    public String getStatus() { return state.getStatus(); }
    public String getOrderId() { return orderId; }
}

// Concrete states
class PendingState implements OrderState {
    public void confirm(OrderContext order) {
        System.out.println("Order confirmed");
        order.setState(new ConfirmedState());
    }
    public void ship(OrderContext o) { throw new IllegalStateException("Must confirm first"); }
    public void deliver(OrderContext o) { throw new IllegalStateException("Must ship first"); }
    public void cancel(OrderContext order) {
        System.out.println("Order cancelled from pending");
        order.setState(new CancelledState());
    }
    public String getStatus() { return "PENDING"; }
}

class ConfirmedState implements OrderState {
    public void confirm(OrderContext o) { System.out.println("Already confirmed"); }
    public void ship(OrderContext order) {
        System.out.println("Order shipped");
        order.setState(new ShippedState());
    }
    public void deliver(OrderContext o) { throw new IllegalStateException("Must ship first"); }
    public void cancel(OrderContext order) {
        System.out.println("Order cancelled from confirmed");
        order.setState(new CancelledState());
    }
    public String getStatus() { return "CONFIRMED"; }
}

class ShippedState implements OrderState {
    public void confirm(OrderContext o) { System.out.println("Already confirmed"); }
    public void ship(OrderContext o) { System.out.println("Already shipped"); }
    public void deliver(OrderContext order) {
        System.out.println("Order delivered!");
        order.setState(new DeliveredState());
    }
    public void cancel(OrderContext o) { throw new IllegalStateException("Cannot cancel shipped order"); }
    public String getStatus() { return "SHIPPED"; }
}

class DeliveredState implements OrderState {
    public void confirm(OrderContext o) { System.out.println("Already delivered"); }
    public void ship(OrderContext o) { System.out.println("Already delivered"); }
    public void deliver(OrderContext o) { System.out.println("Already delivered"); }
    public void cancel(OrderContext o) { throw new IllegalStateException("Cannot cancel delivered order"); }
    public String getStatus() { return "DELIVERED"; }
}

class CancelledState implements OrderState {
    public void confirm(OrderContext o) { throw new IllegalStateException("Order cancelled"); }
    public void ship(OrderContext o) { throw new IllegalStateException("Order cancelled"); }
    public void deliver(OrderContext o) { throw new IllegalStateException("Order cancelled"); }
    public void cancel(OrderContext o) { System.out.println("Already cancelled"); }
    public String getStatus() { return "CANCELLED"; }
}

// Usage
OrderContext order = new OrderContext("ORD-001");
System.out.println(order.getStatus()); // PENDING
order.confirm();
System.out.println(order.getStatus()); // CONFIRMED
order.ship();
order.deliver();
System.out.println(order.getStatus()); // DELIVERED
```

---

## Template Method

**Intent:** Define the skeleton of an algorithm in a base class, deferring some steps to subclasses.

**Use cases:** Report generation, data processing pipelines, game loops.

```java
// Abstract base class — defines the template
abstract class DataMigration {
    // Template method — final to prevent overriding the skeleton
    public final void migrate() {
        extractData();
        transformData();
        loadData();
        generateReport();
    }
    
    protected abstract void extractData();    // hook — must override
    protected abstract void transformData();  // hook — must override
    protected abstract void loadData();       // hook — must override
    
    // Optional hook — has default implementation
    protected void generateReport() {
        System.out.println("Migration complete. Report saved.");
    }
}

// Concrete classes override steps, not the skeleton
class MySQLToPostgresMigration extends DataMigration {
    protected void extractData() { System.out.println("Extracting from MySQL"); }
    protected void transformData() { System.out.println("Transforming MySQL → Postgres types"); }
    protected void loadData() { System.out.println("Loading into Postgres"); }
}

class CSVToMongoDBMigration extends DataMigration {
    protected void extractData() { System.out.println("Reading CSV files"); }
    protected void transformData() { System.out.println("Parsing CSV → JSON documents"); }
    protected void loadData() { System.out.println("Inserting into MongoDB"); }
    
    @Override // Override optional hook
    protected void generateReport() {
        System.out.println("MongoDB migration report: rows inserted = 50000");
    }
}

// Usage
DataMigration migration = new MySQLToPostgresMigration();
migration.migrate(); // Calls all steps in order
```

---

## Chain of Responsibility

**Intent:** Pass a request along a chain of handlers. Each handler decides to handle or pass it on.

**Use cases:** HTTP request middleware, logging levels, event bubbling, approval workflows.

```java
// Handler interface
abstract class SupportHandler {
    protected SupportHandler next;
    
    public SupportHandler setNext(SupportHandler next) {
        this.next = next;
        return next; // fluent chain building
    }
    
    public abstract void handle(SupportTicket ticket);
}

class SupportTicket {
    String issue;
    int severity; // 1=low, 2=medium, 3=high, 4=critical
    SupportTicket(String issue, int severity) { this.issue = issue; this.severity = severity; }
}

// Concrete handlers
class Level1Support extends SupportHandler {
    public void handle(SupportTicket ticket) {
        if (ticket.severity <= 1) {
            System.out.println("L1 Support resolved: " + ticket.issue);
        } else if (next != null) {
            System.out.println("L1 escalating to L2");
            next.handle(ticket);
        }
    }
}

class Level2Support extends SupportHandler {
    public void handle(SupportTicket ticket) {
        if (ticket.severity <= 2) {
            System.out.println("L2 Support resolved: " + ticket.issue);
        } else if (next != null) {
            System.out.println("L2 escalating to L3");
            next.handle(ticket);
        }
    }
}

class Level3Support extends SupportHandler {
    public void handle(SupportTicket ticket) {
        System.out.println("L3 (Engineering) handling critical issue: " + ticket.issue);
        // Always resolves
    }
}

// Usage — build chain
SupportHandler l1 = new Level1Support();
SupportHandler l2 = new Level2Support();
SupportHandler l3 = new Level3Support();

l1.setNext(l2).setNext(l3); // fluent chain

l1.handle(new SupportTicket("Password reset", 1)); // L1 resolves
l1.handle(new SupportTicket("App crashes", 3));    // escalates to L3
```

---

## Iterator

**Intent:** Provide a way to sequentially access elements of a collection without exposing its underlying representation.

```java
// Custom iterator for a range of numbers
class NumberRange implements Iterable<Integer> {
    private final int start, end, step;
    
    public NumberRange(int start, int end, int step) {
        this.start = start; this.end = end; this.step = step;
    }
    
    @Override
    public Iterator<Integer> iterator() {
        return new Iterator<>() {
            private int current = start;
            
            public boolean hasNext() { return current < end; }
            
            public Integer next() {
                if (!hasNext()) throw new NoSuchElementException();
                int val = current;
                current += step;
                return val;
            }
        };
    }
}

// Usage — works with enhanced for-each
for (int n : new NumberRange(0, 20, 3)) {
    System.out.print(n + " "); // 0 3 6 9 12 15 18
}
```

---

## Mediator

**Intent:** Define an object that encapsulates how a set of objects interact. Promotes loose coupling by keeping objects from referring to each other explicitly.

```java
// Mediator interface
interface ChatMediator {
    void sendMessage(String message, User sender);
    void addUser(User user);
}

abstract class User {
    protected ChatMediator mediator;
    protected String name;
    
    public User(ChatMediator mediator, String name) {
        this.mediator = mediator; this.name = name;
    }
    
    public abstract void send(String message);
    public abstract void receive(String message, String from);
}

class ChatRoom implements ChatMediator {
    private List<User> users = new ArrayList<>();
    
    public void addUser(User user) { users.add(user); }
    
    public void sendMessage(String message, User sender) {
        for (User user : users) {
            if (user != sender) { // don't send to self
                user.receive(message, sender.name);
            }
        }
    }
}

class ChatUser extends User {
    public ChatUser(ChatMediator mediator, String name) { super(mediator, name); }
    
    public void send(String message) {
        System.out.println(name + " sends: " + message);
        mediator.sendMessage(message, this);
    }
    
    public void receive(String message, String from) {
        System.out.println(name + " received from " + from + ": " + message);
    }
}
```

---

## Memento

**Intent:** Without violating encapsulation, capture and externalize an object's internal state so the object can be restored to that state later.

```java
// Memento — stores snapshot of state
class EditorMemento {
    private final String content;
    private final int cursorPosition;
    
    EditorMemento(String content, int cursorPosition) {
        this.content = content;
        this.cursorPosition = cursorPosition;
    }
    
    String getContent() { return content; }
    int getCursorPosition() { return cursorPosition; }
}

// Originator — creates and restores mementos
class TextEditorState {
    private String content;
    private int cursorPosition;
    
    public void write(String text) { content += text; cursorPosition = content.length(); }
    
    public EditorMemento save() {
        return new EditorMemento(content, cursorPosition); // snapshot
    }
    
    public void restore(EditorMemento memento) {
        this.content = memento.getContent();
        this.cursorPosition = memento.getCursorPosition();
    }
    
    public String getContent() { return content; }
}

// Caretaker — manages history
class UndoManager {
    private Deque<EditorMemento> history = new ArrayDeque<>();
    
    public void save(TextEditorState editor) { history.push(editor.save()); }
    
    public void undo(TextEditorState editor) {
        if (!history.isEmpty()) editor.restore(history.pop());
    }
}
```

---

## Visitor

**Intent:** Represent an operation to be performed on elements of an object structure. Visitor lets you add new operations without changing element classes.

```java
// Visitor interface — one method per element type
interface ShapeVisitor {
    void visit(Circle circle);
    void visit(Rectangle rectangle);
    void visit(Triangle triangle);
}

// Element interface
interface Shape {
    void accept(ShapeVisitor visitor); // double dispatch
}

class Circle implements Shape {
    double radius;
    Circle(double r) { radius = r; }
    public void accept(ShapeVisitor v) { v.visit(this); }
}
class Rectangle implements Shape {
    double width, height;
    Rectangle(double w, double h) { width = w; height = h; }
    public void accept(ShapeVisitor v) { v.visit(this); }
}

// Concrete visitors — add operations without modifying shapes
class AreaCalculator implements ShapeVisitor {
    public void visit(Circle c) { System.out.printf("Circle area: %.2f%n", Math.PI * c.radius * c.radius); }
    public void visit(Rectangle r) { System.out.printf("Rectangle area: %.2f%n", r.width * r.height); }
    public void visit(Triangle t) { }
}

class ShapeExporter implements ShapeVisitor {
    public void visit(Circle c) { System.out.println("<circle r=\"" + c.radius + "\"/>"); }
    public void visit(Rectangle r) { System.out.println("<rect w=\"" + r.width + "\" h=\"" + r.height + "\"/>"); }
    public void visit(Triangle t) { }
}

// Usage — add new operations (SVG export) without touching shapes
List<Shape> shapes = List.of(new Circle(5), new Rectangle(4, 6));
ShapeVisitor area = new AreaCalculator();
for (Shape s : shapes) s.accept(area);
```

---

## Interview Questions

### Q: Observer vs Event Bus — what's the difference?

**Observer:** Subject directly holds a list of observers and calls them. Tight coupling: subject knows observer interface.

**Event Bus (Pub/Sub):** A mediator sits in between. Publishers fire events to the bus; subscribers listen to specific event types. Completely decoupled — publishers don't know about subscribers.

Java example: Spring's `ApplicationEventPublisher`, Guava's `EventBus`.

### Q: State vs Strategy — they look similar, why are they different?

Both involve swapping behavior:
- **Strategy:** Algorithms are completely interchangeable; the context doesn't care which one runs. Client usually picks the strategy.
- **State:** States know about each other and trigger transitions. The state pattern manages a lifecycle (Pending → Confirmed → Shipped). States often transition automatically.

---

## ⚡ Flashcard Summary

| # | Pattern | One-liner |
|---|---------|-----------|
| 1 | Observer | Subject notifies all observers on state change |
| 2 | Observer Java | EventListener, PropertyChangeListener |
| 3 | Strategy | Encapsulate algorithm; swap at runtime |
| 4 | Strategy vs Template | Strategy: composition; Template: inheritance |
| 5 | Command | Encapsulate request; enables undo/redo/queue |
| 6 | Command components | Invoker → Command → Receiver |
| 7 | State | Object changes behavior based on state; delegates to state object |
| 8 | State vs Strategy | State: lifecycle/transitions; Strategy: interchangeable algorithms |
| 9 | Template Method | Skeleton in base; steps in subclasses; final template method |
| 10 | Chain of Responsibility | Request passes along handler chain; each decides to handle or pass |
| 11 | Iterator | Sequential access without exposing structure; Java Iterable |
| 12 | Mediator | Central coordinator reduces coupling between many objects |
| 13 | Mediator vs Observer | Mediator: bi-directional coordination; Observer: broadcast |
| 14 | Memento | Save/restore snapshot; caretaker manages history |
| 15 | Visitor | Add operations to class hierarchy without modifying it |

---

*Next: [Concurrency Patterns](04-concurrency-patterns.md)*
