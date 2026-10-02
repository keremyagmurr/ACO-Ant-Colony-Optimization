# 🐜 Karınca Kolonisi Optimizasyonu (ACO) ile Ağ Yönlendirme

> **Bilgisayar Ağları** dersi kapsamında geliştirilmiştir.
> 250 düğümlü bir ağda, gecikme, güvenilirlik ve kaynak kullanımını birlikte dikkate alarak kaynak düğümden hedef düğüme en uygun yolu bulan, Python ile sıfırdan yazılmış **Karınca Kolonisi Optimizasyonu (Ant Colony Optimization)** uygulaması.

---

## 📌 Proje Hakkında

Bir bilgisayar ağında iki düğüm arasındaki en iyi yol yalnızca "en kısa" yol değildir. Gecikme, bağlantı ve düğümlerin güvenilirliği ile bant genişliği gibi birden fazla ölçüt birlikte değerlendirilmelidir.

Bu projede ağ, bir JSON dosyasından okunur ve **ACO** algoritması ile seçilen kaynak (S) ve hedef (D) düğümleri arasındaki en düşük maliyetli yol aranır. Repo, proje kapsamında geliştirdiğim ACO algoritmasını ve sonuç görselleştirmesini içerir. Çıktıdaki metrikler, aynı ağ dosyasını kullanan arayüz ve diğer algoritma çalışmalarıyla karşılaştırılabilecek biçimde hazırlanmıştır.

---

## 🧠 Algoritma Nasıl Çalışır?

1. **Başlangıç:** Ağdaki tüm bağlantılara eşit miktarda (1.0) feromon atanır.
2. **Yol oluşturma:** Her karınca kaynak düğümden başlar. Her adımda, daha önce ziyaret etmediği komşu düğümlerden birini feromon miktarına ve bağlantının uygunluğuna göre olasılıkla seçer. Hedefe ulaşamayan (çıkmaza giren) karıncanın yolu elenir.
3. **Maliyet hesabı:** Tamamlanan her yol, aşağıdaki maliyet fonksiyonuyla puanlanır.
4. **En iyi yolun takibi:** Şimdiye kadar bulunan en düşük maliyetli yol ve bulunduğu iterasyon kaydedilir.
5. **Buharlaşma:** Her iterasyon sonunda tüm feromonların bir kısmı silinir.
6. **Pekiştirme:** En iyi yolun bağlantılarına, yolun maliyetiyle ters orantılı (`1 / maliyet`) feromon eklenir.
7. Belirlenen iterasyon sayısı boyunca 2-6 adımları tekrarlanır.

**Seçim olasılığı:** bir `u` düğümündeki karınca, komşu `v` düğümünü şu değerle orantılı bir olasılıkla seçer:

```
P(u → v)  ∝  feromon(u, v)^alpha  ×  sezgisel(u, v)^beta
```

**Sezgisel değer:** Bağlantı ne kadar iyiyse (düşük gecikme, yüksek bant genişliği, yüksek güvenilirlik) değeri o kadar yüksektir:

```
sezgisel(u, v) = 1 / ( gecikme + 1000 / bant_genişliği − ln(güvenilirlik) )
```

---

## 🧮 Maliyet Fonksiyonu

Bir yolun toplam maliyeti üç bileşenin ağırlıklı toplamıdır:

```
TotalCost = w_delay × Delay  +  w_reliability × (−ln Reliability)  +  w_resource × Resource
```

| Bileşen | Hesaplama |
|---------|-----------|
| **Delay** (gecikme) | Yoldaki bağlantı gecikmeleri + düğümlerin işlem gecikmeleri toplamı |
| **Reliability** (güvenilirlik) | Yoldaki bağlantı ve düğüm güvenilirliklerinin **çarpımı** |
| **Resource** (kaynak) | Her bağlantı için `1000 / bant_genişliği` toplamı (bant genişliği arttıkça maliyet düşer) |

Güvenilirlik çarpım olduğu için `−ln(Reliability)` ile toplanabilir bir maliyete dönüştürülür. Varsayılan ağırlıklar: `w_delay = 0.4`, `w_reliability = 0.3`, `w_resource = 0.3`.

