#Requirements Covered Here:
Companies → Multiple warehouses
Products → Stored in multiple warehouses
Track stock changes
Supplier support
Bundle products


#Database Schema (DDL SQL)

CREATE TABLE companies (
id BIGINT PRIMARY KEY AUTO_INCREMENT,
name VARCHAR(150) NOT NULL,
industry VARCHAR(120),
created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE warehouses (
id BIGINT PRIMARY KEY AUTO_INCREMENT,
company_id BIGINT NOT NULL,
name VARCHAR(120),
location VARCHAR(255),
FOREIGN KEY(company_id) REFERENCES companies(id)
);

CREATE TABLE products (
id BIGINT PRIMARY KEY AUTO_INCREMENT,
name VARCHAR(200) NOT NULL,
sku VARCHAR(100) UNIQUE NOT NULL,
type ENUM('NORMAL','BUNDLE') DEFAULT 'NORMAL',
price DECIMAL(10,2),
threshold INT DEFAULT 10
);

CREATE TABLE inventory (
id BIGINT PRIMARY KEY AUTO_INCREMENT,
product_id BIGINT,
warehouse_id BIGINT,
quantity INT DEFAULT 0,
UNIQUE(product_id, warehouse_id),
FOREIGN KEY(product_id) REFERENCES products(id),
FOREIGN KEY(warehouse_id) REFERENCES warehouses(id)
);

CREATE TABLE inventory_logs (
id BIGINT PRIMARY KEY AUTO_INCREMENT,
product_id BIGINT,
warehouse_id BIGINT,
change_qty INT,
reason VARCHAR(255),
created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE suppliers (
id BIGINT PRIMARY KEY AUTO_INCREMENT,
name VARCHAR(200),
email VARCHAR(200),
phone VARCHAR(50)
);

CREATE TABLE product_suppliers (
product_id BIGINT,
supplier_id BIGINT,
lead_time_days INT,
PRIMARY KEY(product_id, supplier_id),
FOREIGN KEY(product_id) REFERENCES products(id),
FOREIGN KEY(supplier_id) REFERENCES suppliers(id)
);

CREATE TABLE bundle_items (
bundle_id BIGINT,
child_product_id BIGINT,
quantity_required INT,
PRIMARY KEY(bundle_id, child_product_id)
);