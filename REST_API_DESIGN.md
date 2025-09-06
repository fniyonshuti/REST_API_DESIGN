# InventoryPro - API Architecture Planning

## Executive Summary
The InventoryPro API is designed as a RESTful service to support inventory management for African retail businesses.  
Its goal is to provide a standardized, scalable, and intuitive API that allows multiple clients (web, mobile, and third-party systems) to manage products, categories, suppliers, stores, and stock levels.  
The design follows REST principles, using HTTP methods consistently, predictable URI patterns, and meaningful status codes.  
This API will help businesses streamline stock tracking, supplier coordination, and sales monitoring across multiple store locations.

---

## Business Domain Analysis

### Primary Business Entities
1. **Product** – Goods available for sale, each belonging to a category and associated with one or more suppliers.  
2. **Category** – Logical grouping of products (e.g., Electronics, Clothing, Food).  
3. **Supplier** – Providers of products; one supplier can supply multiple products, and products can come from multiple suppliers.  
4. **Store** – Physical retail location; each store manages its own stock levels.  
5. **Stock** – Junction resource linking products and stores, tracking available quantity at each location.

### Key Relationships
- **Product → Category**: Many-to-One (a product belongs to one category).  
- **Product ↔ Supplier**: Many-to-Many (a product can have multiple suppliers).  
- **Product ↔ Store (via Stock)**: Many-to-Many (products are stocked across multiple stores).  

### Critical Business Operations
- Add, update, delete, and list products, categories, suppliers, and stores.  
- Manage stock levels per product per store.  
- Associate products with suppliers and categories.  
- Search/filter products by category, supplier, or stock availability.  
- Bulk stock updates for efficient inventory adjustments.  

### Data Attributes by Entity
**Product**: id, name, description, sku, price, category_id, created_at, updated_at  
**Category**: id, name, description, created_at, updated_at  
**Supplier**: id, name, contact_info, address, created_at, updated_at  
**Store**: id, name, location, manager_name, created_at, updated_at  
**Stock**: id, product_id, store_id, quantity, last_updated  

---

## Resource Specifications

### Product
- **Purpose:** Represents individual goods sold.  
- **Attributes:**  
  - id (UUID)  
  - name (string, required)  
  - description (string)  
  - sku (string, unique)  
  - price (decimal, required)  
  - category_id (UUID, required)  
  - created_at (datetime)  
  - updated_at (datetime)  
- **Relationships:**  
  - Belongs to Category  
  - Many-to-Many with Supplier  
  - Many-to-Many with Store (via Stock)  
- **Business Rules:**  
  - Price must be non-negative.  
  - SKU must be unique.  

### Category
- **Purpose:** Groups products under a common type.  
- **Attributes:** id, name, description, created_at, updated_at  
- **Relationships:** One-to-Many with Products  
- **Business Rules:** Name must be unique.  

### Supplier
- **Purpose:** Represents product suppliers.  
- **Attributes:** id, name, contact_info, address, created_at, updated_at  
- **Relationships:** Many-to-Many with Products  
- **Business Rules:** Supplier name must be unique.  

### Store
- **Purpose:** Represents physical retail locations.  
- **Attributes:** id, name, location, manager_name, created_at, updated_at  
- **Relationships:** Many-to-Many with Products via Stock  
- **Business Rules:** Store name must be unique.  

### Stock
- **Purpose:** Tracks product quantity per store.  
- **Attributes:** id, product_id, store_id, quantity, last_updated  
- **Relationships:** Many-to-One with Product, Many-to-One with Store  
- **Business Rules:** Quantity must be ≥ 0.  

---

## Endpoint Documentation

| Resource  | Operation               | HTTP Method | URI                          | Request Body                        | Success Response (200/201)                   | Error Responses (400/404/500) |
|-----------|--------------------------|-------------|------------------------------|-------------------------------------|----------------------------------------------|--------------------------------|
| Product   | Create Product           | POST        | /api/products                | { name, sku, price, category_id }   | { id, name, sku, price, category_id }        | 400 invalid, 500 server error |
| Product   | Get All Products         | GET         | /api/products                | –                                   | [{ id, name, price }]                        | 500 server error |
| Product   | Get Product by ID        | GET         | /api/products/{id}           | –                                   | { id, name, price, category_id }             | 404 not found, 500 server error |
| Product   | Update Product           | PUT         | /api/products/{id}           | { name, price }                     | { id, name, price, category_id }             | 400 invalid, 404 not found |
| Product   | Delete Product           | DELETE      | /api/products/{id}           | –                                   | 204 No Content                               | 404 not found |
| Category  | CRUD Operations          | GET/POST/...| /api/categories...            | –                                   | Similar structure to Products                | – |
| Supplier  | CRUD Operations          | GET/POST/...| /api/suppliers...             | –                                   | Similar structure to Products                | – |
| Store     | CRUD Operations          | GET/POST/...| /api/stores...                | –                                   | Similar structure to Products                | – |
| Stock     | Manage Stock Levels      | POST/PUT    | /api/stores/{storeId}/stock  | { product_id, quantity }            | { store_id, product_id, quantity }           | 400 invalid, 404 not found |

---

## Advanced Features

1. **Association Endpoints**  
   - `/api/categories/{id}/products` → Get all products in a category.  
   - `/api/suppliers/{id}/products` → Get all products supplied by a supplier.  

2. **Bulk Operations**  
   - `/api/stores/{id}/stock/bulk` → Update stock levels for multiple products in one request.  

3. **Search & Filtering**  
   - `/api/products?category=electronics&price_lt=100`  
   - `/api/stock?store=1&low_stock=true`  

4. **Domain-Specific Operations**  
   - `/api/reports/stock/low` → List products below threshold stock levels.  

---

## Design Rationale

- **REST Compliance:** Each resource is represented as a noun with predictable, hierarchical URIs.  
- **Scalability:** Stock is modeled as a separate resource to handle many-to-many relationships efficiently.  
- **Clarity:** CRUD operations are standardized across all resources.  
- **Business Relevance:** The design supports real-world retail operations like supplier management, multi-store stock tracking, and reporting.  
- **Error Handling:** Consistent use of status codes (400 for validation errors, 404 for missing resources, 500 for server issues).  

---
