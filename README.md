# ByteMarket-Analytics: Hardware & Electronics Relational Engine

A PostgreSQL-driven relational database project detailing a hardware manufacturing ecosystem (including PCs, Laptops, and Printers). This repository utilizes Python (`SQLAlchemy`, `pandas`) to connect to a hosted Neon Serverless Postgres instance, execute advanced analytical queries, handle schema inspection, and debug data structural constraints.

The design pattern matches the classic "Computer" database benchmark originally popularized in foundational database systems courses.

---

## 📊 Database Architecture

The data layout maps products to their technical specifications across four primary entities:

* **`product`**: High-level catalog mapping hardware manufacturers (`maker`) to specific `model` entries and product classes (`pc`, `laptop`, `printer`).
* **`pc`**: Micro-specifications of desktop systems including CPU `speed`, `ram`, `hd` capacity, optical drive type, and `price`.
* **`laptop`**: Technical profiles of portable systems including `speed`, `ram`, `hd`, screen `size`, and `price`.
* **`printer`**: Equipment specifications for printing hardware documenting output `color` status, engine `type` (laser/inkjet), and `price`.

---

## 🛠️ Tech Stack & System Requirements

* **Language Platform:** Python 3.12+
* **Database Platform:** PostgreSQL (Hosted via Neon Serverless Engine)
* **Core Drivers & Analytics Packages:**
    * `SQLAlchemy` (Python SQL Toolkit and Object Relational Mapper)
    * `psycopg2-binary` (PostgreSQL database adapter)
    * `pandas` (High-performance data frame analysis)

To install the required workspace ecosystem, execute:
```bash
pip install psycopg2-binary SQLAlchemy pandas





🚀 Key Queries & Analytical Profiles
The analysis suite exercises complex SQL constructs, including subqueries, window functions, and boolean aggregations:

1. Market Efficiency: Unit Cost per Megabyte of RAM
Evaluates hardware pricing metrics by normalizing desktop pricing against total RAM capacity to find the highest value builds.

SQL
SELECT p.maker, pc.model, pc.price, pc.ram,
       ROUND(pc.price::numeric / pc.ram, 2) AS price_per_mb
FROM product p
JOIN pc ON p.model = pc.model
ORDER BY price_per_mb ASC;
2. Supplier Segmentation: Exclusive Portfolio Identification
Leverages the robust PostgreSQL aggregation function BOOL_AND() inside a HAVING clause to mathematically isolate manufacturers whose production catalogs strictly contain laptops.

SQL
SELECT maker
FROM product
GROUP BY maker
HAVING BOOL_AND(type = 'laptop');
3. Rank Partitioning: Top-Tier Price Leaders
Utilizes advanced analytical windowing (RANK() OVER (PARTITION BY...)) to traverse maker subgroups and extract the exact entry-level laptop price tier for every distinct manufacturing firm.

SQL
SELECT maker, model, price
FROM (
  SELECT p.maker, l.model, l.price,
         RANK() OVER (PARTITION BY p.maker ORDER BY l.price ASC) AS rnk
  FROM product p
  JOIN laptop l ON p.model = l.model
) sub
WHERE rnk = 1
ORDER BY maker;
🔍 Debugging Case Study: Schema Discrepancy Identification
During execution, an analytical aggregation query tracking product diversity flagged an error:

Plaintext
UndefinedColumn: column "category" does not exist
LINE 2: SELECT category, COUNT(*) AS total_products
Resolution Pipeline
To resolve this structural mismatch, automated schema verification was implemented to query the database's information_schema.columns definition:

SQL
SELECT column_name, data_type
FROM information_schema.columns
WHERE table_name = 'product';
Diagnostic Discovery: The system profile determined that the operational table utilized the column name type instead of the anticipated category parameter. The functional query was modified to match the production schema data dictionary:

SQL
SELECT type, COUNT(*) AS total_products
FROM product
GROUP BY type
ORDER BY total_products DESC;
