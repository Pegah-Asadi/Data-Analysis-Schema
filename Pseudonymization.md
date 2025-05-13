## SQL general syntax for pseudonymization

Why it’s generic:  
- Uses TO_HEX(SHA256()), available in every major warehouse

```SQL
/* Pseudonymize a primary‑key column with SHA‑256 */
SELECT
  TO_HEX(SHA256(CAST(<id_column> AS STRING))) AS <id_column>_hash,
  <date_column>,
  <status_column>
FROM `<project>.<dataset>.<table>`;
```


## Pyhton general syntax for pseudonymization

```Python
import hashlib
import pandas as pd

def sha256_hex(value: str) -> str:
    """Return a SHA‑256 hex digest for any string/number."""
    return hashlib.sha256(str(value).encode()).hexdigest()

# df is your DataFrame
df["user_hash"] = df["user_id"].apply(sha256_hex)
```

## 🔐 Row‑Level Security (BigQuery example)

#### Generic policy template
```SQL
CREATE ROW ACCESS POLICY <policy_name>
ON `<project>.<dataset>.<table>`
GRANT TO ( '<principal_1>', '<principal_2>', … )  -- users, groups, or service accounts
FILTER USING ( <boolean_condition> );
```

#### Common “user‑sees‑only‑their‑records” pattern
```SQL
CREATE ROW ACCESS POLICY user_specific_filter
ON `<project>.<dataset>.customer_data`
GRANT TO ( 'group:admins@example.com', 'allAuthenticatedUsers' )
FILTER USING (
      account_manager_email = SESSION_USER()
  OR  SESSION_USER() IN ('admin1@example.com', 'admin2@example.com')
);
```
Why it’s generic:  
- <principal> accepts any BigQuery‑supported identity: user:, group:, or serviceAccount:.  
- SESSION_USER() works out of the box without extra parameters.  
- Condition can be swapped for tenant IDs, region filters, etc.
