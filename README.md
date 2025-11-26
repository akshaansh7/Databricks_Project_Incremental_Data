# 🚀 Azure Databricks End-to-End Data Engineering Project (2025)

This project demonstrates how to build a **complete enterprise-grade data pipeline** using:

- Azure Databricks  
- Azure Data Lake Storage Gen2 (ADLS)  
- Unity Catalog Metastore  
- PySpark  
- Delta Lake  
- Bronze → Silver → Gold architecture  
- Auto Loader for incremental ingestion  

This README covers **EVERYTHING**, from cloud setup → connections → ingestion → transformations → optimizations → orchestration.

---

# 🏗️ 1. Architecture

```
Raw Data → ADLS (raw)  
      ↓  
Auto Loader (Incremental Ingestion)  
      ↓  
Bronze Delta Tables  
      ↓  
Cleaning / Standardization  
      ↓  
Silver Delta Tables  
      ↓  
Aggregations / KPIs  
      ↓  
Gold Delta Tables  
      ↓  
Power BI / Analytics
```

---

# 🔧 2. Prerequisites

- Azure Subscription  
- Azure Active Directory  
- App Registration (Service Principal)  
- ADLS Gen2 Storage Account  
- Azure Databricks Workspace  
- Unity Catalog Enabled  
- Basic PySpark knowledge  

---

# 🗄️ 3. ADLS Gen2 Setup

### **Create Storage Account**

1. Azure Portal → Storage Accounts → Create  
2. Enable:
   - Hierarchical Namespace = **ON**  
3. Containers to create:

```
raw
bronze
silver
gold
checkpoint
```

These containers represent Lakehouse zones.

---

# 🔐 4. Create Service Principal (for authentication)

1. Azure Portal → **Azure Active Directory**  
2. App Registrations → New Registration  
3. Copy:
   - **Client ID**
   - **Tenant ID**
4. Certificates & Secrets → New Client Secret  

### Assign RBAC on Storage:
Storage Account → IAM → Add Role Assignment

Give the SP:

- **Storage Blob Data Contributor**

---

# 🔑 5. Store Credentials in Databricks Secret Scope

```python
dbutils.secrets.put("sp_scope", "client_id", "<client-id>")
dbutils.secrets.put("sp_scope", "client_secret", "<client-secret>")
dbutils.secrets.put("sp_scope", "tenant_id", "<tenant-id>")
```

---

# 🔗 6. Mount ADLS to Databricks

```python
configs = {
  "fs.azure.account.auth.type": "OAuth",
  "fs.azure.account.oauth.provider.type": "org.apache.hadoop.fs.azurebfs.oauth2.ClientCredsTokenProvider",
  "fs.azure.account.oauth2.client.id": dbutils.secrets.get("sp_scope","client_id"),
  "fs.azure.account.oauth2.client.secret": dbutils.secrets.get("sp_scope","client_secret"),
  "fs.azure.account.oauth2.client.endpoint":
    "https://login.microsoftonline.com/" + dbutils.secrets.get("sp_scope","tenant_id") + "/oauth2/token"
}

dbutils.fs.mount(
  source = "abfss://raw@<storage-account-name>.dfs.core.windows.net/",
  mount_point = "/mnt/raw",
  extra_configs = configs
)
```

Repeat for `/mnt/bronze`, `/mnt/silver`, `/mnt/gold`, `/mnt/checkpoint`.

---

# 📚 7. Create Unity Catalog Metastore

### **Steps:**

1. Databricks → **Admin Settings**  
2. Click **Data**  
3. Create Metastore  
4. Assign root storage (ADLS path)
5. Attach the metastore to the workspace

---

# 🏛️ 8. Create Catalog, Schemas & Tables

### Create Catalog

```sql
CREATE CATALOG catabricks_cat;
```

### Create Schemas

```sql
CREATE SCHEMA catabricks_cat.bronze;
CREATE SCHEMA catabricks_cat.silver;
CREATE SCHEMA catabricks_cat.gold;
```

---

# 11 Notebooks for Bronze-->Silver-->Gold 

# ⚡ 12. Delta Lake Optimizations

### Optimize Table
```sql
OPTIMIZE catabricks_cat.gold.gold_customers;
```

### Z-Ordering
```sql
OPTIMIZE catabricks_cat.gold.gold_customers ZORDER BY (product_id);
```

### Time Travel
```sql
SELECT * FROM catabricks_cat.gold.gold_customers_metrics VERSION AS OF 3;
```

---

# ⏱️ 13. Schedule as Databricks Job

1. Databricks → **Workflows**  
2. Create Job  
3. Add tasks:

```
bronze_ingestion
silver_transform
gold_aggregation
```

4. Add retries & email alerts  
5. Set schedule (hourly / daily)

---

# 📈 14. Final Output

- Fully automated cloud pipeline  
- Incremental ingestion using Auto Loader  
- Cleaned, optimized data in Silver Layer  
- Aggregated KPIs in Gold Layer  
- Delta optimized for fast BI queries  
- Ready for Power BI, Synapse, or SQL Analytics  

---

# 🎯 15. Learning Outcomes

You will learn:

✔ Azure Databricks setup  
✔ ADLS architecture & mounting  
✔ Unity Catalog governance  
✔ Auto Loader incremental ETL  
✔ Delta Lake best practices  
✔ Bronze/Silver/Gold Lakehouse modeling  
✔ Production-grade orchestration  

---

# 👨‍💻 Author
**Akshaansh Harinkhere**  
Data Engineer | PySpark | Databricks | Azure | Delta Lake  

---

# ⭐ Support  
If you found this helpful, please **star ⭐ the repository**!
