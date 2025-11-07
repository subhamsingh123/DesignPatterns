# Complete Design Patterns Guide - Questions & Solutions

## Table of Contents
1. [Creational Patterns](#creational-patterns)
   - [Singleton Pattern](#1-singleton-pattern)
   - [Factory Method Pattern](#2-factory-method-pattern)
   - [Abstract Factory Pattern](#3-abstract-factory-pattern)
   - [Builder Pattern](#4-builder-pattern)
   - [Prototype Pattern](#5-prototype-pattern)

2. [Structural Patterns](#structural-patterns)
   - [Adapter Pattern](#6-adapter-pattern)
   - [Bridge Pattern](#7-bridge-pattern)
   - [Composite Pattern](#8-composite-pattern)
   - [Decorator Pattern](#9-decorator-pattern)
   - [Facade Pattern](#10-facade-pattern)
   - [Flyweight Pattern](#11-flyweight-pattern)
   - [Proxy Pattern](#12-proxy-pattern)

3. [Behavioral Patterns](#behavioral-patterns)
   - [Chain of Responsibility Pattern](#13-chain-of-responsibility-pattern)
   - [Command Pattern](#14-command-pattern)
   - [Iterator Pattern](#15-iterator-pattern)
   - [Mediator Pattern](#16-mediator-pattern)
   - [Memento Pattern](#17-memento-pattern)
   - [Observer Pattern](#18-observer-pattern)
   - [State Pattern](#19-state-pattern)
   - [Strategy Pattern](#20-strategy-pattern)
   - [Template Method Pattern](#21-template-method-pattern)
   - [Visitor Pattern](#22-visitor-pattern)

---

## Creational Patterns

### 1. Singleton Pattern

**Question:** Design a database connection manager that ensures only one connection pool exists throughout the application. The manager should handle connection creation, provide thread-safe access, and include a method to get available connections count.

**Solution:**

```java
public class DatabaseConnectionManager {
    private static volatile DatabaseConnectionManager instance;
    private static final Object lock = new Object();
    private final int maxConnections = 10;
    private int activeConnections = 0;
    
    private DatabaseConnectionManager() {
        // Private constructor to prevent instantiation
        System.out.println("Initializing Database Connection Manager");
    }
    
    public static DatabaseConnectionManager getInstance() {
        if (instance == null) {
            synchronized (lock) {
                if (instance == null) {
                    instance = new DatabaseConnectionManager();
                }
            }
        }
        return instance;
    }
    
    public synchronized boolean getConnection() {
        if (activeConnections < maxConnections) {
            activeConnections++;
            System.out.println("Connection granted. Active: " + activeConnections);
            return true;
        }
        System.out.println("Connection pool full!");
        return false;
    }
    
    public synchronized void releaseConnection() {
        if (activeConnections > 0) {
            activeConnections--;
            System.out.println("Connection released. Active: " + activeConnections);
        }
    }
    
    public int getAvailableConnections() {
        return maxConnections - activeConnections;
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        DatabaseConnectionManager db1 = DatabaseConnectionManager.getInstance();
        DatabaseConnectionManager db2 = DatabaseConnectionManager.getInstance();
        
        System.out.println("Same instance? " + (db1 == db2)); // true
        
        db1.getConnection();
        db1.getConnection();
        System.out.println("Available connections: " + db2.getAvailableConnections());
    }
}
```

**Why Singleton?**
- Ensures single point of control for database connections
- Prevents resource wastage from multiple connection pools
- Thread-safe implementation using double-checked locking
- Global access point for all parts of the application

---

### 2. Factory Method Pattern

**Question:** Create a notification system where different types of notifications (Email, SMS, Push) can be sent. The system should allow adding new notification types without modifying existing code.

**Solution:**

```java
// Product interface
interface Notification {
    void send(String message);
    String getType();
}

// Concrete Products
class EmailNotification implements Notification {
    private String recipient;
    
    public EmailNotification(String recipient) {
        this.recipient = recipient;
    }
    
    @Override
    public void send(String message) {
        System.out.println("Email sent to " + recipient + ": " + message);
    }
    
    @Override
    public String getType() {
        return "EMAIL";
    }
}

class SMSNotification implements Notification {
    private String phoneNumber;
    
    public SMSNotification(String phoneNumber) {
        this.phoneNumber = phoneNumber;
    }
    
    @Override
    public void send(String message) {
        System.out.println("SMS sent to " + phoneNumber + ": " + message);
    }
    
    @Override
    public String getType() {
        return "SMS";
    }
}

class PushNotification implements Notification {
    private String deviceId;
    
    public PushNotification(String deviceId) {
        this.deviceId = deviceId;
    }
    
    @Override
    public void send(String message) {
        System.out.println("Push notification sent to device " + deviceId + ": " + message);
    }
    
    @Override
    public String getType() {
        return "PUSH";
    }
}

// Creator
abstract class NotificationFactory {
    public abstract Notification createNotification(String recipient);
    
    public void notify(String recipient, String message) {
        Notification notification = createNotification(recipient);
        notification.send(message);
    }
}

// Concrete Creators
class EmailNotificationFactory extends NotificationFactory {
    @Override
    public Notification createNotification(String recipient) {
        return new EmailNotification(recipient);
    }
}

class SMSNotificationFactory extends NotificationFactory {
    @Override
    public Notification createNotification(String recipient) {
        return new SMSNotification(recipient);
    }
}

class PushNotificationFactory extends NotificationFactory {
    @Override
    public Notification createNotification(String recipient) {
        return new PushNotification(recipient);
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        NotificationFactory emailFactory = new EmailNotificationFactory();
        emailFactory.notify("user@example.com", "Your order has been shipped!");
        
        NotificationFactory smsFactory = new SMSNotificationFactory();
        smsFactory.notify("+1234567890", "Your OTP is 123456");
        
        NotificationFactory pushFactory = new PushNotificationFactory();
        pushFactory.notify("device123", "New message received");
    }
}
```

**Why Factory Method?**
- Encapsulates object creation logic
- Supports Open/Closed Principle - new notification types can be added without modifying existing code
- Provides flexibility in choosing notification type at runtime
- Delegates instantiation to subclasses

---

### 3. Abstract Factory Pattern

**Question:** Design a UI component library that can create families of related components (Button, TextField, Checkbox) for different operating systems (Windows, MacOS, Linux). Each OS has its own styling for components.

**Solution:**

```java
// Abstract Products
interface Button {
    void render();
    void onClick();
}

interface TextField {
    void render();
    void setText(String text);
}

interface Checkbox {
    void render();
    void setChecked(boolean checked);
}

// Concrete Products - Windows
class WindowsButton implements Button {
    @Override
    public void render() {
        System.out.println("Rendering Windows style button");
    }
    
    @Override
    public void onClick() {
        System.out.println("Windows button clicked");
    }
}

class WindowsTextField implements TextField {
    private String text = "";
    
    @Override
    public void render() {
        System.out.println("Rendering Windows style text field");
    }
    
    @Override
    public void setText(String text) {
        this.text = text;
        System.out.println("Windows text field: " + text);
    }
}

class WindowsCheckbox implements Checkbox {
    private boolean checked = false;
    
    @Override
    public void render() {
        System.out.println("Rendering Windows style checkbox");
    }
    
    @Override
    public void setChecked(boolean checked) {
        this.checked = checked;
        System.out.println("Windows checkbox: " + (checked ? "✓" : "□"));
    }
}

// Concrete Products - MacOS
class MacOSButton implements Button {
    @Override
    public void render() {
        System.out.println("Rendering MacOS style button");
    }
    
    @Override
    public void onClick() {
        System.out.println("MacOS button clicked");
    }
}

class MacOSTextField implements TextField {
    private String text = "";
    
    @Override
    public void render() {
        System.out.println("Rendering MacOS style text field");
    }
    
    @Override
    public void setText(String text) {
        this.text = text;
        System.out.println("MacOS text field: " + text);
    }
}

class MacOSCheckbox implements Checkbox {
    private boolean checked = false;
    
    @Override
    public void render() {
        System.out.println("Rendering MacOS style checkbox");
    }
    
    @Override
    public void setChecked(boolean checked) {
        this.checked = checked;
        System.out.println("MacOS checkbox: " + (checked ? "✓" : "○"));
    }
}

// Abstract Factory
interface UIFactory {
    Button createButton();
    TextField createTextField();
    Checkbox createCheckbox();
}

// Concrete Factories
class WindowsUIFactory implements UIFactory {
    @Override
    public Button createButton() {
        return new WindowsButton();
    }
    
    @Override
    public TextField createTextField() {
        return new WindowsTextField();
    }
    
    @Override
    public Checkbox createCheckbox() {
        return new WindowsCheckbox();
    }
}

class MacOSUIFactory implements UIFactory {
    @Override
    public Button createButton() {
        return new MacOSButton();
    }
    
    @Override
    public TextField createTextField() {
        return new MacOSTextField();
    }
    
    @Override
    public Checkbox createCheckbox() {
        return new MacOSCheckbox();
    }
}

// Application
class Application {
    private Button button;
    private TextField textField;
    private Checkbox checkbox;
    
    public Application(UIFactory factory) {
        button = factory.createButton();
        textField = factory.createTextField();
        checkbox = factory.createCheckbox();
    }
    
    public void renderUI() {
        button.render();
        textField.render();
        checkbox.render();
    }
    
    public void simulateUserInput() {
        button.onClick();
        textField.setText("Hello World");
        checkbox.setChecked(true);
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        String os = "Windows"; // Could be determined at runtime
        
        UIFactory factory;
        if (os.equals("Windows")) {
            factory = new WindowsUIFactory();
        } else {
            factory = new MacOSUIFactory();
        }
        
        Application app = new Application(factory);
        app.renderUI();
        app.simulateUserInput();
    }
}
```

**Why Abstract Factory?**
- Creates families of related objects without specifying their concrete classes
- Ensures consistency among products (all Windows components or all Mac components)
- Easy to switch between product families
- Supports adding new product families without changing existing code

---

### 4. Builder Pattern

**Question:** Create a system to build custom computers with various configurations (CPU, RAM, Storage, GPU, etc.). Some components are mandatory while others are optional. The system should validate configurations and provide a fluent interface for building.

**Solution:**

```java
// Product
class Computer {
    // Required parameters
    private final String CPU;
    private final String RAM;
    
    // Optional parameters
    private final String storage;
    private final String GPU;
    private final String motherboard;
    private final boolean hasWiFi;
    private final boolean hasBluetooth;
    private final String coolingSystem;
    
    private Computer(ComputerBuilder builder) {
        this.CPU = builder.CPU;
        this.RAM = builder.RAM;
        this.storage = builder.storage;
        this.GPU = builder.GPU;
        this.motherboard = builder.motherboard;
        this.hasWiFi = builder.hasWiFi;
        this.hasBluetooth = builder.hasBluetooth;
        this.coolingSystem = builder.coolingSystem;
    }
    
    @Override
    public String toString() {
        return "Computer Configuration:\n" +
               "CPU: " + CPU + "\n" +
               "RAM: " + RAM + "\n" +
               "Storage: " + storage + "\n" +
               "GPU: " + (GPU != null ? GPU : "Integrated") + "\n" +
               "Motherboard: " + motherboard + "\n" +
               "WiFi: " + (hasWiFi ? "Yes" : "No") + "\n" +
               "Bluetooth: " + (hasBluetooth ? "Yes" : "No") + "\n" +
               "Cooling: " + coolingSystem;
    }
    
    // Builder
    public static class ComputerBuilder {
        // Required parameters
        private final String CPU;
        private final String RAM;
        
        // Optional parameters with default values
        private String storage = "256GB SSD";
        private String GPU;
        private String motherboard = "Standard ATX";
        private boolean hasWiFi = false;
        private boolean hasBluetooth = false;
        private String coolingSystem = "Stock Cooler";
        
        public ComputerBuilder(String CPU, String RAM) {
            this.CPU = CPU;
            this.RAM = RAM;
        }
        
        public ComputerBuilder withStorage(String storage) {
            this.storage = storage;
            return this;
        }
        
        public ComputerBuilder withGPU(String GPU) {
            this.GPU = GPU;
            return this;
        }
        
        public ComputerBuilder withMotherboard(String motherboard) {
            this.motherboard = motherboard;
            return this;
        }
        
        public ComputerBuilder withWiFi() {
            this.hasWiFi = true;
            return this;
        }
        
        public ComputerBuilder withBluetooth() {
            this.hasBluetooth = true;
            return this;
        }
        
        public ComputerBuilder withCoolingSystem(String coolingSystem) {
            this.coolingSystem = coolingSystem;
            return this;
        }
        
        public Computer build() {
            // Validation
            if (GPU != null && GPU.contains("RTX 4090") && !coolingSystem.contains("Liquid")) {
                throw new IllegalStateException("RTX 4090 requires liquid cooling!");
            }
            
            if (RAM.contains("64GB") && !motherboard.contains("Pro")) {
                throw new IllegalStateException("64GB RAM requires Pro motherboard!");
            }
            
            return new Computer(this);
        }
    }
}

// Director (optional)
class ComputerDirector {
    public Computer buildGamingPC() {
        return new Computer.ComputerBuilder("Intel i9-13900K", "32GB DDR5")
                .withStorage("2TB NVMe SSD")
                .withGPU("NVIDIA RTX 4080")
                .withMotherboard("ASUS ROG Pro")
                .withWiFi()
                .withBluetooth()
                .withCoolingSystem("RGB Liquid Cooling")
                .build();
    }
    
    public Computer buildOfficePC() {
        return new Computer.ComputerBuilder("Intel i5-13400", "16GB DDR4")
                .withStorage("512GB SSD")
                .withMotherboard("MSI Standard")
                .withWiFi()
                .build();
    }
    
    public Computer buildServerPC() {
        return new Computer.ComputerBuilder("AMD EPYC 7763", "128GB ECC")
                .withStorage("10TB RAID Array")
                .withMotherboard("Supermicro Server Board")
                .withCoolingSystem("Redundant Cooling System")
                .build();
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        // Custom build
        Computer customPC = new Computer.ComputerBuilder("AMD Ryzen 9 7950X", "32GB DDR5")
                .withStorage("1TB NVMe SSD")
                .withGPU("NVIDIA RTX 4070 Ti")
                .withWiFi()
                .withBluetooth()
                .withCoolingSystem("AIO Liquid Cooler")
                .build();
        
        System.out.println(customPC);
        System.out.println("\n-------------------\n");
        
        // Using director
        ComputerDirector director = new ComputerDirector();
        Computer gamingPC = director.buildGamingPC();
        System.out.println("Gaming PC:\n" + gamingPC);
    }
}
```

**Why Builder?**
- Handles complex object construction with many parameters
- Provides clear, readable code with fluent interface
- Enforces mandatory parameters while allowing optional ones
- Enables validation before object creation
- Immutable objects once built

---

### 5. Prototype Pattern

**Question:** Design a document template system where users can create document templates (Resume, Report, Invoice) and clone them to create new documents with pre-filled structures while allowing modifications.

**Solution:**

```java
import java.util.*;

// Prototype interface
interface DocumentPrototype extends Cloneable {
    DocumentPrototype clone();
    void print();
    void customize(String title, String content);
}

// Concrete Prototype - Resume
class Resume implements DocumentPrototype {
    private String title;
    private String name;
    private List<String> skills;
    private List<String> experience;
    private Map<String, String> sections;
    
    public Resume(String title, String name) {
        this.title = title;
        this.name = name;
        this.skills = new ArrayList<>();
        this.experience = new ArrayList<>();
        this.sections = new HashMap<>();
        initializeTemplate();
    }
    
    private void initializeTemplate() {
        skills.add("Java");
        skills.add("Design Patterns");
        experience.add("Software Developer - 3 years");
        sections.put("Education", "Bachelor's in Computer Science");
        sections.put("Objective", "Seeking challenging position...");
    }
    
    @Override
    public Resume clone() {
        try {
            Resume cloned = (Resume) super.clone();
            // Deep copy for mutable fields
            cloned.skills = new ArrayList<>(this.skills);
            cloned.experience = new ArrayList<>(this.experience);
            cloned.sections = new HashMap<>(this.sections);
            return cloned;
        } catch (CloneNotSupportedException e) {
            return null;
        }
    }
    
    @Override
    public void customize(String title, String content) {
        this.title = title;
        this.name = content;
    }
    
    public void addSkill(String skill) {
        skills.add(skill);
    }
    
    public void addExperience(String exp) {
        experience.add(exp);
    }
    
    @Override
    public void print() {
        System.out.println("=== " + title + " ===");
        System.out.println("Name: " + name);
        System.out.println("Skills: " + String.join(", ", skills));
        System.out.println("Experience: " + String.join("; ", experience));
        sections.forEach((k, v) -> System.out.println(k + ": " + v));
    }
}

// Concrete Prototype - Report
class Report implements DocumentPrototype {
    private String title;
    private String author;
    private List<String> chapters;
    private String summary;
    private Date createdDate;
    
    public Report(String title, String author) {
        this.title = title;
        this.author = author;
        this.chapters = new ArrayList<>();
        this.createdDate = new Date();
        initializeTemplate();
    }
    
    private void initializeTemplate() {
        chapters.add("1. Introduction");
        chapters.add("2. Methodology");
        chapters.add("3. Results");
        chapters.add("4. Conclusion");
        summary = "Executive Summary placeholder...";
    }
    
    @Override
    public Report clone() {
        try {
            Report cloned = (Report) super.clone();
            cloned.chapters = new ArrayList<>(this.chapters);
            cloned.createdDate = (Date) this.createdDate.clone();
            return cloned;
        } catch (CloneNotSupportedException e) {
            return null;
        }
    }
    
    @Override
    public void customize(String title, String content) {
        this.title = title;
        this.summary = content;
    }
    
    public void addChapter(String chapter) {
        chapters.add(chapter);
    }
    
    @Override
    public void print() {
        System.out.println("=== " + title + " ===");
        System.out.println("Author: " + author);
        System.out.println("Date: " + createdDate);
        System.out.println("Summary: " + summary);
        System.out.println("Chapters:");
        chapters.forEach(ch -> System.out.println("  " + ch));
    }
}

// Prototype Registry
class DocumentTemplateRegistry {
    private Map<String, DocumentPrototype> templates = new HashMap<>();
    
    public void registerTemplate(String key, DocumentPrototype prototype) {
        templates.put(key, prototype);
    }
    
    public DocumentPrototype createDocument(String key) {
        DocumentPrototype template = templates.get(key);
        if (template != null) {
            return template.clone();
        }
        return null;
    }
    
    public void listTemplates() {
        System.out.println("Available templates: " + templates.keySet());
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        // Create and register templates
        DocumentTemplateRegistry registry = new DocumentTemplateRegistry();
        
        Resume resumeTemplate = new Resume("Professional Resume", "John Doe");
        Report reportTemplate = new Report("Annual Report", "Company XYZ");
        
        registry.registerTemplate("resume", resumeTemplate);
        registry.registerTemplate("report", reportTemplate);
        
        // Clone and customize documents
        Resume myResume = (Resume) registry.createDocument("resume");
        myResume.customize("My Resume", "Jane Smith");
        myResume.addSkill("Python");
        myResume.addExperience("Senior Developer - 5 years");
        
        Resume anotherResume = (Resume) registry.createDocument("resume");
        anotherResume.customize("Developer Resume", "Bob Johnson");
        anotherResume.addSkill("JavaScript");
        
        Report quarterlyReport = (Report) registry.createDocument("report");
        quarterlyReport.customize("Q1 2024 Report", "First quarter analysis...");
        quarterlyReport.addChapter("5. Recommendations");
        
        // Display documents
        myResume.print();
        System.out.println("\n-------------------\n");
        anotherResume.print();
        System.out.println("\n-------------------\n");
        quarterlyReport.print();
    }
}
```

**Why Prototype?**
- Avoids expensive object creation from scratch
- Provides templates with predefined structures
- Allows runtime addition of new prototypes
- Reduces subclassing for creating variations
- Useful when object initialization is resource-intensive

---

## Structural Patterns

### 6. Adapter Pattern

**Question:** You have a legacy payment system that processes payments in USD only, but your new e-commerce platform needs to support multiple currencies. Create an adapter to make the legacy system work with the new multi-currency interface without modifying the legacy code.

**Solution:**

```java
// Legacy payment system (Adaptee)
class LegacyPaymentSystem {
    public void makePayment(String accountNumber, double amountInUSD) {
        System.out.println("Processing payment through legacy system:");
        System.out.println("Account: " + accountNumber);
        System.out.println("Amount: $" + String.format("%.2f", amountInUSD) + " USD");
        System.out.println("Payment successful!\n");
    }
    
    public double checkBalance(String accountNumber) {
        // Simulated balance check
        return 5000.00; // Returns balance in USD
    }
}

// Target interface (what the client expects)
interface ModernPaymentInterface {
    void processPayment(String accountNumber, double amount, String currency);
    double getBalance(String accountNumber, String currency);
    boolean validateCurrency(String currency);
}

// Adapter class
class PaymentAdapter implements ModernPaymentInterface {
    private LegacyPaymentSystem legacySystem;
    private Map<String, Double> exchangeRates;
    
    public PaymentAdapter(LegacyPaymentSystem legacySystem) {
        this.legacySystem = legacySystem;
        initializeExchangeRates();
    }
    
    private void initializeExchangeRates() {
        exchangeRates = new HashMap<>();
        exchangeRates.put("USD", 1.0);
        exchangeRates.put("EUR", 0.92);
        exchangeRates.put("GBP", 0.79);
        exchangeRates.put("JPY", 149.50);
        exchangeRates.put("INR", 83.12);
    }
    
    @Override
    public void processPayment(String accountNumber, double amount, String currency) {
        if (!validateCurrency(currency)) {
            System.out.println("Error: Unsupported currency " + currency);
            return;
        }
        
        double amountInUSD = convertToUSD(amount, currency);
        System.out.println("Converting " + amount + " " + currency + 
                          " to " + String.format("%.2f", amountInUSD) + " USD");
        
        legacySystem.makePayment(accountNumber, amountInUSD);
    }
    
    @Override
    public double getBalance(String accountNumber, String currency) {
        double balanceInUSD = legacySystem.checkBalance(accountNumber);
        double convertedBalance = convertFromUSD(balanceInUSD, currency);
        return convertedBalance;
    }
    
    @Override
    public boolean validateCurrency(String currency) {
        return exchangeRates.containsKey(currency);
    }
    
    private double convertToUSD(double amount, String currency) {
        return amount / exchangeRates.get(currency);
    }
    
    private double convertFromUSD(double amountInUSD, String currency) {
        return amountInUSD * exchangeRates.get(currency);
    }
}

// Modern client code
class EcommercePaymentService {
    private ModernPaymentInterface paymentInterface;
    
    public EcommercePaymentService(ModernPaymentInterface paymentInterface) {
        this.paymentInterface = paymentInterface;
    }
    
    public void checkout(String accountNumber, double amount, String currency) {
        System.out.println("=== E-commerce Checkout ===");
        
        // Check balance
        double balance = paymentInterface.getBalance(accountNumber, currency);
        System.out.println("Current balance: " + String.format("%.2f", balance) + " " + currency);
        
        // Process payment
        if (balance >= amount) {
            paymentInterface.processPayment(accountNumber, amount, currency);
        } else {
            System.out.println("Insufficient funds!");
        }
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        // Legacy system
        LegacyPaymentSystem legacySystem = new LegacyPaymentSystem();
        
        // Adapter wraps the legacy system
        ModernPaymentInterface adapter = new PaymentAdapter(legacySystem);
        
        // Modern service uses the adapter
        EcommercePaymentService ecommerce = new EcommercePaymentService(adapter);
        
        // Process payments in different currencies
        ecommerce.checkout("ACC123", 100.00, "EUR");
        ecommerce.checkout("ACC456", 50000, "JPY");
        ecommerce.checkout("ACC789", 500.00, "GBP");
    }
}
```

**Why Adapter?**
- Allows incompatible interfaces to work together
- Protects client from legacy system changes
- No modification needed to legacy code
- Can add new functionality during adaptation
- Supports both class and object adapter variations

---

### 7. Bridge Pattern

**Question:** Design a remote control system for different devices (TV, Radio, SmartLight) where each device can have different types of remotes (BasicRemote, AdvancedRemote) with varying functionalities. The system should allow adding new devices and remote types independently.

**Solution:**

```java
// Implementor - Device interface
interface Device {
    void turnOn();
    void turnOff();
    boolean isEnabled();
    void setVolume(int volume);
    int getVolume();
    void setChannel(int channel);
    int getChannel();
    String getDeviceInfo();
}

// Concrete Implementors
class Television implements Device {
    private boolean on = false;
    private int volume = 30;
    private int channel = 1;
    private String brand;
    
    public Television(String brand) {
        this.brand = brand;
    }
    
    @Override
    public void turnOn() {
        on = true;
        System.out.println(brand + " TV is ON");
    }
    
    @Override
    public void turnOff() {
        on = false;
        System.out.println(brand + " TV is OFF");
    }
    
    @Override
    public boolean isEnabled() {
        return on;
    }
    
    @Override
    public void setVolume(int volume) {
        this.volume = Math.max(0, Math.min(100, volume));
        System.out.println(brand + " TV volume set to " + this.volume);
    }
    
    @Override
    public int getVolume() {
        return volume;
    }
    
    @Override
    public void setChannel(int channel) {
        this.channel = channel;
        System.out.println(brand + " TV switched to channel " + channel);
    }
    
    @Override
    public int getChannel() {
        return channel;
    }
    
    @Override
    public String getDeviceInfo() {
        return brand + " TV [" + (on ? "ON" : "OFF") + 
               "] Channel: " + channel + ", Volume: " + volume;
    }
}

class Radio implements Device {
    private boolean on = false;
    private int volume = 20;
    private int channel = 88; // FM frequency
    
    @Override
    public void turnOn() {
        on = true;
        System.out.println("Radio is ON");
    }
    
    @Override
    public void turnOff() {
        on = false;
        System.out.println("Radio is OFF");
    }
    
    @Override
    public boolean isEnabled() {
        return on;
    }
    
    @Override
    public void setVolume(int volume) {
        this.volume = Math.max(0, Math.min(100, volume));
        System.out.println("Radio volume set to " + this.volume);
    }
    
    @Override
    public int getVolume() {
        return volume;
    }
    
    @Override
    public void setChannel(int frequency) {
        this.channel = frequency;
        System.out.println("Radio tuned to " + frequency + " FM");
    }
    
    @Override
    public int getChannel() {
        return channel;
    }
    
    @Override
    public String getDeviceInfo() {
        return "Radio [" + (on ? "ON" : "OFF") + 
               "] Frequency: " + channel + " FM, Volume: " + volume;
    }
}

class SmartLight implements Device {
    private boolean on = false;
    private int brightness = 50; // Using volume as brightness
    private int colorTemp = 3000; // Using channel as color temperature
    
    @Override
    public void turnOn() {
        on = true;
        System.out.println("Smart Light is ON");
    }
    
    @Override
    public void turnOff() {
        on = false;
        System.out.println("Smart Light is OFF");
    }
    
    @Override
    public boolean isEnabled() {
        return on;
    }
    
    @Override
    public void setVolume(int brightness) {
        this.brightness = Math.max(0, Math.min(100, brightness));
        System.out.println("Light brightness set to " + this.brightness + "%");
    }
    
    @Override
    public int getVolume() {
        return brightness;
    }
    
    @Override
    public void setChannel(int colorTemp) {
        this.colorTemp = Math.max(2700, Math.min(6500, colorTemp));
        System.out.println("Light color temperature set to " + this.colorTemp + "K");
    }
    
    @Override
    public int getChannel() {
        return colorTemp;
    }
    
    @Override
    public String getDeviceInfo() {
        return "Smart Light [" + (on ? "ON" : "OFF") + 
               "] Brightness: " + brightness + "%, Color Temp: " + colorTemp + "K";
    }
}

// Abstraction - Remote Control
abstract class RemoteControl {
    protected Device device;
    
    public RemoteControl(Device device) {
        this.device = device;
    }
    
    public void togglePower() {
        if (device.isEnabled()) {
            device.turnOff();
        } else {
            device.turnOn();
        }
    }
    
    public void volumeUp() {
        device.setVolume(device.getVolume() + 10);
    }
    
    public void volumeDown() {
        device.setVolume(device.getVolume() - 10);
    }
    
    public void channelUp() {
        device.setChannel(device.getChannel() + 1);
    }
    
    public void channelDown() {
        device.setChannel(device.getChannel() - 1);
    }
    
    public abstract void showInfo();
}

// Refined Abstractions
class BasicRemote extends RemoteControl {
    public BasicRemote(Device device) {
        super(device);
    }
    
    @Override
    public void showInfo() {
        System.out.println("Basic Remote: " + device.getDeviceInfo());
    }
}

class AdvancedRemote extends RemoteControl {
    public AdvancedRemote(Device device) {
        super(device);
    }
    
    public void mute() {
        System.out.println("Muting device");
        device.setVolume(0);
    }
    
    public void setFavoriteChannel(int channel) {
        System.out.println("Setting favorite channel: " + channel);
        device.setChannel(channel);
    }
    
    @Override
    public void showInfo() {
        System.out.println("Advanced Remote: " + device.getDeviceInfo());
    }
}

// Voice-controlled remote (new abstraction)
class VoiceRemote extends RemoteControl {
    public VoiceRemote(Device device) {
        super(device);
    }
    
    public void voiceCommand(String command) {
        System.out.println("Voice command: \"" + command + "\"");
        switch (command.toLowerCase()) {
            case "turn on":
                device.turnOn();
                break;
            case "turn off":
                device.turnOff();
                break;
            case "volume up":
                volumeUp();
                break;
            case "volume down":
                volumeDown();
                break;
            default:
                System.out.println("Command not recognized");
        }
    }
    
    @Override
    public void showInfo() {
        System.out.println("Voice Remote: " + device.getDeviceInfo());
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        // Different devices
        Device sonyTV = new Television("Sony");
        Device radio = new Radio();
        Device smartLight = new SmartLight();
        
        // Different remotes for same device type
        System.out.println("=== Basic Remote with TV ===");
        BasicRemote basicTVRemote = new BasicRemote(sonyTV);
        basicTVRemote.togglePower();
        basicTVRemote.volumeUp();
        basicTVRemote.channelUp();
        basicTVRemote.showInfo();
        
        System.out.println("\n=== Advanced Remote with TV ===");
        AdvancedRemote advancedTVRemote = new AdvancedRemote(sonyTV);
        advancedTVRemote.setFavoriteChannel(45);
        advancedTVRemote.mute();
        advancedTVRemote.showInfo();
        
        System.out.println("\n=== Voice Remote with Radio ===");
        VoiceRemote voiceRadioRemote = new VoiceRemote(radio);
        voiceRadioRemote.voiceCommand("turn on");
        voiceRadioRemote.voiceCommand("volume up");
        voiceRadioRemote.showInfo();
        
        System.out.println("\n=== Advanced Remote with Smart Light ===");
        AdvancedRemote lightRemote = new AdvancedRemote(smartLight);
        lightRemote.togglePower();
        lightRemote.volumeUp(); // Controls brightness
        lightRemote.setFavoriteChannel(4000); // Sets color temperature
        lightRemote.showInfo();
    }
}
```

**Why Bridge?**
- Separates abstraction (Remote) from implementation (Device)
- Both hierarchies can evolve independently
- Avoids cartesian product complexity (n remotes × m devices)
- Runtime binding of implementation
- Promotes loose coupling and flexibility

---

### 8. Composite Pattern

**Question:** Create a file system structure where both files and directories can be treated uniformly. Implement operations like calculating total size, searching for files, and displaying the structure with proper indentation.

**Solution:**

```java
import java.util.*;

// Component
abstract class FileSystemComponent {
    protected String name;
    
    public FileSystemComponent(String name) {
        this.name = name;
    }
    
    public abstract long getSize();
    public abstract void display(String indent);
    public abstract boolean search(String keyword);
    
    // Default implementations for composite operations
    public void add(FileSystemComponent component) {
        throw new UnsupportedOperationException("Cannot add to a file");
    }
    
    public void remove(FileSystemComponent component) {
        throw new UnsupportedOperationException("Cannot remove from a file");
    }
    
    public List<FileSystemComponent> getChildren() {
        throw new UnsupportedOperationException("File has no children");
    }
}

// Leaf
class File extends FileSystemComponent {
    private long size;
    private String extension;
    
    public File(String name, long size) {
        super(name);
        this.size = size;
        this.extension = name.substring(name.lastIndexOf('.') + 1);
    }
    
    @Override
    public long getSize() {
        return size;
    }
    
    @Override
    public void display(String indent) {
        System.out.println(indent + "📄 " + name + " (" + formatSize(size) + ")");
    }
    
    @Override
    public boolean search(String keyword) {
        return name.toLowerCase().contains(keyword.toLowerCase());
    }
    
    private String formatSize(long bytes) {
        if (bytes < 1024) return bytes + " B";
        if (bytes < 1024 * 1024) return (bytes / 1024) + " KB";
        return (bytes / (1024 * 1024)) + " MB";
    }
}

// Composite
class Directory extends FileSystemComponent {
    private List<FileSystemComponent> children;
    
    public Directory(String name) {
        super(name);
        this.children = new ArrayList<>();
    }
    
    @Override
    public void add(FileSystemComponent component) {
        children.add(component);
    }
    
    @Override
    public void remove(FileSystemComponent component) {
        children.remove(component);
    }
    
    @Override
    public List<FileSystemComponent> getChildren() {
        return children;
    }
    
    @Override
    public long getSize() {
        long totalSize = 0;
        for (FileSystemComponent component : children) {
            totalSize += component.getSize();
        }
        return totalSize;
    }
    
    @Override
    public void display(String indent) {
        System.out.println(indent + "📁 " + name + " (" + formatSize(getSize()) + ")");
        for (FileSystemComponent component : children) {
            component.display(indent + "  ");
        }
    }
    
    @Override
    public boolean search(String keyword) {
        if (name.toLowerCase().contains(keyword.toLowerCase())) {
            return true;
        }
        for (FileSystemComponent component : children) {
            if (component.search(keyword)) {
                return true;
            }
        }
        return false;
    }
    
    private String formatSize(long bytes) {
        if (bytes < 1024) return bytes + " B";
        if (bytes < 1024 * 1024) return (bytes / 1024) + " KB";
        return (bytes / (1024 * 1024)) + " MB";
    }
    
    public List<File> findFilesByExtension(String extension) {
        List<File> files = new ArrayList<>();
        findFilesByExtensionRecursive(this, extension, files);
        return files;
    }
    
    private void findFilesByExtensionRecursive(Directory dir, String extension, List<File> files) {
        for (FileSystemComponent component : dir.getChildren()) {
            if (component instanceof File) {
                File file = (File) component;
                if (file.name.endsWith("." + extension)) {
                    files.add(file);
                }
            } else if (component instanceof Directory) {
                findFilesByExtensionRecursive((Directory) component, extension, files);
            }
        }
    }
}

// Client
class FileExplorer {
    private Directory root;
    
    public FileExplorer() {
        createSampleFileSystem();
    }
    
    private void createSampleFileSystem() {
        root = new Directory("root");
        
        // Create structure
        Directory documents = new Directory("Documents");
        Directory pictures = new Directory("Pictures");
        Directory projects = new Directory("Projects");
        
        // Add files to Documents
        documents.add(new File("resume.pdf", 245000));
        documents.add(new File("cover_letter.docx", 34000));
        documents.add(new File("notes.txt", 2048));
        
        // Add files to Pictures
        pictures.add(new File("vacation.jpg", 3145728));
        pictures.add(new File("family.png", 2097152));
        
        Directory wallpapers = new Directory("Wallpapers");
        wallpapers.add(new File("abstract.jpg", 1048576));
        wallpapers.add(new File("nature.jpg", 2097152));
        pictures.add(wallpapers);
        
        // Add files to Projects
        Directory javaProject = new Directory("JavaProject");
        javaProject.add(new File("Main.java", 4096));
        javaProject.add(new File("Utils.java", 2048));
        javaProject.add(new File("README.md", 1024));
        
        Directory src = new Directory("src");
        src.add(new File("App.java", 8192));
        src.add(new File("Config.java", 1024));
        javaProject.add(src);
        
        projects.add(javaProject);
        projects.add(new File("todo.txt", 512));
        
        // Add to root
        root.add(documents);
        root.add(pictures);
        root.add(projects);
        root.add(new File("system.log", 10240));
    }
    
    public void showFileSystem() {
        System.out.println("=== File System Structure ===");
        root.display("");
    }
    
    public void showStatistics() {
        System.out.println("\n=== File System Statistics ===");
        System.out.println("Total size: " + formatSize(root.getSize()));
        System.out.println("Total items: " + countItems(root));
    }
    
    private int countItems(Directory dir) {
        int count = 0;
        for (FileSystemComponent component : dir.getChildren()) {
            count++;
            if (component instanceof Directory) {
                count += countItems((Directory) component);
            }
        }
        return count;
    }
    
    public void searchFiles(String keyword) {
        System.out.println("\n=== Searching for '" + keyword + "' ===");
        searchRecursive(root, keyword, "");
    }
    
    private void searchRecursive(FileSystemComponent component, String keyword, String path) {
        String currentPath = path + "/" + component.name;
        if (component.name.toLowerCase().contains(keyword.toLowerCase())) {
            System.out.println("Found: " + currentPath);
        }
        
        if (component instanceof Directory) {
            for (FileSystemComponent child : ((Directory) component).getChildren()) {
                searchRecursive(child, keyword, currentPath);
            }
        }
    }
    
    private String formatSize(long bytes) {
        if (bytes < 1024) return bytes + " B";
        if (bytes < 1024 * 1024) return (bytes / 1024) + " KB";
        return String.format("%.2f MB", bytes / (1024.0 * 1024.0));
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        FileExplorer explorer = new FileExplorer();
        
        explorer.showFileSystem();
        explorer.showStatistics();
        explorer.searchFiles("java");
    }
}
```

**Why Composite?**
- Treats individual objects and compositions uniformly
- Represents tree structures naturally
- Simplifies client code - no need to distinguish between files and directories
- Easy to add new component types
- Recursive operations are straightforward

---

### 9. Decorator Pattern

**Question:** Build a coffee ordering system where customers can add various condiments (milk, sugar, whipped cream, etc.) to their base coffee. Each addition should modify both the description and the cost dynamically.

**Solution:**

```java
// Component interface
interface Coffee {
    String getDescription();
    double getCost();
}

// Concrete Component - Base coffees
class Espresso implements Coffee {
    @Override
    public String getDescription() {
        return "Espresso";
    }
    
    @Override
    public double getCost() {
        return 2.00;
    }
}

class Cappuccino implements Coffee {
    @Override
    public String getDescription() {
        return "Cappuccino";
    }
    
    @Override
    public double getCost() {
        return 3.50;
    }
}

class Latte implements Coffee {
    @Override
    public String getDescription() {
        return "Latte";
    }
    
    @Override
    public double getCost() {
        return 4.00;
    }
}

// Base Decorator
abstract class CoffeeDecorator implements Coffee {
    protected Coffee coffee;
    
    public CoffeeDecorator(Coffee coffee) {
        this.coffee = coffee;
    }
    
    @Override
    public String getDescription() {
        return coffee.getDescription();
    }
    
    @Override
    public double getCost() {
        return coffee.getCost();
    }
}

// Concrete Decorators
class MilkDecorator extends CoffeeDecorator {
    private String milkType;
    
    public MilkDecorator(Coffee coffee, String milkType) {
        super(coffee);
        this.milkType = milkType;
    }
    
    @Override
    public String getDescription() {
        return coffee.getDescription() + ", " + milkType + " Milk";
    }
    
    @Override
    public double getCost() {
        double milkCost = 0.5;
        if (milkType.equals("Soy") || milkType.equals("Almond")) {
            milkCost = 0.7;
        }
        return coffee.getCost() + milkCost;
    }
}

class SugarDecorator extends CoffeeDecorator {
    private int teaspoons;
    
    public SugarDecorator(Coffee coffee, int teaspoons) {
        super(coffee);
        this.teaspoons = teaspoons;
    }
    
    @Override
    public String getDescription() {
        return coffee.getDescription() + ", " + teaspoons + " Sugar";
    }
    
    @Override
    public double getCost() {
        return coffee.getCost() + (0.1 * teaspoons);
    }
}

class WhippedCreamDecorator extends CoffeeDecorator {
    public WhippedCreamDecorator(Coffee coffee) {
        super(coffee);
    }
    
    @Override
    public String getDescription() {
        return coffee.getDescription() + ", Whipped Cream";
    }
    
    @Override
    public double getCost() {
        return coffee.getCost() + 0.75;
    }
}

class ChocolateDecorator extends CoffeeDecorator {
    public ChocolateDecorator(Coffee coffee) {
        super(coffee);
    }
    
    @Override
    public String getDescription() {
        return coffee.getDescription() + ", Chocolate";
    }
    
    @Override
    public double getCost() {
        return coffee.getCost() + 0.6;
    }
}

class CaramelDecorator extends CoffeeDecorator {
    public CaramelDecorator(Coffee coffee) {
        super(coffee);
    }
    
    @Override
    public String getDescription() {
        return coffee.getDescription() + ", Caramel Syrup";
    }
    
    @Override
    public double getCost() {
        return coffee.getCost() + 0.8;
    }
}

class ExtraShotDecorator extends CoffeeDecorator {
    public ExtraShotDecorator(Coffee coffee) {
        super(coffee);
    }
    
    @Override
    public String getDescription() {
        return coffee.getDescription() + ", Extra Shot";
    }
    
    @Override
    public double getCost() {
        return coffee.getCost() + 1.2;
    }
}

// Size Decorator
class SizeDecorator extends CoffeeDecorator {
    private String size;
    
    public SizeDecorator(Coffee coffee, String size) {
        super(coffee);
        this.size = size;
    }
    
    @Override
    public String getDescription() {
        return size + " " + coffee.getDescription();
    }
    
    @Override
    public double getCost() {
        double multiplier = 1.0;
        switch (size) {
            case "Small": multiplier = 0.8; break;
            case "Medium": multiplier = 1.0; break;
            case "Large": multiplier = 1.3; break;
            case "Venti": multiplier = 1.5; break;
        }
        return coffee.getCost() * multiplier;
    }
}

// Coffee Shop
class CoffeeShop {
    public void displayReceipt(Coffee coffee) {
        System.out.println("☕ " + coffee.getDescription());
        System.out.println("Total: $" + String.format("%.2f", coffee.getCost()));
        System.out.println("------------------------");
    }
    
    public Coffee createCustomCoffee() {
        // Simulating a complex custom order
        Coffee coffee = new Latte();
        coffee = new SizeDecorator(coffee, "Large");
        coffee = new MilkDecorator(coffee, "Soy");
        coffee = new ExtraShotDecorator(coffee);
        coffee = new CaramelDecorator(coffee);
        coffee = new WhippedCreamDecorator(coffee);
        return coffee;
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        CoffeeShop shop = new CoffeeShop();
        
        // Simple coffee
        Coffee espresso = new Espresso();
        shop.displayReceipt(espresso);
        
        // Coffee with decorations
        Coffee cappuccino = new Cappuccino();
        cappuccino = new MilkDecorator(cappuccino, "Whole");
        cappuccino = new SugarDecorator(cappuccino, 2);
        shop.displayReceipt(cappuccino);
        
        // Complex order
        Coffee latte = new Latte();
        latte = new SizeDecorator(latte, "Venti");
        latte = new MilkDecorator(latte, "Almond");
        latte = new ChocolateDecorator(latte);
        latte = new WhippedCreamDecorator(latte);
        latte = new ExtraShotDecorator(latte);
        shop.displayReceipt(latte);
        
        // Custom coffee from shop
        System.out.println("=== Today's Special ===");
        Coffee special = shop.createCustomCoffee();
        shop.displayReceipt(special);
    }
}
```

**Why Decorator?**
- Adds responsibilities dynamically without altering structure
- More flexible than static inheritance
- Supports combination of behaviors
- Follows Open/Closed Principle
- Can add/remove decorations at runtime

---

### 10. Facade Pattern

**Question:** Create a home theater system with multiple components (TV, Sound System, DVD Player, Lights, etc.). Provide a simple interface to perform complex operations like "watch movie" which involves multiple steps across different subsystems.

**Solution:**

```java
// Complex subsystem classes
class Television {
    private int channel;
    private int volume;
    private boolean on;
    
    public void turnOn() {
        on = true;
        System.out.println("TV: Turning ON");
    }
    
    public void turnOff() {
        on = false;
        System.out.println("TV: Turning OFF");
    }
    
    public void setInputChannel(int channel) {
        this.channel = channel;
        System.out.println("TV: Setting input to HDMI " + channel);
    }
    
    public void setVolume(int volume) {
        this.volume = volume;
        System.out.println("TV: Volume set to " + volume);
    }
}

class SoundSystem {
    private int volume;
    private String mode;
    private boolean on;
    
    public void turnOn() {
        on = true;
        System.out.println("Sound System: Turning ON");
    }
    
    public void turnOff() {
        on = false;
        System.out.println("Sound System: Turning OFF");
    }
    
    public void setSurroundSound() {
        mode = "Surround";
        System.out.println("Sound System: Surround sound enabled");
    }
    
    public void setStereoMode() {
        mode = "Stereo";
        System.out.println("Sound System: Stereo mode enabled");
    }
    
    public void setVolume(int volume) {
        this.volume = volume;
        System.out.println("Sound System: Volume set to " + volume);
    }
}

class DVDPlayer {
    private String movie;
    private boolean playing;
    
    public void turnOn() {
        System.out.println("DVD Player: Turning ON");
    }
    
    public void turnOff() {
        System.out.println("DVD Player: Turning OFF");
    }
    
    public void insertDVD(String movie) {
        this.movie = movie;
        System.out.println("DVD Player: Inserting '" + movie + "'");
    }
    
    public void play() {
        playing = true;
        System.out.println("DVD Player: Playing '" + movie + "'");
    }
    
    public void pause() {
        playing = false;
        System.out.println("DVD Player: Paused");
    }
    
    public void stop() {
        playing = false;
        System.out.println("DVD Player: Stopped");
    }
    
    public void ejectDVD() {
        System.out.println("DVD Player: Ejecting DVD");
        movie = null;
    }
}

class StreamingService {
    private String service;
    private boolean connected;
    
    public void connect(String service) {
        this.service = service;
        connected = true;
        System.out.println("Streaming: Connected to " + service);
    }
    
    public void disconnect() {
        connected = false;
        System.out.println("Streaming: Disconnected from " + service);
        service = null;
    }
    
    public void selectMovie(String movie) {
        System.out.println("Streaming: Selected '" + movie + "' on " + service);
    }
    
    public void play() {
        System.out.println("Streaming: Playing movie");
    }
    
    public void pause() {
        System.out.println("Streaming: Paused");
    }
}

class Lights {
    private int brightness;
    
    public void on() {
        brightness = 100;
        System.out.println("Lights: ON at full brightness");
    }
    
    public void off() {
        brightness = 0;
        System.out.println("Lights: OFF");
    }
    
    public void dim(int level) {
        brightness = level;
        System.out.println("Lights: Dimmed to " + level + "%");
    }
}

class Popcorn {
    public void turnOn() {
        System.out.println("Popcorn Maker: Turning ON");
    }
    
    public void turnOff() {
        System.out.println("Popcorn Maker: Turning OFF");
    }
    
    public void pop() {
        System.out.println("Popcorn Maker: Popping corn... 🍿");
    }
}

class AirConditioner {
    private int temperature;
    
    public void turnOn() {
        System.out.println("AC: Turning ON");
    }
    
    public void turnOff() {
        System.out.println("AC: Turning OFF");
    }
    
    public void setTemperature(int temp) {
        temperature = temp;
        System.out.println("AC: Temperature set to " + temp + "°C");
    }
}

// Facade
class HomeTheaterFacade {
    private Television tv;
    private SoundSystem soundSystem;
    private DVDPlayer dvdPlayer;
    private StreamingService streaming;
    private Lights lights;
    private Popcorn popcorn;
    private AirConditioner ac;
    
    public HomeTheaterFacade() {
        this.tv = new Television();
        this.soundSystem = new SoundSystem();
        this.dvdPlayer = new DVDPlayer();
        this.streaming = new StreamingService();
        this.lights = new Lights();
        this.popcorn = new Popcorn();
        this.ac = new AirConditioner();
    }
    
    public void watchMovie(String movie) {
        System.out.println("\n🎬 Preparing to watch '" + movie + "'...\n");
        
        // Prepare environment
        lights.dim(10);
        ac.turnOn();
        ac.setTemperature(22);
        
        // Prepare snacks
        popcorn.turnOn();
        popcorn.pop();
        
        // Setup audio/video
        tv.turnOn();
        tv.setInputChannel(1);
        tv.setVolume(20);
        
        soundSystem.turnOn();
        soundSystem.setSurroundSound();
        soundSystem.setVolume(50);
        
        dvdPlayer.turnOn();
        dvdPlayer.insertDVD(movie);
        dvdPlayer.play();
        
        System.out.println("\n✅ Everything is ready! Enjoy your movie! 🎬\n");
    }
    
    public void streamMovie(String movie, String service) {
        System.out.println("\n📺 Preparing to stream '" + movie + "'...\n");
        
        // Prepare environment
        lights.dim(20);
        ac.turnOn();
        ac.setTemperature(23);
        
        // Setup streaming
        tv.turnOn();
        tv.setInputChannel(2);
        tv.setVolume(25);
        
        soundSystem.turnOn();
        soundSystem.setSurroundSound();
        soundSystem.setVolume(45);
        
        streaming.connect(service);
        streaming.selectMovie(movie);
        streaming.play();
        
        System.out.println("\n✅ Streaming started! Enjoy! 📺\n");
    }
    
    public void endMovie() {
        System.out.println("\n🛑 Shutting down home theater...\n");
        
        dvdPlayer.stop();
        dvdPlayer.ejectDVD();
        dvdPlayer.turnOff();
        
        streaming.disconnect();
        
        tv.turnOff();
        soundSystem.turnOff();
        
        lights.on();
        popcorn.turnOff();
        ac.turnOff();
        
        System.out.println("\n✅ Everything is turned off. Good night! 🌙\n");
    }
    
    public void pauseMovie() {
        System.out.println("\n⏸ Pausing movie...\n");
        dvdPlayer.pause();
        lights.dim(50);
        System.out.println("\n✅ Movie paused\n");
    }
    
    public void resumeMovie() {
        System.out.println("\n▶️ Resuming movie...\n");
        lights.dim(10);
        dvdPlayer.play();
        System.out.println("\n✅ Movie resumed\n");
    }
    
    // Gaming mode
    public void gameMode() {
        System.out.println("\n🎮 Setting up gaming mode...\n");
        
        tv.turnOn();
        tv.setInputChannel(3);
        tv.setVolume(30);
        
        soundSystem.turnOn();
        soundSystem.setStereoMode();
        soundSystem.setVolume(60);
        
        lights.dim(40);
        ac.setTemperature(21);
        
        System.out.println("\n✅ Gaming mode activated! 🎮\n");
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        HomeTheaterFacade homeTheater = new HomeTheaterFacade();