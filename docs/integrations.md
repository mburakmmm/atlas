# Ekosistem Entegrasyonları (Ecosystem Integrations)

Atlas, Nox ekosistemindeki diğer kütüphanelerle gevşek bağlı (loosely-coupled), sıfır zorunlu bağımlılık ilkesine dayanan entegrasyon adaptörleri sunar.

---

## 1. İlişkisel Veritabanı ve Feather Adaptörü (`sql_adapter`)

`sql_adapter` modülü, Atlas `Table` nesneleri ile SQL veritabanları (SQLite, PostgreSQL) ve Feather ORM arasında tip-güvenli köprü kurar.

### DDL ve INSERT Üretimi

```nox
import atlas
from atlas import Table, table_to_create_table_sql, table_to_insert_sql

# SQLite uyumlu CREATE TABLE DDL
ddl: str = table_to_create_table_sql(t, "users", dialect="sqlite", primary_key_col="id")

# Parametrik INSERT (SQL enjeksiyonuna karşı %100 korumalı ?)
insert_sql: str = table_to_insert_sql(t, "users")
# Çıktı: INSERT INTO users (id, name, age) VALUES (?, ?, ?);
```

### Veritabanı Kayıtlarından Tablo Oluşturma

```nox
from atlas import table_from_records, cell_from_db_value

# Sütun adları ve tipleri ile doğrudan kayıt listesinden Table oluşturma:
t: Table = table_from_records(col_names, col_types, record_rows)
```

---

## 2. Noxen-Uyumlu Şema Doğrulama (`validation`)

`validation` modülü, veri temizleme ve ETL süreçlerinde tablodaki satırların kurallara uygunluğunu denetler.

### Desteklenen Kurallar

| Fonksiyon | Açıklama |
| :--- | :--- |
| `rule_non_empty(col)` | Hücrenin null veya boş dize olmamasını zorunlu kılar. |
| `rule_range(col, min, max)` | Sayısal değerin `[min, max]` aralığında olmasını denetler. |
| `rule_min(col, min)` | Sayısal değerin $\ge min$ olmasını denetler. |
| `rule_max(col, max)` | Sayısal değerin $\le max$ olmasını denetler. |
| `rule_length(col, min, max)` | Metin uzunluğunun `[min, max]` karakter arasında olmasını denetler. |
| `rule_allowed_values(col, list)` | Değerin izin verilenler kümesinde bulunmasını denetler. |
| `rule_email(col)` | Geçerli bir e-posta adresi formatı gerektirir. |

### Doğrulama ve Satır Filtreleme

```nox
from atlas import TableValidator, ValidationReport, rule_non_empty, rule_email, rule_range

validator: TableValidator = TableValidator()
validator.add_rule(rule_non_empty("email"))
validator.add_rule(rule_email("email"))
validator.add_rule(rule_range("age", 18.0, 65.0))

# 1. Rapor alma:
report: ValidationReport = validator.validate(table)
if not report.is_valid():
    print(report.summary())

# 2. Geçerli satırları süzme:
clean_table: Table = validator.filter_valid(table)

# 3. Hatalı satırları inceleme için ayırma:
quarantine_table: Table = validator.filter_invalid(table)
```
