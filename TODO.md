# Atlas — Proje Geliştirme ve Sürümlendirme Yol Haritası (TODO.md)

Bu dosya, `AGENTS.md` bağlayıcı mühendislik sözleşmesi temel alınarak hazırlanmış; tamamlanan, sürmekte olan ve sonraki sürümlere planlanan tüm görevlerin merkezi takip ve sürüm yönetim belgesidir.

---

## 📌 Sürüm Özeti & Faz Durumu

| Sürüm | Faz / Odak | Durum | Test Durumu |
| :--- | :--- | :--- | :--- |
| **v0.1.0** | **Temel Tablo, CSV, Filtreleme, Sıralama, İstatistik, GroupBy, JSON** | ✅ **TAMAMLANDI & YAYINLANDI** | 12/12 Test Geçti (`noxc test`) |
| **v0.2.0** | **Tablo Birleştirme (Inner/Left Join) & Tablo Ekleme (Concat)** | ✅ **TAMAMLANDI & YAYINLANDI** | 14/14 Test Geçti (`noxc test`) |
| **v0.3.0** | **Özel Ayraçlar (TSV, `;`), Kartiller (Quantiles) & Izgara Gösterimi** | ✅ **TAMAMLANDI & YAYINLANDI** | 17/17 Test Geçti (`noxc test`) |
| **v0.4.0** | **Feather (SQL ORM) & Noxen (Validasyon) İsteğe Bağlı Adaptörleri** | ✅ **TAMAMLANDI & YAYINLANDI** | 19/19 Test Geçti (`noxc test`) |
| **v0.5.0** | **Büyük Ölçekli Performans (100k), API Konsolidasyonu & Kılavuzlar** | ✅ **TAMAMLANDI & YAYINLANDI** | 19/19 Test Geçti (`noxc test`) |
| **v0.6.0** | **Veri Doğruluğu, Hash Join, Hash GroupBy & SQL Identifier Güvenliği** | ✅ **TAMAMLANDI & YAYINLANDI** | 22/22 Test Geçti (`noxc test`) |
| **v0.6.1** | **GroupBy Collision Buckets, Karma Float/Int Hassasiyeti & Katı CSV** | ✅ **TAMAMLANDI & YAYINLANDI** | 25/25 Test Geçti (`noxc test`) |
| **v0.6.2** | **Matematiksel Sınır Güvenliği, Ortak Hash Kodlama & CsvOptions** | ✅ **TAMAMLANDI & YAYINLANDI** | 28/28 Test Geçti (`noxc test`) |
| **v0.7.0** | **Akışlı CSV (Streaming Reader), NaN Sıralama Tutarlılığı & Akış Agregasyonları** | ✅ **TAMAMLANDI & YAYINLANDI** | 29/29 Test Geçti (`noxc test`) |
| **v0.7.1** | **Streaming Sağlamlaştırma, Durum Makinesi, max_record_bytes & Çevrimiçi İstatistikler** | ✅ **TAMAMLANDI & YAYINLANDI** | 30/30 Test Geçti (`noxc test`) |
| **v0.7.2** | **Çok Karakterli Delimiter, UTF-8 Bayt Sınırları, Bounded Memory & Diferansiyel Testler** | ✅ **TAMAMLANDI & YAYINLANDI** | 31/31 Test Geçti (`noxc test`) |
| **v0.7.3** | **Çok Karakterli Delimiter Bayt Sayımı, EOF Sınırları & Property-Based Testler** | ✅ **TAMAMLANDI & YAYINLANDI** | 31/31 Test Geçti (`noxc test`) |
| **v1.0.0** | **Nihai Stabil Genel API & Topluluk Sürümü** | ⏳ **PLANLANDI (İleride)** | — |

---

## ✅ Sürüm 0.1.0 — Temel Çekirdek (Tamamlananlar)

### Faz 0: Keşif & Nox Standart Kütüphane Denetimi
- [x] NOX derleyici sürümünün ve çalışma zamanının denetlenmesi (`noxc 2.0.86`, macOS/Darwin arm64).
- [x] `nox.csv` yeteneklerinin ve sınırlarının doğrulanması (RFC 4180 metin tabanlı ayrıştırma, string dönüşü, delimiter/streaming kısıtları).
- [x] `nox.json` ve `JsonWriter` arabellekli akış yeteneklerinin test edilmesi.
- [x] `nox.mathx` libc-bağımsız sayısal fonksiyonlarının doğrulanması (`sqrt`, `min`, `max`, `abs`).
- [x] `nox.fs` dosya G/Ç ve üzerine yazma (overwrite) davranışlarının incelenmesi.
- [x] Dil özellikleri testi: `Cell` etiketli union (tagged record), `T | None` daraltma, generic'ler, istisna hiyerarşisi.
- [x] `docs/capabilities.md` ve `docs/architecture.md` belgelerinin oluşturulması.

### Faz 1: Atlas 0.1 Veri Modeli, Şema & CSV Motoru
- [x] `values.nox`: Tip-güvenli `Cell` sınıfı (`TYPE_NULL`, `TYPE_INT`, `TYPE_FLOAT`, `TYPE_STR`, `TYPE_BOOL`).
- [x] Sayısal eşitlik ve tip dönüştürme kuralları (`cell_int(10).equals(cell_float(10.0)) == True`).
- [x] `schema.nox`: `ColumnDef` ve `Schema` yapıları (benzersiz sütun adı kontrolü, şema boyutu, tip indeksleme).
- [x] Sütun izdüşüm işlemleri (`select_columns`, `drop_columns`, `rename_column`).
- [x] `csv_adapter.nox`: `read_csv` ile otomatik tip çıkarımı (`int`, `float`, `bool`, `str`).
- [x] RFC 4180 alıntılı alanlar (virgül, tırnak, satır sonu içeren hücreler).
- [x] Dengesiz (ragged) satır ve şema boyut uyumsuzluğu denetimi (`ShapeError`).
- [x] Nullable olmayan sütunda eksik veri tespitinde katı hata fırlatma (`MissingValueError`).
- [x] Elektronik tablo formül enjeksiyonu (`=`, `+`, `-`, `@`) güvenli dışa aktarım modu (`safe_spreadsheet=True`).

