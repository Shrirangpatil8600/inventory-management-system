# Project Guide: Inventory Management System

Welcome! This document is a comprehensive guide to understanding, setting up, and deploying the **Inventory Management System** desktop application. It covers the project architecture, database design, module-level functionalities, runtime setup, and key technical concepts with detailed explanations.

---

## 1. Project Overview

The **Inventory Management System** is a high-performance desktop application built using **JavaFX** and **MySQL**. It is designed to streamline day-to-day warehouse and retail business operations, including product stock control, customer profile management, billing/invoice generation, monthly sales analysis, and supplier purchase logs.

The system features a clean, borderless custom UI with smooth transitions and leverages **JasperReports** for compiling and exporting dynamic PDF invoices and statistical reports.

---

## 2. Technical Stack

*   **GUI Framework**: JavaFX (v11.0.2 / v12.0.1) using FXML layouts for views and CSS stylesheets for presentation.
*   **Programming Language**: Java 11.
*   **Build & Dependency Management**: Apache Maven (configured via `pom.xml`).
*   **Database (Model)**: MySQL Database.
*   **Database Helper**: Singleton database wrapper (`Database.java`) utilizing JDBC connection parameters loaded from classpath resource bundles.
*   **Report Generation**: JasperReports (v6.20.5) for compiling XML templates (`.jrxml`) into printable billing receipts and sales logs.
*   **Additional UI Components**: ControlsFX (for advanced dialogs and auto-suggest search boxes) and FontAwesomeFX (for Vector UI icons).
*   **Packaging Tool**: Maven Shade Plugin to bundle all application assets, XML templates, and dependencies into a single runnable fat JAR.

---

## 3. Directory & File Structure

Here is a guide to the project layout and the role of key files:

```text
inventory-management-system-main/
├── database/                       # Database scripts
│   └── inventory_management_system.sql # MySQL SQL Dump containing schema, tables, and seed data
├── pom.xml                         # Maven project configuration (dependencies, shade plugins)
├── dependency-reduced-pom.xml      # Maven generated transient pom
└── src/                            # Java Source Code
    └── main/
        ├── java/                   # Java Code base
        │   ├── module-info.java    # Module descriptor specifying required packages
        │   └── com/inventorymanagementsystem/
        │       ├── Application.java # Main application class (sets up transparent JavaFX Stage)
        │       ├── LoginController.java # FXML Controller for Login Scene authentication
        │       ├── DashboardController.java # FXML Controller for dashboard menus & metrics
        │       ├── BillsController.java # FXML Controller for generating customer bills
        │       ├── app/
        │       │   └── Launcher.java # Main entry wrapper (used to launch fat JARs safely)
        │       ├── config/
        │       │   └── Database.java # Singleton Database connector class
        │       └── entity/         # Data Models (POJOs)
        │           ├── Product.java, Customer.java, Sales.java, Purchase.java, User.java, Billing.java
        └── resources/              # Project resources and assets
            ├── application.properties # Stores MySQL connection URL, user, and password
            ├── com/inventorymanagementsystem/
            │   ├── login-view.fxml & login.css # View and styling for the login screen
            │   ├── dashboard.fxml & dashboard.css # View and styling for the main dashboard tabs
            │   └── bills.fxml & bills.css # View and styling for generating invoices
            └── jasper-reports/     # JasperReports XML design templates
                ├── Invoice.jrxml   # Printable receipt invoice design template
                ├── customers.jrxml # Customer logs layout
                ├── sales_report.jrxml # Monthly Sales report layout
                ├── purchase_report.jrxml # Supplier Purchase logs layout
                └── invoice_logo.png # Logo image loaded by report templates
```

---

## 4. Module-wise Working

### A. Authentication Module
*   **Window Stage Design**: The application starts by loading `login-view.fxml`. The UI style is configured to `StageStyle.TRANSPARENT` to render a clean, borderless modern layout. To make the window movable, custom mouse listeners are set on the root pane to update the screen coordinates dynamically.
*   **Sign-In Logic**: `LoginController.java` utilizes a prepared query against the `users` table. If credentials match, the current login window is hidden, and `dashboard.fxml` is initialized. The authenticated user is stored globally in `User.name`.