---

## ⚙️ Parametreler

| Parametre | Varsayılan | Açıklama |
|-----------|-----------|----------|
| `num_ants` | 25 | Her iterasyonda yol arayan karınca sayısı |
| `iterations` | 40 | İterasyon sayısı |
| `alpha` | 1 | Feromonun seçim üzerindeki etkisi |
| `beta` | 2 | Sezgisel değerin (bağlantı kalitesi) etkisi |
| `evaporation` | 0.3 | Her iterasyonda buharlaşan feromon oranı |
| `w_delay` | 0.4 | Gecikme ağırlığı |
| `w_reliability` | 0.3 | Güvenilirlik ağırlığı |
| `w_resource` | 0.3 | Kaynak (bant genişliği) ağırlığı |
| `S`, `D` | 20, 200 | Kaynak ve hedef düğüm numaraları |

---

## 📊 Örnek Sonuç

**Ağ:** 250 düğüm, 12.426 bağlantı | **Kaynak → Hedef:** 20 → 200

| Metrik | Değer |
|--------|-------|
| En iyi yol | `20 → 211 → 207 → 200` |
| Toplam maliyet | 8.8846 |
| Gecikme | 18.24 |
| Güvenilirlik | 0.8716 (≈ %87) |
| Kaynak maliyeti | 5.16 |
| En iyi sonucun bulunduğu iterasyon | 26 / 40 |

**Yakınsama:** En iyi maliyet ilk iterasyonda 155.97 iken 3. iterasyonda 16.89'a, 11. iterasyonda 9.47'ye ve 26. iterasyonda 8.88'e düşmüştür.

> ACO sezgisel ve rastgelelik içeren bir yöntemdir. Sonuçlar her çalıştırmada farklı olabilir ve global optimum garanti edilmez. Tekrarlanabilir sonuç için koda `random.seed(42)` eklenebilir.

### Görselleştirme

Bulunan yol `matplotlib` ve `networkx` ile çizilir: kaynak düğüm **yeşil**, hedef düğüm **kırmızı**, bulunan yol **kalın mavi** çizgidir.

<!-- Görselleştirme çıktısının ekran görüntüsünü images/ klasörüne koyup aşağıdaki satırı aç:
![ACO sonuç görselleştirmesi](images/aco_sonuc.png)
-->

---

## 📁 Dosyalar

| Dosya | Açıklama |
|-------|----------|
| `ACO_Algoritması.ipynb` | Ağ yükleme, maliyet fonksiyonu, ACO algoritması ve görselleştirme |
| `network_250_nodes.json` | 250 düğümlü ağ verisi (düğüm ve bağlantı özellikleri) |
| `README.md` | Proje açıklaması |

**Ağ dosyası biçimi:** JSON dosyası `edges` (bağlantı listesi), `node_properties` (düğümlerin `processing_delay` ve `reliability` değerleri) ve `link_properties` (bağlantıların `delay`, `reliability` ve `bandwidth` değerleri) alanlarını içerir.

---

## 🚀 Çalıştırma

```bash
git clone https://github.com/keremyagmurr/Bilgisayar_aglari_ACO.git
cd Bilgisayar_aglari_ACO
pip install networkx numpy matplotlib jupyter
jupyter notebook
```

`ACO_Algoritması.ipynb` dosyasını açıp hücreleri sırayla çalıştırın. `network_250_nodes.json` dosyasının notebook ile aynı klasörde olması gerekir.

**Farklı bir senaryo denemek için:**

```python
S = 5            # kaynak düğüm
D = 150          # hedef düğüm

best_path, metrics, best_iter = run_aco(
    network, G, S, D,
    num_ants=50,
    iterations=100,
    alpha=1,
    beta=3,
    evaporation=0.2
)
```

---

## 🛠️ Kullanılan Teknolojiler

- Python 3
- Jupyter Notebook
- NetworkX (ağ/graf yapısı)
- NumPy
- Matplotlib (görselleştirme)

---

## 👤 Geliştirici

**Kerem Yağmur** — Bartın Üniversitesi, Bilgisayar Mühendisliği
GitHub: [@keremyagmurr](https://github.com/keremyagmurr)
