# Kartiller, Yüzdelikler ve Aykırı Değer Tespiti (Quantiles & Outliers)

Atlas, istatistiksel analizlerde veri dağılımını anlamak ve gürültülü aykırı değerleri filtrelemek için standart doğrusal enterpolasyonlu kartil ve IQR (Interquartile Range) fonksiyonları sunar.

---

## 1. Kartil ve Yüzdelik Hesabı (`quantile`)

Sıralı gözlemler üzerinde $0.0 \le q \le 1.0$ aralığında sürekli doğrusal enterpolasyon (NumPy / R Type-7 standardı) uygulanır:

$$pos = q \times (n - 1)$$
$$idx = \lfloor pos \rfloor, \quad fraction = pos - idx$$
$$Q(q) = x_{idx} + fraction \times (x_{idx+1} - x_{idx})$$

```nox
import atlas
from atlas import Table

# t bir Table nesnesi olsun:
q0: float   = t.quantile("score", 0.0)   # Minimum (stat_min ile uyumlu)
q25: float  = t.quantile("score", 0.25)  # Q1 (1. Çeyrek)
q50: float  = t.quantile("score", 0.50)  # Medyan (stat_median ile uyumlu)
q75: float  = t.quantile("score", 0.75)  # Q3 (3. Çeyrek)
q100: float = t.quantile("score", 1.0)   # Maksimum (stat_max ile uyumlu)
```

---

## 2. Çeyrekler Açıklığı (IQR - Interquartile Range)

$$IQR = Q3 - Q1 = \text{quantile}(0.75) - \text{quantile}(0.25)$$

```nox
iqr_val: float = t.iqr("score")
```

---

## 3. Aykırı Değer Filtreleme (`filter_outliers_iqr`)

Veri kümesindeki aşırı uç değerleri Tukey'in çit kuralı (Tukey's Fences) yöntemiyle otomatik olarak süzebilirsiniz:

$$\text{Alt Sınır} = Q1 - (k \times IQR)$$
$$\text{Üst Sınır} = Q3 + (k \times IQR)$$

Varsayılan çarpan $k = 1.5$ tipik aykırı değerleri yakalar ($k = 3.0$ aşırı uç değerler için tercih edilir).

```nox
# reading sütunundaki aykırı satırları temizle
clean_table: Table = t.filter_outliers_iqr("reading", multiplier=1.5, keep_missing=True)
```

### Parametreler
- `col_name` (`str`): Aykırı analizi yapılacak sayısal sütun adı.
- `multiplier` (`float = 1.5`): IQR tolerans katsayısı.
- `keep_missing` (`bool = True`): Eksik (`null`) hücreye sahip satırların korunup korunmayacağı (`drop_missing` ayrı çağrılabilir).
