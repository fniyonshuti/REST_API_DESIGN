# InventoryPro - REST API Design

## Executive Summary
This document presents the complete API architecture for **InventoryPro**, an inventory management system tailored for African retail businesses. The API is designed following REST principles to provide clear, predictable, and scalable endpoints for managing products, categories, suppliers, and stock levels across multiple store locations. The design emphasizes simplicity, reusability, and real-world applicability, enabling seamless integration with web and mobile clients.

---

## Business Domain Analysis

### Primary Business Entities
1. **Product** – Represents an item sold by the business.
2. **Category** – Groups products into logical collections.
3. **Supplier** – Provides products to the business.
4. **Store** – Represents a physical retail location.
5. **Stock** – Tracks product availability per store.

### Key Relationships
- A **Product** belongs to one **Category** (many-to-one).
- A **Product** can be supplied by multiple **Suppliers** (many-to-many).
- A **Product** is stocked in multiple **Stores** via **Stock** (many-to-many).
- Each **Store** maintains its own **Stock** records for each product.

### Critical Business Operations
- CRUD operations on products, categories, suppliers, and stores.
- Tracking and updating stock levels per store.
- Associating products with suppliers and categories.
- Querying inventory by store or supplier.
- Searching products by name, category, or availability.

### Data Attributes per Entity
- **Product**: id, name, description, category_id, supplier_ids, price, created_at, updated_at  
- **Category**: id, name, description, created_at, updated_at  
- **Supplier**: id, name, contact_info, created_at, updated_at  
- **Store**: id, name, location, created_at, updated_at  
- **Stock**: id, product_id, store_id, quantity, last_updated  

---

## Resource Specifications

### Product
- **Description:** Represents items available for sale.  
- **Attributes:** id (UUID), name (string), description (string), category_id (UUID), price (decimal), created_at (datetime), updated_at (datetime)  
- **Relationships:** Belongs to Category, has many Suppliers, stocked in many Stores.  
- **Constraints:** Name required, price ≥ 0.  

### Category
- **Description:** Organizes products into logical groups.  
- **Attributes:** id (UUID), name (string), description (string), created_at (datetime), updated_at (datetime)  
- **Relationships:** Has many Products.  
- **Constraints:** Name must be unique.  

### Supplier
- **Description:** Represents providers of products.  
- **Attributes:** id (UUID), name (string), contact_info (string), created_at (datetime), updated_at (datetime)  
- **Relationships:** Supplies many Products.  
- **Constraints:** Name required.  

### Store
- **Description:** Represents a retail location.  
- **Attributes:** id (UUID), name (string), location (string), created_at (datetime), updated_at (datetime)  
- **Relationships:** Stocks many Products via Stock.  
- **Constraints:** Name required.  

### Stock
- **Description:** Tracks product availability at stores.  
- **Attributes:** id (UUID), product_id (UUID), store_id (UUID), quantity (integer), last_updated (datetime)  
- **Relationships:** Belongs to Product and Store.  
- **Constraints:** Quantity ≥ 0.  

---

## Endpoint Documentation

| Resource  | Operation        | HTTP Method | URI | Request Body | Success Response | Error Responses |
|-----------|------------------|-------------|-----|--------------|------------------|----------------|
| Product   | Create           | POST        | /products | {name, description, category_id, price} | 201 Created (Product JSON) | 400 Bad Request |
| Product   | Read (all)       | GET         | /products | N/A | 200 OK (List of Products) | 404 Not Found |
| Product   | Read (one)       | GET         | /products/{id} | N/A | 200 OK (Product JSON) | 404 Not Found |
| Product   | Update           | PUT         | /products/{id} | {fields to update} | 200 OK (Updated Product) | 400 Bad Request, 404 Not Found |
| Product   | Delete           | DELETE      | /products/{id} | N/A | 204 No Content | 404 Not Found |
| Category  | CRUD             | Similar endpoints under `/categories` | | | | |
| Supplier  | CRUD             | Similar endpoints under `/suppliers` | | | | |
| Store     | CRUD             | Similar endpoints under `/stores` | | | | |
| Stock     | Update quantity  | PATCH       | /stocks/{id} | {quantity} | 200 OK (Updated Stock) | 400 Bad Request, 404 Not Found |

**Filtering & Pagination**  
- `/products?category_id={id}` → Products by category  
- `/products?supplier_id={id}` → Products by supplier  
- `/stocks?store_id={id}` → Stock by store  
- Pagination with `?page=1&limit=20`  

---

## Advanced Features

1. **Resource Association**
   - `/categories/{id}/products` → Get all products in a category  
   - `/suppliers/{id}/products` → Get all products from a supplier  
   - `/stores/{id}/stocks` → Get stock levels at a store  

2. **Bulk Operations**
   - `POST /stocks/bulk-update` → Update multiple stock records in one request  

3. **Search**
   - `/products/search?query=laptop` → Search by product name or description  

4. **Domain-Specific Operations**
   - `/stocks/low` → Retrieve products below threshold levels for restocking  
   - `/suppliers/{id}/deliveries` → Planned endpoint for tracking deliveries  

---

## Design Rationale

The design follows **REST principles** to ensure clarity and consistency:  
- **Resources** are nouns (`/products`, `/categories`) with predictable CRUD operations.  
- **Relationships** are exposed through sub-resources (`/categories/{id}/products`).  
- **Status codes** provide meaningful feedback (`201 Created`, `204 No Content`, `400 Bad Request`).  
- **Filtering and pagination** allow efficient data access.  
- **Bulk operations** improve efficiency in real-world retail scenarios.  
- **Domain-specific endpoints** (e.g., low stock alerts) reflect actual business needs.  

This architecture provides a strong foundation for scalable development, ensuring InventoryPro can grow to meet the diverse needs of African retail businesses.

---