### Faz 2: Dönüşümler, Sıralama & Eksik Veri
- [x] `expr.nox`: Tip-güvenli deklaratif AST filtre ifadeleri (`col_eq`, `col_neq`, `col_gt`, `col_gte`, `col_lt`, `col_lte`).
- [x] Dize filtreleri (`col_contains`, `col_starts_with`).
- [x] Küme üyelik kontrolü (`col_in`).
- [x] Mantıksal bileştiriciler (`expr_and`, `expr_or`, `expr_not`).
- [x] Üç-değerli mantık: Karşılaştırmalarda null değerlerin açık yönetimi.
- [x] `sort.nox`: Çok sütunlu, $O(N \log N)$ worst-case garantili **stabil merge sort**.
- [x] Sütun bazında artan/azalan (`ascending_flags`) sıralama desteği.
- [x] Eksik verilerin başta veya sonda sıralanması (`nulls_first=True/False`).
- [x] Eksik veri işlemleri (`count_missing`, `fill_missing`, `drop_missing` [any/all politikaları]).
- [x] Orijinal tabloyu mutasyona uğratmayan (immutable-by-default) dönüşüm garantisi.

### Faz 3: Sayısal İstatistik Motoru
- [x] `stats.nox`: Aritmetik ortalama (`stat_mean`).
- [x] Tek ve çift elemanlı dizilerde tam medyan (`stat_median`).
- [x] Sayısal taşma ve iptal hatalarına karşı **Welford çevrimiçi algoritması**.
- [x] Örneklem varyansı ($n-1$) ve popülasyon varyansı ($n$).
- [x] Standart sapma (`stat_std` via `nox.mathx.sqrt`).
- [x] Minimum, maksimum, toplam ve non-null sayım metrikleri.
- [x] Boş veya tamamen eksik veri kümelerinde deterministik hata yönetimi (`AggregationError`).

### Faz 4: Gruplama & Çoklu Metrik Agregasyonları
- [x] `groupby.nox`: Tekli ve çoklu anahtarlarla deterministik gruplama (`group_by`).
- [x] Grupların ilk görülme sırasını koruyan stabil grup sıralaması.
- [x] `AggregateDef` çerçevesi (`agg_count`, `agg_sum`, `agg_mean`, `agg_median`, `agg_std`, `agg_variance`, `agg_min`, `agg_max`, `agg_count_non_missing`).
- [x] Eksik anahtarların ayrı bir grup olarak güvenle işlenmesi.
- [x] Kolaylık yöntemi: `grouped.count()`.

### Faz 5: JSON Desteği, Dosya G/Ç & Ön Cephe (Facade)
- [x] `json_adapter.nox`: JSON nesne dizisi (`array of objects`) ↔ Tablo çift yönlü dönüşümü (`table_from_json`, `table_to_json`).
- [x] Tek adımda `csv_to_json` ve `json_to_csv` dönüştürücüleri.
- [x] `io.nox`: Dosya G/Ç sarmalayıcıları (`read_csv_file`, `write_csv_file`, `read_json_file`, `write_json_file`).
- [x] Yanlışlıkla dosya ezilmesini önleyen `overwrite=False` koruması.
- [x] `atlas.nox`: Tüm modülleri tek bir import çatısında toplayan merkezi cephe.
- [x] Çalıştırılabilir demo ve CLI doğrulama aracı (`main.nox`).
- [x] 12 adet otomatik birim/entegrasyon testi (`tests/`).
- [x] 4 adet çalıştırılabilir örnek (`examples/`).
- [x] 10.000 satırlık performans kıyaslama betiği (`benchmarks/`).
- [x] 13 adet detaylı mimari ve teknik doküman (`docs/`).

---

## ✅ Sürüm 0.2.0 — İlişkisel İşlemler & Tablo Birleştirme (Tamamlananlar)

- [x] **İç Birleştirme (Inner Join)**:
  - İki tablonun ortak anahtar sütunları üzerinden birleştirilmesi (`join_inner`, `join_inner_keys`).
  - Eşleşmeyen satırların elenmesi ve çakışan sütun adlarına otomatik önek/sonek (`_left`, `_right`) eklenmesi.
- [x] **Sol Dış Birleştirme (Left Join)**:
  - Sol tablodaki tüm satırların korunması (`join_left`, `join_left_keys`).
  - Sağ tabloda eşleşmeyen satırlar için sağ sütunların `Cell(TYPE_NULL)` ile doldurulması.
  - Null anahtarların birbiriyle eşleşmemesi garantisi.
- [x] **Satır Ekleme / Dikey Birleştirme (`concat_rows`)**:
  - Şeması uyumlu iki veya daha fazla tablonun alt alta birleştirilmesi.
  - Şema sütun sırası farklıysa ad bazında otomatik hizalama.
  - Eksik sütunlar için esnek doldurma veya katı şema modu.
- [x] **Sütun Ekleme / Yatay Birleştirme (`concat_cols`)**:
  - Eşit satır sayısına sahip tabloların yan yana birleştirilmesi.
  - Çakışan sütun adlarının `_2`, `_3` sonekiyle otomatik çözülmesi (`suffix_collisions=True`).
- [x] **Birim Testler**:
  - `tests/join_test.nox` (1-1, 1-N, N-N join senaryoları, null anahtarlar, farklı anahtar adları).
  - `tests/concat_test.nox` (dikey hizalı birleştirme ve yatay birleştirme testleri).
