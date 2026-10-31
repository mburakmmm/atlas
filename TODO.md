# Atlas — Proje Geliştirme ve Sürümlendirme Yol Haritası (TODO.md)

Bu dosya, `AGENTS.md` bağlayıcı mühendislik sözleşmesi temel alınarak hazırlanmış; tamamlanan, sürmekte olan ve sonraki sürümlere planlanan tüm görevlerin merkezi takip ve sürüm yönetim belgesidir.

---

## 📌 Sürüm Özeti & Faz Durumu

| Sürüm | Faz / Odak | Durum | Test Durumu |
| :--- | :--- | :--- | :--- |
| **v0.1.0** | **Temel Tablo, CSV, Filtreleme, Sıralama, İstatistik, GroupBy, JSON** | ✅ **TAMAMLANDI & YAYINLANDI** | 12/12 Test Geçti (`noxc test`) |
| **v0.2.0** | **Tablo Birleştirme (Inner/Left Join) & Tablo Ekleme (Concat)** | ✅ **TAMAMLANDI & YAYINLANDI** | 14/14 Test Geçti (`noxc test`) |
| **v0.3.0** | **Özel Ayraçlar (TSV, `;`), Kartiller (Quantiles) & Izgara Gösterimi** | ✅ **TAMAMLANDI & YAYINLANDI** | 17/17 Test Geçti (`noxc test`) |
| **v0.4.0** | **Feather (SQL ORM) & Noxen (Validasyon) İsteğe Bağlı Adaptörleri** | ⏳ **PLANLANDI (Sıradaki)** | — |
| **v1.0.0** | **Stabil Genel API Dondurma, Tam Belgelendirme & Üretim Onayı** | ⏳ **HEDEF SÜRÜM** | — |

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

## 🔌 Sürüm 0.4.0 — Ekosistem Adaptörleri

- [ ] **Feather Entegrasyonu (SQL Database Mapper)**:
  - Feather `SelectQuery` sonuçlarının doğrudan Atlas `Table` nesnesine dönüştürülmesi (`table_from_query(session, query)`).
  - Atlas `Table` verilerinin Feather üzerinden veritabanına toplu eklenmesi (`table.to_sql(session, table_name, if_exists="append")`).
- [ ] **Noxen Şema Doğrulama Entegrasyonu**:
  - Tablo sütunlarının ve hücrelerinin Noxen validator kuralları ile otomatik doğrulanması.
- [ ] **Birim Testler & Dokümantasyon**:
  - `tests/feather_adapter_test.nox`
  - `docs/integrations.md`

---

## 🏆 Sürüm 1.0.0 — Üretim Sürümü (Release Gate)

- [ ] Genel API semantiğinin ve metod imzalarının dondurulması (SemVer garantisi).
- [ ] Tüm dokümantasyonun ve API referansının eksiksiz doğrulanması.
- [ ] Çapraz platform ve backend doğrulama testleri (macOS, Linux, Windows; LLVM & QBE).
- [ ] Büyük veri kümesi performans bütçelerinin doğrulanması (100.000+ satır).
- [ ] Güvenlik denetimi ve son kontrol listesi onayı.
- [ ] `noxc publish` ile nihai v1.0.0 indeks yayını.
