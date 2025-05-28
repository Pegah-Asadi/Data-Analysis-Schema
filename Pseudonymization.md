# 01 | 🔐 Pseudonymization & Row-Level Security

Many shops strip or hash identifiers before analysis.  
The pattern below uses **SHA-256 → hex**—portable across BigQuery, Snowflake, Databricks, Redshift, Postgres.

---

## 1. SQL template (copy-paste friendly)

```sql
/* ===========================================
   GENERIC PSEUDONYMIZATION EXAMPLE
   ===========================================
   • Works in BigQuery, Snowflake, Databricks SQL
   • Works in Redshift/Postgres with tiny tweaks  ➜ see bullets below
   • Replace the placeholders in <> with real names
   ------------------------------------------- */

WITH src AS (
  SELECT
      user_id,                -- your primary key (PII)
      signup_date,
      status
  FROM   <project>.<dataset>.<table>
)

SELECT
    -- 1️⃣ Hash the PK so analysts never see raw IDs
    TO_HEX(SHA256(CAST(user_id AS STRING))) AS user_id_hash,

    -- 2️⃣ Pass non-PII columns through untouched
    signup_date,
    status

FROM src;
```

### 🗒️ Warehouse tweaks in one glance:

| Warehouse                  | Function                                 | One-liner patch                                      |
| -------------------------- | ---------------------------------------- | ---------------------------------------------------- |
| **Redshift**               | `ENCODE(sha256(to_utf8(col)), 'hex')`    | Replace `TO_HEX(SHA256())` with the left expression. |
| **Snowflake**              | `SHA2(col, 256)` or `SHA2_HEX(col, 256)` | Either works; append `::STRING` if col isn’t text.   |
| **PostgreSQL ≥ 13**        | `encode(digest(col, 'sha256'), 'hex')`   | Need the `pgcrypto` extension.                       |
| **Databricks SQL / Spark** | `hex(sha2(col, 256))`                    | Identical to Snowflake.                              |

#### Pro tip:
Add a salt column and concatenate it with the PK if you need deterministic, cross-dataset joins later.<br><br><br>


## Python template (pandas)

```Python
import hashlib
import pandas as pd

def sha256_hex(val) -> str:
    """
    Return a SHA-256 hex digest for any scalar.
    - Handles numbers, strings, bytes, NaN/None.
    - Converts None/NaN to empty string for stability.
    """
    if pd.isna(val):
        val = ""
    return hashlib.sha256(str(val).encode("utf-8")).hexdigest()

# --------------------------------------------------------------------
#  Load your data
#  df = pd.read_csv("path/to/file.csv")  # or any reader
# --------------------------------------------------------------------
df["user_id_hash"] = df["user_id"].apply(sha256_hex)

# Optionally release memory / reduce risk
del df["user_id"]        # or df = df.drop(columns="user_id")
```

## 🔐 Row-Level Security (BigQuery example)

BigQuery’s row-access policies hide rows a user shouldn’t see.

```SQL
-- Generic policy template ------------------------------------------
CREATE ROW ACCESS POLICY <policy_name>
ON   `<project>.<dataset>.<table>`
GRANT TO  ( 'group:data_analysts@example.com',
            'serviceAccount:etl@appspot.gserviceaccount.com' )
FILTER USING ( <boolean_condition> );

-- “Each user sees only their own rows” ------------------------------
CREATE ROW ACCESS POLICY rls_user_filter
ON   `<project>.<dataset>.customer_data`
GRANT TO ( 'group:admins@example.com', 'allAuthenticatedUsers' )
FILTER USING (
      account_manager_email = SESSION_USER()
   OR SESSION_USER() IN ('admin1@example.com', 'admin2@example.com')
);
```

<details> <summary>🎛️ No native RLS in your warehouse?</summary>

Create a secure view that wraps the same filter.
```SQL
CREATE OR REPLACE VIEW analytics.v_customer_data AS
SELECT * FROM raw.customer_data
WHERE account_manager_email = current_user();
```
</details>
