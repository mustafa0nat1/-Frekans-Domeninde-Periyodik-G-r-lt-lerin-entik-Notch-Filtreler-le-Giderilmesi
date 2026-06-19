# Frekans Domeninde Periyodik Gürültülerin Çentik (Notch) Filtreler İle Giderilmesi

## 📌 Proje Hakkında
[cite_start]Görüntü elde etme ve iletim süreçlerinde donanımsal kısıtlılıklar, elektromanyetik girişimler veya sensör hataları sebebiyle görüntülerde çeşitli bozulmalar meydana gelmektedir[cite: 649]. [cite_start]Bu bozulma türlerinden biri olan periyodik gürültüler, uzamsal domende birbirini tekrar eden geometrik veya sinüzoidal desenler olarak gözlemlenir[cite: 650]. 

[cite_start]Medyan veya ortalama filtreler gibi uzamsal filtreleme teknikleri, bu tür sistematik hataları gidermekte yetersiz kalmakta ve görüntüdeki yüksek frekanslı yapısal detayların (kenarların) kaybolmasına yol açmaktadır[cite: 651, 652]. 

[cite_start]Frekans domeni analizi, periyodik gürültülerin karakterizasyonu ve filtrelenmesi için en etkili metodolojik yaklaşımdır[cite: 653]. [cite_start]İki Boyutlu Ayrık Fourier Dönüşümü (2D-DFT) kullanılarak görüntü frekans uzayına aktarıldığında, periyodik gürültüler merkez frekans etrafında simetrik, yüksek enerjili tepe noktaları (spike) olarak belirginleşir[cite: 654]. 

[cite_start]Bu projede, sentetik olarak üretilmiş üç yönlü periyodik gürültülerin İdeal Çentik (Notch) filtresi kullanılarak hedeflenen frekans koordinatlarında sönümlendirilmesi ve görüntünün onarılması gerçekleştirilmiştir[cite: 655, 656].

---

## ⚙️ Algoritmik İş Akışı
[cite_start]Proje kapsamında izlenen iş akışı temel olarak 5 ardışık aşamadan oluşmaktadır[cite: 658]:

1. [cite_start]**Ön İşleme:** Görüntünün matris formatında okunması, gri tonlamaya çevrilmesi, **512x512** boyutlarına çözünürlüğün zorlanması ve genlik değerlerinin **[0,1]** aralığına normalize edilmesi[cite: 659, 669].
2. [cite_start]**Gürültü Modellemesi:** Belirlenen genlik ($A=0.15$) ve frekans ($f=0.1$) parametrelerine sahip yatay, dikey ve çapraz (diyagonal) sinüzoidal gürültülerin matematiksel olarak oluşturulup orijinal görüntüye entegre edilmesi[cite: 660, 671].
3. [cite_start]**Frekans Dönüşümü:** Hızlı Fourier Dönüşümü (FFT) algoritması uygulanarak görüntünün frekans spektrumunun elde edilmesi ve sıfır frekans (DC) bileşeninin merkeze kaydırılması[cite: 661].
4. [cite_start]**Filtreleme:** Logaritmik spektrum üzerinde gürültüye sebep olan koordinatların tespit edilerek bu noktalardaki enerjiyi sıfırlayan Çentik (Notch) maskesinin tasarlanması ve spektruma uygulanması[cite: 662].
5. [cite_start]**Görüntü Geri Çatımı:** Ters Fourier Dönüşümü (IFFT) ile frekans domeninden uzamsal domene geri dönülerek onarılmış görüntünün elde edilmesi[cite: 663].

---

## 🛠️ Uygulama ve Matematiksel Model