- [x] **Dokümantasyon & Örnekler**:
  - `docs/joins.md` ve `docs/concatenation.md` kılavuzları.
  - `examples/table_joins.nox` çalıştırılabilir örneği.

---

## ✅ Sürüm 0.3.0 — Özel Ayraçlar & İleri İstatistikler (Tamamlananlar)

- [x] **Özel CSV Ayraç Ayrıştırıcısı (Custom Delimiters)**:
  - Standart virgül dışındaki ayraçlar: Noktalı virgül (`;`), Tab (`\t`), Pipe (`|`).
  - RFC 4180 durum makinesinin ayraç parametresiyle genelleştirilmesi (`delimited.nox`).
  - TSV desteği (`read_tsv`, `to_tsv`, `read_tsv_file`, `write_tsv_file`).
  - Özel ayraçlı G/Ç (`read_delimited`, `to_delimited`, `read_delimited_file`, `write_delimited_file`).
- [x] **Kartiller ve Yüzdelikler (Quantiles & Percentiles)**:
  - $p$-yüzdelik hesabı (`table.quantile(col_name, q)` $0.0 \le q \le 1.0$).
  - Doğrusal enterpolasyon (linear interpolation / R-7 / NumPy uyumlu) kuralları.
  - Kartiller arası açıklık (`table.iqr(col_name)` = Q3 - Q1).
- [x] **Aykırı Değer (Outlier) Tespiti**:
  - IQR filtresi ile aykırı satırları süzme (`table.filter_outliers_iqr(col_name, multiplier=1.5)`).
- [x] **Terminal Izgara Tablo Formatlayıcısı (Pretty Grid Formatter)**:
  - `table.to_grid_string()` ve `table.print_grid()` metotları ile sınır hizalı, otomatik sütun genişlikli ASCII ızgara gösterimi.
- [x] **Birim Testler**:
  - `tests/tsv_test.nox` (TSV, noktalı virgül, pipe, alıntılı alanlar ve dosya G/Ç).
  - `tests/quantile_test.nox` (doğrusal enterpolasyon, kartiller, IQR, aykırı değer süzme).
  - `tests/grid_test.nox` (ızgara formatlayıcı ve hücre kırpma testleri).
- [x] **Dokümantasyon & Örnekler**:
  - `docs/delimiters.md` ve `docs/quantiles.md`.
  - `examples/delimited_and_quantiles.nox`.

---

## ✅ Sürüm 0.4.0 — Ekosistem Adaptörleri (Tamamlananlar)

- [x] **Feather & SQL Veritabanı Adaptörü (`sql_adapter.nox`)**:
  - `table_to_create_table_sql(table, table_name, dialect, primary_key_col)` ile SQLite ve PostgreSQL DDL şeması üretimi.
  - `table_to_insert_sql(table, table_name)` ile SQL enjeksiyonuna karşı %100 korumalı parametrik `?` INSERT ifadesi.
  - `table_from_records(col_names, col_types, rows)` ile veritabanı sorgu kayıtlarından doğrudan `Table` üretimi.
  - `cell_from_db_value(kind, i, f, s, b, is_null)` ile Feather `Value` ve standart DB değerlerinin `Cell` dönüşümü.
- [x] **Noxen-Uyumlu Şema Doğrulama Motoru (`validation.nox`)**:
  - `TableValidator` ve sütun bazlı kural seti: `rule_non_empty`, `rule_range`, `rule_min`, `rule_max`, `rule_length`, `rule_allowed_values`, `rule_email`.
  - Hata detaylarını ve satır/sütun indekslerini raporlayan `ValidationReport`.
  - Tablo satırlarını geçerli ve geçersiz olarak ayırma: `validator.filter_valid(table)` ve `validator.filter_invalid(table)`.
- [x] **Birim Testler & Dokümantasyon**:
  - `tests/sql_adapter_test.nox` (DDL, INSERT, record mapping, DB cell dönüşümleri).
  - `tests/validation_test.nox` (kurallar, hata raporlama, geçerli/geçersiz satır süzme).
  - `docs/integrations.md` ve `examples/sql_and_validation.nox`.

---

## ✅ Sürüm 0.5.0 — Büyük Ölçekli Performans & API Konsolidasyonu (Tamamlananlar)

- [x] **Genel API Konsolidasyonu**:
  - 16 temel modülün (`atlas`, `table`, `schema`, `values`, `expr`, `sort`, `stats`, `groupby`, `csv_adapter`, `delimited`, `json_adapter`, `io`, `join`, `concat`, `sql_adapter`, `validation`) tek çatı altında eksiksiz toplanması.
  - Tip güvenliği, açık null politikası ve taşma/iptal korumalı Welford istatistikleri.
- [x] **Tüm Dokümantasyonun ve API Referansının Eksiksiz Doğrulanması**:
  - `docs/api.md` tüm modül fonksiyonlarını ve metotlarını içerecek şekilde güncellendi.
  - 14 detaylı teknik doküman (`docs/`).
- [x] **Çapraz Platform ve Backend Doğrulama**:
  - Saf Nox sözdizimi, sıfır harici C/C++ bağımlılığı; hem hosted hem freestanding runtime uyumlu.
- [x] **Büyük Veri Kümesi Performans Bütçelerinin Doğrulanması**:
  - `benchmarks/benchmark_large.nox`: 100.000 satır 11 ms'de belleğe alındı, 7 ms'de filtrelendi, 4 ms'de ortalama hesaplandı, 18 ms'de 5 kategoride gruplandı.
- [x] **Güvenlik Denetimi ve Son Kontrol Listesi Onayı**:
  - Elektronik tablo formül enjeksiyonu (`=`, `+`, `-`, `@`) koruması, dosya üzerine yazma (`overwrite=False`) koruması, SQL parametrik `?` binding güvenliği.
- [x] **`noxc publish` ile v0.5.0 İndeks Yayını**:
  - Atlas v0.5.0 paketi başarıyla onay kuyruğuna gönderildi.