### B. Dashboard Metrics & Analytics
*   The dashboard tab compiles statistical widgets of the business:
    *   **Monthly Sales Revenue**: Aggregates the total revenue earned during the current calendar month.
    *   **Monthly Items Sold**: Computes total quantity sold.
    *   **Low Stock Warnings**: Monitors stock counts in real-time, flashing alert panels when inventory falls below critical levels.
    *   **Graphical Charts**: Renders active LineCharts/BarCharts illustrating monthly sales trends to help managers identify peak hours and products.

### C. Inventory Management
*   Provides full CRUD operations for product inventory:
    *   **Add Product**: Input custom item numbers, groups, initial stock quantities, and base sales prices.
    *   **Update/Delete**: Select items from the table view, alter quantities/pricing, or delete records. 
    *   Implements key validation checks to prevent negative values or duplicate item numbers.

### D. Customer Directory & Logs
*   Tracks client transactions:
    *   Stores name and unique phone numbers.
    *   Allows searching through database logs, modifying contact details, and printing out client logs via JasperReports.

### E. Billing & PDF Invoice Generation
*   **Cart System**: Cashiers can search for items by name or scan barcodes. Clicking add moves the items to the billing cart.
*   **Receipt Export**: Upon clicking "Pay", the transaction is committed to the `sales` table, stock levels are automatically decremented, and the `Invoice.jrxml` template is compiled dynamically using JasperReports to export a PDF receipt featuring customer details, itemized totals, tax computations, and shop addresses.

---

## 5. Database Schema & Tables

Here is the database design compiled for MySQL (`inventory_management_system`):

```sql
CREATE DATABASE inventory_management_system;
USE inventory_management_system;

-- 1. Users Table (System operators)
CREATE TABLE users (
    id INT(11) NOT NULL AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(255) NOT NULL,
    password VARCHAR(255) NOT NULL
);

-- Seed default operator
INSERT INTO users (username, password) VALUES ('admin', 'admin');

-- 2. Customers Table
CREATE TABLE customers (
    id INT(11) NOT NULL AUTO_INCREMENT UNIQUE KEY,
    Name VARCHAR(255) NOT NULL,
    PhoneNumber VARCHAR(10) NOT NULL PRIMARY KEY
);

-- 3. Products Table (Active stock)
CREATE TABLE products (
    id INT(11) NOT NULL AUTO_INCREMENT PRIMARY KEY,
    item_number VARCHAR(255) NOT NULL UNIQUEKEY,
    item_group VARCHAR(255) NOT NULL,
    quantity INT(11) NOT NULL,
    price DOUBLE(50,2) NOT NULL
);

-- 4. Sales Table (Log of client billing purchases)
CREATE TABLE sales (
    id INT(11) NOT NULL AUTO_INCREMENT PRIMARY KEY,
    inv_num VARCHAR(255) NOT NULL,
    cust_id INT(11) NOT NULL,
    price DOUBLE(50,2) NOT NULL,
    quantity INT(11) NOT NULL,
    total_amount DOUBLE(50,2) NOT NULL,
    date VARCHAR(255) NOT NULL,
    item_number VARCHAR(255) NOT NULL,
    FOREIGN KEY (cust_id) REFERENCES customers(id) ON DELETE CASCADE
);

-- 5. Purchase Table (Log of incoming supplier stock shipments)
CREATE TABLE purchase (
    id INT(11) NOT NULL AUTO_INCREMENT PRIMARY KEY,
    invoice VARCHAR(255) NOT NULL,
    `shop and address` VARCHAR(1000) NOT NULL,
    total_items INT(50) NOT NULL,
    total_amount DOUBLE(50,2) NOT NULL,
    date_of_purchase VARCHAR(255) NOT NULL
);

-- 6. Temporary Billing Table (Handles active cart caching)
CREATE TABLE billing (
    item_number VARCHAR(255) NOT NULL,
    quantity INT(25) NOT NULL,
    price DOUBLE(50,2) NOT NULL,
    total_amount DOUBLE(50,2) NOT NULL,
    id INT(11) NOT NULL AUTO_INCREMENT PRIMARY KEY
);
```

---

## 6. How to Set Up the Project

Follow these steps to deploy and run the project locally on your machine.

