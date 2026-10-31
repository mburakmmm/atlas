# Ekosistem Entegrasyonları (Ecosystem Integrations)

Atlas, Nox ekosistemindeki diğer kütüphanelerle gevşek bağlı (loosely-coupled), sıfır zorunlu bağımlılık ilkesine dayanan entegrasyon adaptörleri sunar.

---

## 1. İlişkisel Veritabanı ve Feather Adaptörü (`sql_adapter`)

`sql_adapter` modülü, Atlas `Table` nesneleri ile SQL veritabanları (SQLite, PostgreSQL) ve Feather ORM arasında tip-güvenli köprü kurar.

### DDL ve INSERT Üretimi

```nox
import atlas
from atlas import Table, table_to_create_table_sql, table_to_insert_sql, quote_identifier

# SQLite uyumlu ve güvenli biçimde tırnaklanmış CREATE TABLE DDL
ddl_sqlite: str = table_to_create_table_sql(t, "users", dialect="sqlite", primary_key_col="id")
# Çıktı: CREATE TABLE IF NOT EXISTS "users" ( "id" INTEGER NOT NULL PRIMARY KEY, ... );

# PostgreSQL uyumlu parametrik INSERT ($1, $2, ...)
insert_pg: str = table_to_insert_sql(t, "users", dialect="postgres")
# Çıktı: INSERT INTO "users" ("id", "name") VALUES ($1, $2);

# SQLite uyumlu parametrik INSERT (?, ?)
insert_sqlite: str = table_to_insert_sql(t, "users", dialect="sqlite")
# Çıktı: INSERT INTO "users" ("id", "name") VALUES (?, ?);
```

### Feather ORM & Veritabanı Kayıtlarından Tablo Oluşturma

Feather sorgularından (`SelectQuery` / `Session.execute`) dönen satırlar doğrudan Atlas tablosuna aktarılabilir:

```nox
from atlas import table_from_records, cell_from_db_value, Cell

# Feather veya DB sorgu sonuçlarını Atlas Table nesnesine dönüştürme:
col_names: list[str] = ["id", "username", "balance"]
col_types: list[int] = [TYPE_INT, TYPE_STR, TYPE_FLOAT]

# DB kayıt satırları:
records: list[list[Cell]] = [
    [cell_from_db_value(1, 101, 0.0, "", False, False), cell_from_db_value(3, 0, 0.0, "alice", False, False), cell_from_db_value(2, 0, 150.50, "", False, False)],
]

t: Table = table_from_records(col_names, col_types, records)
```

---

## 2. Noxen-Uyumlu Şema Doğrulama (`validation`)

`validation` modülü, veri temizleme ve ETL süreçlerinde tablodaki satırların kurallara uygunluğunu denetler.

### Desteklenen Kurallar

| Fonksiyon | Açıklama |
| :--- | :--- |
| `rule_non_empty(col)` | Hücrenin null veya boş dize olmamasını zorunlu kılar. |
| `rule_range(col, min, max)` | Sayısal değerin `[min, max]` aralığında olmasını denetler; sayısal olmayan değerleri yakalar. |
| `rule_min(col, min)` | Sayısal değerin $\ge min$ olmasını denetler; sayısal olmayan değerleri yakalar. |
| `rule_max(col, max)` | Sayısal değerin $\le max$ olmasını denetler; sayısal olmayan değerleri yakalar. |
| `rule_type(col, type)` | Hücrenin açık tipini (`TYPE_INT`, `TYPE_FLOAT`, `TYPE_STR`, `TYPE_BOOL`) zorunlu kılar. |
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