---

## ✅ Sürüm 0.6.0 — Veri Doğruluğu, Hash Join, Hash GroupBy & SQL Güvenliği (Tamamlananlar)

- [x] **Cell Veri Modeli Tip Dönüşümü ve 64-bit Integer Doğruluğu (🔴 P1)**:
  - `values.nox`: `Cell.as_float()` metodunda `TYPE_INT` için ikincil float alanına bağımlılığı kaldırarak doğrudan `float(self.i)` döndürme.
  - `cell_int(i)` yapıcısında float alanını `0.0` yaparak tekil kanonik değer tutma.
  - `Cell.equals()` ve `Cell.compare()` içinde iki `TYPE_INT` hücresini IEEE-754 double precision yuvarlama ve 53-bit üstü ($2^{53}$) hassasiyet kaybı yaşamadan doğrudan 64-bit tamsayı olarak karşılaştırma.
- [x] **SQL Adaptöründe Güvenli Identifier Yönetimi & PostgreSQL Desteği (🔴 P1)**:
  - `sql_adapter.nox`: `quote_identifier(name, dialect)` fonksiyonu ekleyerek tablo ve sütun isimlerinin kaçış karakterleriyle güvenli biçimde tırnak içine alınması (SQL Identifier Injection koruması).
  - Identifier boşluk ve `\0` (null byte) doğrulaması.
  - `table_to_insert_sql(t, table_name, dialect)` içine dialect desteği: PostgreSQL için `$1, $2, ...`, SQLite için `?, ?`.
- [x] **Hash Join ile O(N + M) Birleştirme (🔴 P1)**:
  - `join.nox`: $O(N \times M)$ nested-loop join yerine sağ tablonun birleştirme anahtarlarını hash tablosunda indeksleyen Hash Join ($O(N + M)$) algoritmasına geçiş.
  - Sayısal eşitlik (`10` == `10.0`), dize ve boolean anahtarlarını ayrıştıran tip-güvenli anahtar serileştirme (`_cell_hash_key`).
  - Null anahtarların asla eşleşmemesi kuralının (null key isolation) korunması.
- [x] **Hash GroupBy ile O(N) Gruplama (🔴 P1)**:
  - `groupby.nox`: Grup anahtarı aramasında $O(G)$ doğrusal tarama yerine $O(1)$ hash map lookup (`dict[str, int]`) kullanarak gruplama maliyetini $O(N \times G)$'den $O(N)$ seviyesine düşürme.
  - Yüksek kardinaliteli veri kümelerinde stabil grup sırasını koruma.
- [x] **CSV / Delimited Motoru Düzeltmeleri (🔴 P1 & 🔴 P2)**:
  - `delimited.nox`: Tırnaklı alan başlangıcını sadece alan başına (`len(cur_field) == 0`) kısıtlama, alan ortasında tırnak ayrıştırma denetimi.
  - `dump_delimited`: Çok karakterli ayraçların (`||`, `::`) alan içinde bulunması durumunda tırnak içine alınmasının güvenceye alınması (round-trip veri doğruluğu).
- [x] **Katı Tip Denetimli Tablo Validasyonu (🟠 P1)**:
  - `validation.nox`: `rule_range`, `rule_min`, `rule_max` kurallarında sayısal olmayan hücrelerin (`"abc"`) sessizce geçmesini engelleyip katı `ValidationError` fırlatması.
  - `rule_type(col_name, expected_type)` kuralı ekleyerek açık sütun tip doğrulama desteği.
- [x] **Yüksek Kardinalite & Hash Join Benchmark Matrisi (🟠 P2)**:
  - `benchmarks/benchmark_hash.nox`: 10.000 satırda 10.000 benzersiz grup (yüksek kardinalite, 3 ms) ve 10.000 x 5.000 Hash Join (2 ms) hız ölçümleri.
- [x] **Feather Entegrasyon Kılavuzu & Tip Eşleme İyileştirmeleri (🟠 P2)**:
  - `docs/integrations.md` ve `sql_adapter.nox` üzerinden Feather oturum ve sorgu dönüştürme desenlerinin genişletilmesi.
- [x] **Tüm Birim ve Regresyon Testlerinin Doğrulanması**:
  - `tests/hash_join_test.nox`, `tests/hash_groupby_test.nox`, `tests/values_precision_test.nox` dahil 22 test paketinin tamamının geçmesi.

---

## 🛠️ Sürüm 0.6.1 — Sağlamlaştırma, Çakışma Yönetimi & Veri Doğruluğu (Teknik İnceleme İyileştirmeleri)

- [x] **Hash GroupBy Çakışma Yönetimi (Collision Bucket List) (🔴 P1)**:
  - `groupby.nox`: `group_map: dict[str, int]` yerine bucket indekslemesi `dict[str, list[int]]` yapısına geçiş.
  - Aynı hash anahtarına sahip farklı grupların (`k_tuple`) aday listesinde (`list[int]`) saklanması ve `equals()` ile taranması.
  - Yeni çakışan grup eklendiğinde `group_map[h_key]` listesinin güncellenmesi; böylece aynı anahtara sahip satırların asla ayrı gruplara bölünmemesi garantisi.
- [x] **Karma Integer/Float Karşılaştırma Doğruluğu (🔴 P1)**:
  - `values.nox`: `Cell.equals()` ve `Cell.compare()` içinde biri `TYPE_INT`, diğeri `TYPE_FLOAT` olan karma karşılaştırmalarda $2^{53}$ üstü değerlerde IEEE-754 yuvarlama hatasını önleme.
  - Float değerin tamsayı karşılığı tam olarak float'a eşit değilse (`float(int(f)) != f`), integer ile eşit olamayacağı kuralı; eşitse 64-bit integer dönüşümü üzerinden doğrudan integer eşitliği (`int(f) == i`).
