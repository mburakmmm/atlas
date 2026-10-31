# Özel Ayraçlar ve TSV Desteği (Custom Delimiters & TSV)

Atlas, standart virgül (`,`) ayracının yanı sıra sekmeyle ayrılmış değerler (**TSV - Tab-Separated Values**), noktalı virgül (`;`), dikey çizgi (`|`) veya kullanıcı tanımlı herhangi bir ayraç karakterini destekler.

Tüm ayraç işlemleri **RFC 4180** uyumlu tırnak koruma (escaping) ve çok satırlı alan kurallarına tam olarak uyar.

---

## 1. TSV Okuma ve Yazma

Sekmeyle ayrılmış metinler doğrudan `read_tsv` ve `to_tsv` fonksiyonları ile işlenir:

```nox
import atlas
from atlas import read_tsv, to_tsv, Table

tsv_data: str = "name\tage\tcity\nAlice\t30\tAnkara\nBob\t25\tIstanbul\n"
t: Table = read_tsv(tsv_data)

print(t.cell(0, 0).as_str()) # "Alice"
print(to_tsv(t))
```

### Dosya G/Ç Kolaylığı

```nox
from atlas import read_tsv_file, write_tsv_file

t: Table = read_tsv_file("data/inventory.tsv")
write_tsv_file(t, "output/clean_inventory.tsv", overwrite=True)
```

---

## 2. Özel Ayraçlı Metinler (`read_delimited` & `to_delimited`)

Avrupa standartlarında sıkça CSV ayracı olarak kullanılan noktalı virgül (`;`) veya günlük kütüklerinde kullanılan dikey çizgi (`|`) gibi ayraçlar:

```nox
from atlas import read_delimited, to_delimited

data: str = "product;price;in_stock\nLaptop;1200;true\nMouse;25;true\n"
t: Table = read_delimited(data, delimiter=";")

# Dışa aktarma
sc_text: str = to_delimited(t, delimiter=";")
```

### Dosya G/Ç:

```nox
from atlas import read_delimited_file, write_delimited_file

t: Table = read_delimited_file("data/export.txt", delimiter="|")
write_delimited_file(t, "output/export.psv", delimiter="|", overwrite=True)
```

---

## 3. RFC 4180 Uyumluluk Kuralları

Özel ayraç ayrıştırıcısı aşağıdaki durumları eksiksiz çözer:
- **Alıntı İçindeki Ayraçlar**: `"Item; with semicolon";100` gibi alanlarda tırnak içi ayraçlar alan bölücü olarak algılanmaz.
- **Çift Tırnak Kaçışı**: `"He said ""hi"""` ifadesi `He said "hi"` olarak çözülür.
- **Çok Satırlı Alanlar**: Tırnak içine alınmış `\n` veya `\r\n` satır sonları hücre metninin bir parçası olarak korunur.
- **Formül Enjeksiyonu Koruması**: `safe_spreadsheet=True` parametresi ile Excel/LibreOffice açılışlarında formül çalıştırmayı önlemek için `=`, `+`, `-`, `@` ile başlayan hücrelere tek tırnak (`'`) öneki eklenir.