### Gürültü Oluşturma
[cite_start]Görüntüye eklenen periyodik kafes gürültüsü, üç farklı sinüzoidal dalganın birleşimiyle üretilmiştir[cite: 670, 681]. [cite_start]Koordinat matrisleri üretildikten sonra şu bileşenler eklenmiştir[cite: 670]:
* [cite_start]**Dikey Gürültü:** $genlik \times \sin(2\pi \times freq \times X)$ [cite: 679]
* [cite_start]**Yatay Gürültü:** $genlik \times \sin(2\pi \times freq \times Y)$ [cite: 679]
* [cite_start]**Diyagonal Gürültü:** $genlik \times \sin(2\pi \times freq \times X + 2\pi \times freq \times Y)$ [cite: 677, 678]

### Çentik Maskesi Hesaplama
[cite_start]Görüntü boyutu $N=512$ piksel ve gürültü frekansı $f=0.1$ olduğundan, gürültü noktalarının spektrum merkezinden olan sapma uzaklığı deterministik olarak şu şekilde hesaplanır[cite: 752, 753]:

$$\text{sapma} = N \times f = 512 \times 0.1 = 51 \text{ piksel}$$

[cite_start]Merkezden 51 piksel uzaklıkta konumlanan 6 simetrik gürültü odağı için Öklid uzaklığı formülü kullanılarak yarıçapı $r=6$ piksel olan bir İdeal Çentik maskesi oluşturulmuştur[cite: 749, 754, 758]:

$$\text{mesafe} = \sqrt{(satir - nokta_y)^2 + (sutun - nokta_x)^2}$$

[cite_start]Bu yarıçapın içerisinde kalan frekans bileşenlerinin enerjisi 0'a, dışındakiler ise 1'e eşitlenmiştir[cite: 756].

---

## 💻 Kaynak Kod Çalışması

```python
import cv2
import numpy as np
import matplotlib.pyplot as plt

# 1. Görüntü Okuma ve Ön İşleme
img = cv2.imread('goruntu1.jpeg', 0)
img_512 = cv2.resize(img, (512, 512))
I = img_512 / 255.0

# 2. Sinüzoidal Gürültü Entegrasyonu
freq = 0.1
genlik = 0.15
x = np.arange(0, 512)
y = np.arange(0, 512)
X, Y = np.meshgrid(x, y)

diagonal = genlik * np.sin(2 * np.pi * freq * X + 2 * np.pi * freq * Y)
dikey = genlik * np.sin(2 * np.pi * freq * X)
yatay = genlik * np.sin(2 * np.pi * freq * Y)
I_noisy = I + diagonal + dikey + yatay

# 3. İleri Fourier Dönüşümü (FFT)
F = np.fft.fft2(I_noisy)
F_merkez = np.fft.fftshift(F)

# 4. İdeal Çentik Filtresi Maskesi Oluşturma
maske = np.ones((512, 512))
yari_cap = 6
merkez = 256
sapma = 51

noktalar = [
    (merkez + sapma, merkez + sapma),
    (merkez - sapma, merkez - sapma),
    (merkez, merkez + sapma),
    (merkez, merkez - sapma),
    (merkez + sapma, merkez),
    (merkez - sapma, merkez)
]

for satir in range(512):
    for sutun in range(512):
        for (nokta_y, nokta_x) in noktalar:
            mesafe = np.sqrt((satir - nokta_y)**2 + (sutun - nokta_x)**2)
            if mesafe <= yari_cap:
                maske[satir, sutun] = 0

# Maskeyi Spektruma Uygulama
F_filtreli = F_merkez * maske

# 5. Ters Fourier Dönüşümü (IFFT) ile Görüntü Onarımı
F_ters_kaydirma = np.fft.ifftshift(F_filtreli)
I_sonuc = np.fft.ifft2(F_ters_kaydirma)
I_sonuc = np.real(I_sonuc)

# Sonuçları Görselleştirme
plt.figure(figsize=(10,5))
plt.subplot(1, 2, 1)
plt.imshow(I_noisy, cmap='gray')
plt.title("Gürültülü Hal")
plt.subplot(1, 2, 2)
plt.imshow(I_sonuc, cmap='gray')
plt.title("Temizlenmiş Hal")
plt.show()