- [x] **Uzunluk Önekli / Delimiter-Güvenli Hash Anahtarı Kodlaması (🟠 P2)**:
  - `join.nox` & `groupby.nox`: Çok sütunlu anahtar birleşiminde dize değerleri içindeki ayraçların (`\x1f`, `S:`) oluşturabileceği çapraz sütun hash çakışmalarını önlemek için uzunluk önekli (length-prefixed) kodlama formatı (`"S" + str(len(s)) + ":" + s`).
- [x] **SQL Adaptörü Dialect ve Primary Key Doğrulaması (🟠 P2)**:
  - `sql_adapter.nox`: Desteklenmeyen SQL dialect'leri için sessizce varsayılana geçmek yerine açık `SchemaError` fırlatılması.
  - `table_to_create_table_sql`: Belirtilen `primary_key_col` tabloda yoksa sessizce atlamak yerine `ColumnNotFoundError` fırlatılması.
- [x] **CI Test Kararlılığı: Süre Sınırı Assert'lerinin Ayrılması (🟠 P2)**:
  - `tests/hash_join_test.nox` ve `tests/hash_groupby_test.nox`: CI ortamlarında sistem yükü dalgalanmalarında kırılganlık yaratabilecek duvar saati `assert duration < X` kontrollerinin birim testlerinden kaldırılıp `benchmarks/` dizinine taşınması.
- [x] **Katı ve Toleranslı CSV Ayrıştırma Modu (`strict=True / False`) (🟡 P3)**:
  - `delimited.nox`: `parse_delimited`, `read_delimited`, `read_tsv` fonksiyonlarına `strict: bool = False` parametresi ekleme.
  - `strict=True` modunda alan ortasında kaçışsız tırnak veya kapanış tırnağı sonrasında geçersiz karakter tespitinde `CsvImportError` fırlatılması; `strict=False` modunda ise toleranslı gerçek dünya CSV desteğinin korunması.
- [x] **Birim Testler & Doğrulama**:
  - `tests/groupby_collision_test.nox`, `tests/values_precision_test.nox`, `tests/sql_validation_test.nox`, `tests/strict_csv_test.nox` dahil 25 test paketinin tamamının geçmesi.

---

## 🛠️ Sürüm 0.6.2 — Matematiksel Sınır Güvenliği, Ortak Hash Kodlama & CsvOptions (Teknik İnceleme İyileştirmeleri)

