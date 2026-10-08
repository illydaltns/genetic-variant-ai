# Bitirme Projesi Çalışma Günlüğü

## Tarih: 08.10.2026
## Çalışılan Konu: ClinVar Veri Seti Keşfi ve Ön İnceleme

### Bugün Ne Yaptım?
- ClinVar `variant_summary.txt.gz` veri seti üzerinde bellek verimliliğini gözeterek ilk kapsamlı keşifçi veri analizi (EDA) gerçekleştirildi.
- Dosya yapısı, veri tipleri, GRCh37 / GRCh38 assembly dağılımları, varyant tipleri ve VCF alanlarının doluluk durumları incelendi.
- `ClinicalSignificance` etiket yapısı, özel eksik veri gösterimleri ve dış veri kaynaklarıyla eşleştirmede kullanılabilecek bileşik anahtar adayları analiz edildi.
- Veri üzerinde henüz silme, filtreleme veya preprocessing işlemi uygulanmadı; bulgular `notebooks/01_clinvar_exploration.ipynb` içinde belgelendi.

### Ne Öğrendim?
- `ClinicalSignificance` sütununun model eğitim etiketlerinin oluşturulmasında temel kaynak olarak kullanılabileceği görüldü.
- Aynı `AlleleID` değerlerinin çoğunun GRCh37 ve GRCh38 için ayrı satırlarda tutulduğu, bu nedenle tekrarların önemli bölümünün doğrudan veri hatası olmadığı anlaşıldı.
- `Uncertain significance` (VUS) kayıtlarının veri setinin yarısından fazlasını oluşturduğu ve bunun ayrı bir biyolojik sınıftan çok klinik/deneysel kanıt yetersizliğini temsil ettiği görüldü.
- ClinVar'da eksik bilgilerin yalnızca `NaN` ile değil; `na`, `-`, `not provided` ve `-1` gibi özel placeholder değerlerle de tutulduğu belirlendi.
- Varyant eşleştirmesinde kullanılabilir Ref/Alt nükleotid bilgilerinin büyük ölçüde `ReferenceAlleleVCF` ve `AlternateAlleleVCF` alanlarında bulunduğu gözlendi.

### Önemli Bulgular
- **Genel Boyut ve Assembly:** Veri seti 43 sütun ve yaklaşık 9.22 milyon satırdan oluşmaktadır. GRCh37 (%50.21) ve GRCh38 (%49.63) kayıtları yaklaşık eşit dağılmıştır.
- **Varyant Türleri ve VCF Doluluğu:** Verinin yaklaşık %80'ini SNV kayıtları oluşturmaktadır. SNV kayıtlarında VCF koordinat alanları büyük ölçüde doluyken, bazı CNV ve yapısal varyantlarda bu alanların eksik olduğu görülmüştür.
- **Nükleotid Alanları:** Klasik `ReferenceAllele` ve `AlternateAllele` sütunları çok yüksek oranda `na` içerirken, kullanılabilir nükleotid bilgilerinin büyük ölçüde VCF sütunlarında bulunduğu gözlenmiştir.
- **Ham Sınıf Dağılımı:**
  - `Uncertain significance`: %51.64
  - `Likely benign`: %24.64
  - `Benign`: %4.29
  - `Pathogenic`: %4.37
  - `Likely pathogenic`: %2.85
  - `Conflicting classifications of pathogenicity`: %3.86
  - Diğer bileşik veya etiketsiz sınıflar: yaklaşık %8.35
- **Sınıf Dengesizliği:** Ham ClinVar kayıtlarında Benign yönündeki aday etiketlerin sayısı Pathogenic yönündeki aday etiketlerden yaklaşık 3.7 kat fazladır. Nihai oran, assembly ve etiket filtreleri belirlendikten sonra yeniden hesaplanacaktır.
- **Tekrarların Doğası:** Tek assembly seçildiğinde tekrar oranının büyük ölçüde azaldığı görülmüştür. Kalan tekrarların bir kısmının PAR bölgeleri veya yapısal varyantlar gibi biyolojik/temsil kaynaklı nedenlerle ilişkili olabileceği düşünülmektedir; bu durum ayrıca doğrulanmalıdır.
- **Bileşik Anahtar:** İncelenen GRCh38 alt kümesinde, VCF alanları eksiksiz olan kayıtlar için `Chromosome + PositionVCF + ReferenceAlleleVCF + AlternateAlleleVCF` bileşik anahtarında çakışma gözlenmemiştir.

### Verilen Kararlar
- EDA aşamasında veri üzerinde silme, kalıcı filtreleme veya sınıf birleştirme yapılmamasına karar verilmiştir.

### Karar Gerekçesi
- Kesin assembly seçimi, gnomAD ve dbNSFP veri setlerinin kullandığı referans genom sürümleri incelenmeden yapılmamalıdır.
- Etiket konsolidasyonu; ClinVar sınıf tanımları, review/evidence düzeyi, ilgili literatür ve gerektiğinde ACMG/AMP sınıflandırma çerçevesi dikkate alınarak belirlenmelidir.
- VUS ve conflicting kayıtların nasıl ele alınacağı netleşmeden bu kayıtların eğitim verisinden çıkarılması veya sınıflandırılması doğru olmayacaktır.

### Henüz Cevaplanmamış Sorular
- gnomAD ve dbNSFP hangi assembly sürümlerini kullanmaktadır?
- Hangi assembly seçildiğinde entegrasyon kaybı en aza indirilebilir?
- VUS kayıtları eğitim dışında tutulup daha sonra olasılık temelli değerlendirme için mi kullanılmalıdır?
- `Conflicting classifications of pathogenicity` kayıtları `ReviewStatus` ve `NumberSubmitters` gibi alanlara göre filtrelenebilir mi?
- gnomAD ve dbNSFP koordinat formatları ClinVar VCF alanlarıyla birebir uyumlu mudur?
- `chr1` / `1` ve 0-based / 1-based koordinat farkları entegrasyonda problem oluşturacak mıdır?

### Takıldığım / Dikkat Gerektiren Noktalar
- Veri setinin büyük olması nedeniyle bellek kullanımını azaltmak için `nrows`, `usecols` ve chunk tabanlı okuma stratejileri gerekmektedir.
- `na`, `-`, `not provided` ve `-1` gibi placeholder değerler standart `isna()` kontrolleriyle yakalanmadığı için özel missing-value kontrolleri gerekmektedir.

### Bir Sonraki Adım
- gnomAD veri yapısı, sürümü ve assembly bilgisinin incelenmesi.
- dbNSFP veri yapısı, sürümü ve assembly bilgisinin incelenmesi.
- ClinVar, gnomAD ve dbNSFP arasında en az veri kaybıyla kullanılabilecek ortak assembly'nin belirlenmesi.
- Dış kaynak uyumu netleştikten sonra ClinVar için preprocessing kurallarının oluşturulması.