### Step 1: Install Prerequisites
1.  **Java Development Kit (JDK)**: Install **JDK 11** or **JDK 17**.
2.  **MySQL Server**: Ensure MySQL Server is running locally.
3.  **Maven**: Ensure Maven is configured in your system environment path.

### Step 2: Set Up Database
1.  Open your MySQL command-line or GUI workbench client.
2.  Execute the database dump script [inventory_management_system.sql](file:///c:/Users/Gaurav%20Gawali/Documents/GitHub/inventory-management-system-main/database/inventory_management_system.sql) to create the schema and seed mock values.

### Step 3: Configure Database Settings
Open [application.properties](file:///c:/Users/Gaurav%20Gawali/Documents/GitHub/inventory-management-system-main/src/main/resources/application.properties) and update variables to match your local database settings:
```properties
javafx.jdbc.driver=com.mysql.cj.jdbc.Driver
javafx.datasource.url=jdbc:mysql://127.0.0.1:3306/inventory_management_system
javafx.datasource.username=your_mysql_username
javafx.datasource.password=your_mysql_password
```

### Step 4: Compile & Run the Application
Open your terminal in the directory containing `pom.xml`:

1.  **Resolve dependencies and compile**:
    ```bash
    mvn clean install
    ```
2.  **Run the GUI client**:
    ```bash
    mvn compile exec:java
    ```
3.  **Create Runnable Fat JAR**:
    ```bash
    mvn compile package
    ```
    This generates a runnable jar file inside `shade/inventory-management-system.jar` containing all FXML templates, CSS, graphics, and dependencies.

---

## 7. Technical FAQ & Deep-Dive

This section provides detailed answers to key technical design and operational questions about the system:

### Q1: How does the borderless window design work without standard OS frames?
**Answer**:
> "In `Application.java`, we configure the stage initialization mode to `StageStyle.TRANSPARENT`. This strips the default window frames (minimize, maximize, close buttons and border borders) provided by the operating system.
>
> To let users drag the window, we set mouse event listeners on the root FXML pane:
> ```java
> root.setOnMousePressed((event) -> {
>     x = event.getSceneX();
>     y = event.getSceneY();
> });
> root.setOnMouseDragged((event) -> {
>     stage.setX(event.getScreenX() - x);
>     stage.setY(event.getScreenY() - y);
> });
> ```
> This captures the initial click offset coordinates and moves the application Window stage matching screen movement coordinates."

### Q2: Why is a Launcher class required in `Launcher.java` to start the application?
**Answer**:
> "Java 9 modules introduce strict restrictions. When launching a JavaFX application from a compiled fat JAR, the runtime checks if the launcher class extends `javafx.application.Application`. If it does, and the JavaFX modules are not loaded on the system classpath, the application crashes with a *'JavaFX runtime components are missing'* error.
>
> Exposing a separate main class `Launcher.java` that does **not** inherit from `Application` bypasses this validation, letting Java load all JavaFX jars bundled inside the Maven shaded archive automatically."

### Q3: How does the application integrate JasperReports to compile and export PDF bills?
**Answer**:
> "The project stores report designs as XML templates (`.jrxml` extension) inside `src/main/resources/jasper-reports`. At runtime:
> 1.  The controller load the template input stream:
>     `InputStream stream = getClass().getResourceAsStream("/jasper-reports/Invoice.jrxml");`
> 2.  It compiles the report design:
>     `JasperReport report = JasperCompileManager.compileReport(stream);`
> 3.  It populates the report data by passing the active JDBC connection and parameters (e.g., invoice ID):
>     `JasperPrint print = JasperFillManager.fillReport(report, parameters, connection);`
> 4.  Finally, it prints or exports the document:
>     `JasperViewer.viewReport(print, false);` or exports directly to a PDF file."

### Q4: How is database connection state managed?
**Answer**:
> "Database connection uses the Singleton Pattern implemented in `Database.java`. A private constructor prevents external instantiation, and a static `getInstance()` method provides a global access point.
>
> The `connectDB()` method loads values dynamically from `application.properties` and checks if the connection is active. Maintaining a single open connection reference avoids the overhead of establishing new TCP sockets, optimizing performance on desktop client setups."

---
*Created by Shrirang Patil.*