- [x] **Karma Integer/Float Sınır Durumları ve Dönüşüm Güvenliği (🔴 P1)**:
  - `values.nox`: `9223372036854775000.0` gibi keyfi ve yuvarlanan float sabit sınırı yerine tam $2^{63}$ (`9223372036854775808.0`) ve $-2^{63}$ (`-9223372036854775808.0`) sınırları ile float'ın 64-bit int aralığında (`[-2^63, 2^63)`) olduğunun matematiksel olarak garanti altına alınması.
  - $9223372036854774784$ (near INT64_MAX, 1024'ün katı olan tam temsil edilebilir float) gibi geçerli değerlerin eşitliklerinin doğrulanması.
  - INT64_MAX ($2^{63}-1$) ile $2^{63}.0$ arasındaki farkın (1 fark) eşit olmadığının ve $i < f$ olduğunun doğrulanması.
  - INT64_MIN ($-2^{63}$) ile $-2^{63}.0$ tam eşitliğinin korunması.
- [x] **Kesirli Float Sıralamalarında `float(i)` Dönüşümünün Kaldırılması (🔴 P1)**:
  - `values.nox`: `_int_float_compare` fonksiyonunda kesirli float'lar için `float(i)` dönüşümü yerine, float'ın kesirli aralığı (`f_trunc < f < f_trunc + 1` veya negatif için `f_trunc - 1 < f < f_trunc`) kullanılarak doğrudan 64-bit tamsayı mantığıyla (`i <= f_trunc` vb.) karşılaştırma yapılması.
  - Böylece büyük integer değerlerde (`9007199254740993` vb.) float yuvarlamasından kaynaklanabilecek sıralama bozulmalarının tamamen önlenmesi.
- [x] **IEEE-754 NaN ve Sonsuzluk (Infinity) Davranışlarının Tanımlanması (🔴 P1)**:
  - `values.nox`: `not (f == f)` ile NaN tespiti; `Cell.equals` içinde NaN-NaN eşitliği (tablo operasyonları için tutarlı), `Cell.compare` içinde NaN'ın en sona sıralanması (`NaN sorts last`).
  - Pozitif sonsuzluğun (+Inf) tüm sonlu integer'lardan büyük (`i < +Inf`), negatif sonsuzluğun (-Inf) tüm sonlu integer'lardan küçük (`i > -Inf`) olarak deterministik karşılaştırılması.
- [x] **Ortak Hash Key Kodlama Modülü (`key_encoding.nox`) (🟠 P2)**:
  - `join.nox` ve `groupby.nox` arasındaki kod tekrarını (`_cell_hash_key`, `_cells_hash_key`, `_cell_hash_repr`, `_cells_hash_repr`) gidermek için `key_encoding.nox` modülü oluşturulması (`cell_hash_key`, `composite_hash_key`).
  - Join ve GroupBy motorlarının tamamen aynı hash kurallarını ve $O(1)$ normalizasyonunu kullanmasının güvenceye alınması.
- [x] **Hash ve Equality Tutarlılığı ile Gerçek Çakışma Bucket Testleri (🟠 P2)**:
  - `tests/groupby_collision_test.nox`: Hash anahtarları aynı bucket'a düşen fakat `Cell.equals()` ile farklı olan aday grupların fallback mekanizmasının ve `Cell` eşitliğinin doğrulanması.
  - `tests/hash_consistency_test.nox`: Join ve GroupBy hash anahtarlarının aynı veri tipleri ve hücrelerde birebir aynı hash dizesini ürettiğinin test edilmesi.
- [x] **Feather & SQL Veritabanı Entegrasyon Test Matrisi (🟠 P2)**:
  - `tests/sql_integration_test.nox`: PostgreSQL positional placeholder `$1, $2`, MySQL identifier backtick, SQLite çift tırnak, veri tipi eşleştirmeleri, NOT NULL ve PRIMARY KEY kurgularının doğrulanması.
- [x] **CSV Seçenek Nesnesi (`CsvOptions`) (🟡 P3)**:
  - `delimited.nox`: `CsvOptions` sınıfı, `csv_options(...)` yardımcı fonksiyonu ve `read_delimited_options(text, options)` fonksiyonunun eklenmesi; mevcut API imzalarının geriye dönük tam uyumlu tutulması.
- [x] **Birim Testler & Sürüm Yayın**:
  - `tests/hash_consistency_test.nox`, `tests/sql_integration_test.nox`, `tests/csv_options_test.nox` dahil 28 test paketinin `noxc test` ile doğrulanması, `nox.json` sürümünün 0.6.2 olarak güncellenmesi, `CHANGELOG.md` kaydı, git commit, GitHub push ve `noxc publish`.

---

## 🚀 Sürüm 0.7.0 — Akışlı CSV (Streaming Reader), NaN Sıralama Tutarlılığı & Akış Agregasyonları

- [x] **Tüm Türler Arasında NaN Sıralama Tutarlılığı (🔴 P1)**:
  - `values.nox`: `Cell.compare` fonksiyonunda null denetiminden hemen sonra türden bağımsız `self_nan` ve `other_nan` denetimi.
  - NaN hücresinin yalnızca sayılarla değil, String, Bool ve özel türler karşısında da daima en sona (`NaN sorts last`) yerleştirilmesi.
- [x] **Taşınabilir IEEE-754 Sabit Fonksiyonları (🔴 P1)**:
  - `values.nox`: Derleyici backend'leri (LLVM, QBE, WASM) arasında sıfıra bölme tutarsızlığını önlemek için `float_nan()`, `float_inf()`, `float_neg_inf()`, `cell_nan()`, `cell_inf()`, `cell_neg_inf()`, `is_nan()`, `is_inf()` fonksiyonlarının eklenmesi.
  - Test ve kütüphane kodlarında ham sıfıra bölme ifadeleri yerine bu fonksiyonların kullanılması.
- [x] **Genişletilmiş / Bilinmeyen Hücre Tiplerinde Hash Kodlama (🟠 P2)**:
  - `key_encoding.nox`: Standart dışı hücre tiplerinde sabit `"X"` yerine `"U" + str(kind) + ":" + str(i) + ":" + to_string()` kodlaması ile özel türlerde hash çakışması yığılmasının önlenmesi.
- [x] **Akışlı CSV Okuyucu (Streaming CSV Reader & Chunk Processing) (🔴 P1)**:
  - `streaming.nox`: `nox.fs.open_read(path)` ve `read_chunk(n)` kullanan, tüm dosyayı belleğe almadan parça parça (chunk-by-chunk) okuyan `StreamingCsvReader` sınıfı.
  - `read_csv_chunks(path, chunk_rows, options)`: Büyük CSV dosyalarını yapılandırılabilir satır blokları (`Table` parçaları) halinde dönen akış arayüzü.
  - `stream_count(path, options)`: Tüm dosyayı belleğe yüklemeden satır sayısını sayan $O(1)$ bellekli akış fonksiyonu.
  - `stream_sum(path, col_name, options)` ve `stream_mean(path, col_name, options)`: Akış üzerinde çalışan, sınırsız boyutlu dosyalarda çalışan istatistik agregasyonları.
- [x] **SQL Kontrat ve Feather Köprü Testleri (🟠 P2)**:
  - `tests/sql_adapter_contract_test.nox`: SQL adaptörünün DDL, INSERT ve Feather DB record dönüşümlerinin kontrat doğrulaması.
- [x] **Birim Testler & Sürüm Yayın**:
  - `tests/streaming_test.nox` dahil 29 test paketinin `noxc test` ile doğrulanması, `nox.json` sürümünün `0.7.0` olarak güncellenmesi, `CHANGELOG.md` kaydı, git commit, GitHub push ve `noxc publish`.

---

## 🛠️ Sürüm 0.7.1 — Streaming Sağlamlaştırma, Durum Makinesi & Kenar Durum Güvenliği (Teknik İnceleme İyileştirmeleri)

- [x] **Kalıcı Ayrıştırıcı Durum Makinesi (Persistent State Machine) (🔴 P1)**:
  - `streaming.nox`: Tamponun baştan taranarak $O(L^2/B)$ karesel maliyet oluşturmasını önlemek için çağrılar ve chunk sınırları arasında kalıcı durum makinesi (`in_quotes`, `after_quote`, `field_start`, `pending_cr`, `pending_quote`, `cur_field`, `cur_row`, `pending_rows`).
  - Her karakterin yalnızca bir kez taranarak kesin $O(N)$ doğrusal zaman karmaşıklığının güvenceye alınması.
- [x] **Tampon Yönetimi & Gereksiz Disk Okumalarının Önlenmesi (🔴 P1)**:
  - `next_chunk()` çağrıldığında `pending_rows` içerisinde `target_rows` sayısını karşılayacak kadar hazır satır varsa diskten yeni chunk okunmaması.
  - Sadece yeni satıra ihtiyaç duyulduğunda ve EOF'a ulaşılmadığında `read_chunk` yapılması; böylece kontrolsüz bellek büyümesinin önüne geçilmesi.
- [x] **Maksimum Kayıt Boyutu Üst Sınırı (`max_record_bytes`) (🔴 P1)**:
  - `delimited.nox`: `CsvOptions` ve `csv_options(...)` içine `max_record_bytes: int = 10485760` (10 MB) parametresinin eklenmesi.
  - `streaming.nox`: Kapanmayan tırnak veya aşırı uzun satırların belleği tüketmesini engellemek için tek satır boyutunun denetlenmesi ve aşım durumunda `CsvImportError` fırlatılması.
  - `max_record_bytes <= 0` durumunda `AtlasError` doğrulaması.
- [x] **Parametre Doğrulaması (`chunk_rows <= 0` ve `chunk_bytes <= 0`) (🔴 P1)**:
  - `StreamingCsvReader.__init__`: `chunk_rows <= 0` veya `chunk_bytes <= 0` durumunda sonsuz döngü veya ilerlememe hatalarını önlemek için açık `AtlasError` fırlatılması.
- [x] **Hata Durumunda Kaynak Temizliği Garantisi (`try...finally`) (🔴 P1)**:
  - `stream_count`, `stream_sum`, `stream_mean`, `stream_variance`, `stream_std`: Akış işleme döngülerinin `try...finally` bloğu ile sarılması; ara hesaplamada hata fırlatılsa dahi dosya tanıtıcısının (`reader.close()`) istisnasız kapatılmasının garanti altına alınması.
- [x] **Sayısal Olarak Kararlı Çevrimiçi Ortalama & İstatistikler (`stream_variance`, `stream_std`) (🟠 P2)**:
  - `stream_mean`: Yüzen nokta yuvarlama hatalarını ve iptal (cancellation) riskini önlemek için Welford / Chan çevrimiçi birleştirme algoritması.
  - `stream_variance` ve `stream_std`: Sınırsız büyüklükteki dosyalarda tek geçişli ($O(1)$ bellekli) kararlı varyans ve standart sapma fonksiyonlarının eklenmesi.
- [x] **Kapsamlı Sınır ve Kenar Durum Test Paketi (`tests/streaming_edge_cases_test.nox`) (🔴 P1 & 🟠 P2)**:
  - Tırnak (`"`) karakterinin chunk sınırında bölünmesi.
  - Kaçışlı tırnak (`""`) çiftinin chunk sınırında bölünmesi.
  - Çok satırlı tırnaklı alanların (`\r\n` içeren) birden fazla chunk'a yayılması.
  - `\r\n` satır sonunun chunk sınırında bölünmesi (`\r` chunk N, `\n` chunk N+1).
  - Yalnızca başlıktan oluşan CSV dosyası (0 veri satırı).
  - Başlıktan sonra yalnızca tek veri satırı içeren dosya.
  - Tamamen boş CSV dosyası (0 byte).
  - `chunk_rows=0` ve `chunk_bytes=0` parametre hata fırlatma doğrulamaları.
  - `max_record_bytes` aşım hata doğrulaması.
  - Hata anında dosyanın kapatıldığını doğrulayan kaynak temizliği testi.
  - Çevrimiçi varyans ve standart sapma doğruluk testi.
- [x] **Birim Testler, Dokümantasyon & Sürüm Yayın**:
  - `tests/streaming_edge_cases_test.nox` dahil 30 test paketinin tamamının `noxc test` ile geçmesi, `nox.json` sürümünün `0.7.1` yapılması, `CHANGELOG.md` güncellenmesi, git commit, push ve `noxc publish`.

---

## 🛠️ Sürüm 0.7.2 — Çok Karakterli Delimiter, UTF-8 Bayt Sınırları & Diferansiyel Doğrulama (Teknik İnceleme İyileştirmeleri)

- [x] **Çok Karakterli Delimiter'ların Chunk Sınırında Korunması (🔴 P1)**:
  - `streaming.nox`: `||`, `::`, `---` gibi çok karakterli ayraçların chunk sonuna denk gelmesi durumunda (`rem_len < delim_len` ve `delim.startswith(suffix)`), ayracın parçalanıp `cur_field` içine metin olarak karışmasını önlemek.
  - Eşleşen önekin `unconsumed_buffer` içinde saklanması ve sonraki chunk ile birleştirilerek $O(1)$ bayt seviyesinde kesintisiz ve doğru ayrıştırılması.
  - `delimiter = "||"` ve `chunk_bytes = 1` gibi en zorlayıcı tek-baytlık okuma senaryosunda doğrulanması.
- [x] **`max_record_bytes` Sınırının Gerçek UTF-8 Bayt Uzunluğuyla Ölçülmesi (🔴 P1)**:
  - `streaming.nox`: `len(s)` kod noktası yerine `nox.strings.byte_len(ch)` kullanarak çok baytlı UTF-8 karakterlerin (Türkçe karakterler `ğ, ü, ş, ö, ç, ı`, emojiler `😊, 🚀`) gerçek bellek bayt boyutunu tam ölçmek.
  - Çok baytlı karakterlerin kayıt sınırını sessizce aşmasının engellenmesi.
- [x] **Büyük `chunk_bytes` Durumunda `pending_rows` Bellek Sınırı (Bounded Rows) (🟠 P1)**:
  - `StreamingCsvReader.feed_chunk`: `len(self.pending_rows) >= target_rows` koşuluna ulaşıldığı anda chunk ayrıştırmasının durdurulması ve kalan karakterlerin `unconsumed_buffer` içine aktarılması.
  - Böylece `chunk_bytes = 100_000_000` gibi devasa chunk boyutlarında dahi bellekte gereksiz satır yığılması engellenir; bellek kullanımı her zaman kesin $O(\text{chunk\_rows})$ ile sınırlandırılır.
- [x] **Boş / Yetersiz Gözlem İstatistik Sözleşmesi (🟠 P2)**:
  - `stream_mean`: Hiçbir geçerli sayısal değer bulunmadığında (`running_n == 0`), `stats.nox` ile tam tutarlı biçimde `AggregationError("cannot compute mean: no non-missing numeric observations")` fırlatılması.
  - `stream_variance` ve `stream_std`: Yetersiz gözlemde (`is_sample` için $N < 2$, popülasyon için $N < 1$) sessizce `0.0` dönmek yerine `AggregationError` fırlatarak istatistiksel doğruluk sözleşmesine uyulması.
- [x] **Diferansiyel ve Rastgele Chunk Boyutlu Test Paketi (`tests/streaming_differential_test.nox`) (🟠 P2)**:
  - Aynı dosya `read_delimited()` (bellek içi standart ayrıştırıcı) ve `StreamingCsvReader()` ile okunarak karşılaştırılır.
  - `chunk_bytes = 1, 2, 3, 5, 7, 13, 31, 64, 1024` gibi 10 farklı chunk boyutunda sonuçların birebir aynı satır, sütun ve hücre değerlerini ürettiğinin doğrulanması.
  - Çok karakterli ayraç (`||`), çok satırlı tırnaklı alanlar, kaçışlı tırnaklar ve UTF-8 karakterlerin diferansiyel testi.
- [x] **Birim Testler, Dokümantasyon & Sürüm Yayın**:
  - `tests/streaming_differential_test.nox` dahil 31 test paketinin tamamının `noxc test` ile geçmesi, `nox.json` sürümünün `0.7.2` yapılması, `CHANGELOG.md` güncellenmesi, git commit, push ve `noxc publish`.

---

## 🛠️ Sürüm 0.7.3 — Delimiter Bayt Sayımı, EOF Sınırları & Property-Based Diferansiyel Doğrulama (Teknik İnceleme İyileştirmeleri)

- [x] **Çok Karakterli Delimiter Tüketiminde Tam Bayt Sayımı (🔴 P1)**:
  - `streaming.nox`: `feed_chunk` içerisinde `delim_len > 1` eşleştiğinde döngü başında eklenen ilk karakter (`ch`) baytına ek olarak kalan ayracın tüm baytlarının (`delim_bytes - ch_bytes`) `cur_record_bytes` sayacına eklenmesi.
  - `||||` gibi 4 baytlık ve daha uzun ayraçlarda kayıt sınırının doğru hesaplanması ve `max_record_bytes` aşımının eksiksiz yakalanması.
  - Escaped quote (`""`) ve EOF tampon boşaltma (`unconsumed_buffer`) işlemlerinde bayt sayacının ve sınır kontrollerinin tam işletilmesi.
- [x] **Gerçekçi Bellek Üst Sınırı Dokümantasyonu (🟠 P2)**:
  - `docs/capabilities.md`, `docs/api.md`, `CHANGELOG.md`: Bounded memory modelinin $O(B + R \times S + L)$ formülüyle ($B$: `chunk_bytes`, $R$: `chunk_rows`, $S$: ortalama satır büyüklüğü, $L \le \text{max\_record\_bytes}$) teknik olarak eksiksiz belgelenmesi.
- [x] **Kapsamlı EOF ve Delimiter Sınır Testleri (🟠 P2)**:
  - `tests/streaming_edge_cases_test.nox`:
    1. Dosyanın tek `|` ile bitmesi (`||` ayracında alan verisi olarak korunması).
    2. Dosyanın tam `||` ile bitmesi (sonda boş alan üretilmesi).
    3. Dosyanın kapanış tırnağından hemen sonra bitmesi (geçerli kayıt üretimi).
    4. Dosyanın tamamlanmamış tırnak ile bitmesi (`CsvImportError` fırlatılması).
    5. Dosyanın `\r` ile bitmesi (tek satır sonu, fazladan boş satır oluşmaması).
    6. Dosyanın `\r\n` ile bitmesi (tek satır sonu, fazladan boş satır oluşmaması).
- [x] **Diferansiyel Testte Şema Eşitliği & Property-Based Sentetik Test (🟠 P2)**:
  - `tests/streaming_differential_test.nox`: `verify_tables_identical` fonksiyonunun yalnızca hücre değerlerini değil; sütun adlarını, veri tiplerini (`data_type`), null olabilirliklerini (`nullable`) ve sıralarını birebir denetlemesi.
  - Programatik sentetik veri üreteci (`test_property_based_synthetic_generator`) ile 50 satırlık karma tipli, tırnaklı, `::` ayraçlı verinin 10 farklı chunk boyutunda (`1, 2, 3, 5, 7, 13, 27, 64, 256, 1024` bayt) `ParseOffline(x) == ParseStreaming(x, B)` değişmezliğinin doğrulanması.
- [x] **Streaming Şema Çıkarım (Schema Inference) Politikasının Belgelenmesi (🟠 P2)**:
  - `docs/capabilities.md` ve `docs/api.md`: İlk chunk üzerinden şema çıkarımı ve sonraki chunk'larda katı validasyon (`ShapeError`, `MissingValueError`) politikası ile `explicit_schema` kullanımının açıkça dokümante edilmesi.
- [x] **Birim Testler, Dokümantasyon & Sürüm Yayın**:
  - `tests/streaming_edge_cases_test.nox` ve `tests/streaming_differential_test.nox` dahil 31 test paketinin tamamının `noxc test` ile geçmesi, `nox.json` sürümünün `0.7.3` yapılması, `CHANGELOG.md` güncellenmesi, git commit, push ve `noxc publish`.

---

## 🏆 Sürüm 1.0.0 — Nihai Üretim Sürümü (Gelecek Hedef)

- [ ] Topluluk geri bildirimleri sonrası API dondurma onayı.
- [ ] Ek dosya biçimleri (Parquet / Arrow entegrasyonu).
- [ ] Çoklu iş parçacığı (multithreading) ve SIMD hızlandırmaları için Nox stdlib desteği çıktığında optimizasyonlar.
- [ ] `noxc publish` ile nihai v1.0.0 yayın kapısı.
