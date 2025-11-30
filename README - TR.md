# SUAS 2024 – Otonom Kargo Dağıtım İHA Sistemi

Bu proje, **AUVSI Student Unmanned Aerial Systems (SUAS) 2024** yarışması için geliştirilmiş otonom bir kargo dağıtım İHA sistemini kapsamaktadır.  
Platform, tam otonom bir görev icra eder: önceden planlanmış bir rotada uçar, yerdeki alfasayısal işaretleyicileri (harfler ve rakamlar) tespit eder, motor kontrollü bir vinç mekanizması ile kargo bırakma işlemini gerçekleştirir ve normal koşullarda herhangi bir manuel müdahaleye ihtiyaç duymadan eve dönüş yapar.

Bu depo, projenin arkasındaki **sistem mimarisini, edge AI hattını ve entegrasyon çalışmalarını** belgelemek için hazırlanmıştır.

🎥 **Promotional Video:** [Watch on YouTube](https://youtu.be/MoKNS4MnHKc?si=5vaya1-DzLdw4j8K)

🎥 **Promotional Video2:** [Watch on YouTube](https://youtu.be/0MTjb65SmeQ?si=WlFSHK1B2cfJrwUZ)

---

## 1. Yarışma Bağlamı

**AUVSI Student Unmanned Aerial Systems (SUAS)**, üniversite takımlarının aşağıdaki kabiliyetlere sahip bir İHA sistemi geliştirmesini bekleyen uluslararası bir yarışmadır:

- Otonom uçuş ve seyrüsefer,
- Üzerindeki faydalı yük sensörleri ile uzaktan algılama,
- Yarışma kılavuzunda tanımlanan görevlerin icra edilmesi.

Ekibimiz **SUAS 2024**’e katılmış, gerekli eleme aşamalarını geçmiş ve resmi video raporunu sunmuştur. Otonom kargo dağıtımı senaryosunu başarıyla göstermiş ve SUAS 2024 için katılım sertifikası almaya hak kazanmıştır.

---

## 2. Proje Genel Bakış

Bu projenin hedefi, aşağıdaki kabiliyetlere sahip **otonom bir kargo dağıtım İHA’sı** geliştirmektir:

- Yer kontrol istasyonundan yüklenen görev/waypoint setine göre kalkış yapması ve görevi otonom şekilde uçması,
- Yerde boyanmış harf ve rakamları (kargo bırakma bölgelerini temsil eden işaretleyiciler) tespit etmesi,
- Tespit edilen hedeflerin üzerinde durması veya konum tutması,
- Motorlu vinç mekanizması ve misina (monofilament) kullanarak kargoyu aşağı indirip bırakması,
- Misina’yı tekrar topladıktan sonra bir sonraki hedefe devam etmesi,
- Tüm teslimatlar tamamlandığında eve (home konumuna) otonom olarak dönüp iniş yapması.

Bu süreçte tüm algılama ve karar verme adımları, havadayken **NVIDIA Jetson Orin** üzerinde çalışan, **Raspberry Pi High Quality Camera** ve **YOLOv8** tabanlı bir model kullanılarak **tamamen yerleşik (onboard)** yürütülmektedir. Uçuş kontrolcüsü olarak **Pixhawk Cube Orange** kullanılmakta, görev planlama, telemetri takibi ve güvenlik denetimi için yer kontrol istasyonu tarafında **Mission Planner** devreye girmektedir.

---

## 3. Sistem Mimarisi

Sistem yüksek seviyede aşağıdaki bileşenlerden oluşur:

- **Gövde ve Kargo Mekanizması**
  - Otonom uçuşa uygun şekilde yapılandırılmış çok rotorlu (multirotor) bir hava aracı.
  - Kargo, İHA’nın altında taşınan ve **misina** üzerinden indirilen bir yük (örneğin su şişesi) olarak tasarlanmıştır.
  - Kargo, **motor kontrollü bir vinç mekanizması** ile yukarı/aşağı hareket ettirilir; vinç motoru uçuş kontrolcüsü tarafından sürülür.

- **Uçuş Kontrolcüsü ve Otopilot**
  - **Pixhawk Cube Orange** üzerinde çalışan standart bir otopilot yazılımı (örneğin ArduPilot/PX4).
  - Düşük seviyeli stabilizasyon, waypoint takibi ve otonom uçuş modlarının icrasından sorumludur.
  - Telemetri verilerini (konum, attitude, hız, durum bilgileri) hem Jetson’a hem de yer kontrol istasyonuna sağlar.

- **Yerleşik Hesaplama Birimi ve Kamera**
  - Yerleşik bilgisayar: **NVIDIA Jetson Orin**.
  - Görüntü sensörü: **Raspberry Pi High Quality Camera**, genellikle aşağı bakan konfigürasyonda.
  - Jetson, kameradan gelen akışı alır, gerçek zamanlı nesne tespiti yapar ve kargo bırakma kararlarını uçuş kontrolcüsü ile koordine eder.

- **Algılama (Perception) ve Görev Mantığı**
  - Yerdeki alfasayısal işaretleyicileri (harf ve rakamlar) tespit etmek üzere eğitilmiş **YOLOv8** modeli.
  - Jetson üzerinde çalışan bir görev yönetimi katmanı:
    - Tespitleri izler,
    - Uçağın hangi durumda olduğunu değerlendirir,
    - Ne zaman durulacağı, bekleme yapılacağı ve kargo bırakma döngüsünün tetikleneceği konusunda karar verir.

- **Yer Kontrol İstasyonu ve Güvenlik**
  - **Mission Planner**:
    - Görev/waypoint yükleme,
    - Telemetri (konum, batarya, uçuş modu, durum) izleme,
    - Otonom görev takibi için kullanılır.
  - Manuel RC override ve standart failsafe mekanizmaları (link kaybı, batarya kritik seviye, jeofence) güvenlik için konfigüre edilmiştir.

---

## 4. Donanım ve Yazılım Yığını

**Donanım**

- **Yerleşik Hesaplama Birimi:** NVIDIA Jetson Orin (Linux tabanlı edge AI platformu).
- **Uçuş Kontrolcüsü:** Pixhawk Cube Orange (Cube sınıfı otopilot).
- **Kamera:** Raspberry Pi High Quality Camera.
- **Kargo Mekanizması:** Motorlu vinç + misina (monofilament) ile su şişesi gibi yüklerin indirilmesi ve geri toplanması.
- **Yer Segmenti:**
  - Mission Planner çalıştıran yer kontrol bilgisayarı,
  - Manuel override ve güvenlik için RC verici/alıcı.

**Yazılım**

- **Otopilot Yazılımı:** ArduPilot/PX4 (otonom multirotor uçuşu ve payload kontrolü için konfigüre edilmiştir).
- **Yer Kontrol İstasyonu:** Görev yükleme, telemetri ve parametre ayarları için Mission Planner.
- **Bilgisayarlı Görü:** Yerdeki işaretleyicilerin gerçek zamanlı tespiti için YOLOv8 (Ultralytics).
- **Donanım Hızlandırma:** Jetson Orin üzerinde CUDA tabanlı GPU hızlandırmalı çıkarım.
- **Haritalama:** Uçuş sırasında çekilen görüntülerden harita/ortofoto üretmek için OpenDroneMap (ODM).
- **Veri Transferi:** İniş sonrası üretilen harita çıktılarının Jetson’dan yer istasyonuna aktarımı için FTP.
- **Diller ve Araçlar:** Algılama ve görev mantığı ağırlıklı olarak Python; başlatma otomasyonu için shell script’ler / systemd servisleri.

---

## 5. Algılama ve Edge AI Hattı

Algılama hattı, uçuş sırasında tamamen Jetson Orin üzerinde çalışır:

1. **Veri Toplama ve Model Eğitimi**
   - Ekip, yerde boyanmış alfasayısal işaretleyicileri (harfler ve rakamlar) içeren bir veri kümesi hazırlamıştır.
   - Görseller, görev ortamına benzer açı ve ışık koşullarında toplanarak gerçek senaryoya yaklaşacak şekilde seçilmiştir.
   - Bu veri kümesi kullanılarak, ilgili işaretleyicileri tespit edecek şekilde özelleştirilmiş bir **YOLOv8** modeli eğitilmiştir.

2. **Yerleşik Çıkarım (Inference)**
   - Jetson Orin, Raspberry Pi HQ Camera’dan gelen kareleri yakalar.
   - YOLOv8 modeli, Jetson GPU’su üzerinde **CUDA** kullanılarak gerçek zamanlı olarak çalıştırılır; böylece İHA’nın hareketine yetişecek hızda tespit yapılabilir.
   - Çıktılarda, tespit edilen nesnelerin bounding box’ları, sınıf etiketleri (harf/rakam) ve güven skorları yer alır.

3. **Karar Mantığı Entegrasyonu**
   - Algılama modülü, tespit sonuçlarını görev mantığı katmanına iletir.
   - Görev mantığı;  
     - Tespitin görev açısından geçerli bir bırakma noktasına karşılık gelip gelmediğini,  
     - Tespitin kararlılığını (örneğin anlık gürültü/yanlış pozitif olmamasını),  
     - Mevcut görev durumunu (rota üzerinde, bırakma bekleniyor, bırakma tamamlandı, eve dönüş vb.)  
     değerlendirir.
   - Koşullar sağlandığında görev mantığı, **kargo bırakma döngüsünü** tetikler ve uçuş kontrolcüsüne ilgili komutları gönderir.

Bu hattın tamamı **onboard** çalışır; bulut ya da yer istasyonunda inference yapılmaz. Sistem, gerçek bir uçuş ortamında çalışan **edge AI tabanlı İHA entegrasyonu**dur.

---

## 6. Otonom Görev ve Kargo Mantığı

Otonom görev mantığı kabaca şu adımlara göre yapılandırılmıştır:

1. **Görev Başlangıcı**
   - Operatörler, Mission Planner üzerinden görev/waypoint setini uçuş kontrolcüsüne yükler.
   - Standart pre-flight kontroller tamamlandıktan sonra İHA kurulur, silahlanır (arm edilir) ve otonom uçuş moduna alınır.
   - Jetson, güç verildiği anda boot olmuş ve servislerini başlatmış durumdadır; kamera akışını ve telemetriyi izlemeye hazırdır.

2. **Rota Üzerinde Uçuş**
   - İHA, otopilotun kontrolünde önceden tanımlı waypoint rotasını takip eder.
   - Jetson, bu sırada kamera görüntüsünü sürekli işleyerek YOLOv8 ile tespit yapar.

3. **Hedef Tespiti ve Konum Tutma**
   - İlgili kargo bırakma bölgesini temsil eden bir harf/rakam tespit edildiğinde:
     - Jetson, gerekli konum bilgisini otopilottan alır veya hesaplar.
     - Jetson, otopilota (örneğin MAVLink üzerinden) hedef üzerinde yavaşlama veya konum koruma (loiter/position hold) komutları göndererek İHA’nın hedef bölge üzerinde stabil kalmasını sağlar.

4. **Kargo Bırakma Döngüsü**
   - İHA hedef üzerinde stabil konuma geldikten sonra:
     - Jetson, uçuş kontrolcüsüne payload motor çıkışını aktive etmesi için komut gönderir.
     - Motor, misinayı salarak kargoyu (su şişesi vb.) yere doğru indirir.
     - Uygun noktada yük bırakılır.
     - Motor ters yönde çalışarak misinayı geri toplar; böylece sistem bir sonraki uçuş segmenti için temiz bir şekilde hazır hale gelir.

5. **Görevin Devamı**
   - Başarılı bir bırakma sonrası Jetson, otopilota görevin devam ettirilmesi komutunu verir.
   - İHA, bir sonraki hedeflere ilerleyerek tespit–konum tutma–bırakma döngüsünü tekrarlayabilir.

6. **Eve Dönüş**
   - Tüm teslimatlar tamamlandığında veya görev başka bir nedenle sona erdiğinde, İHA eve dönüş (Return to Home) moduna geçerek home konumuna otonom şekilde döner ve otopilot kontrolünde iniş yapar.

Operatör perspektifinden bakıldığında, silahlanma ve görevi başlatma adımlarından sonra algılama ve kargo bırakma mantığının tamamı otonom yürür; manuel müdahale yalnızca güvenlik veya olağan dışı senaryolar için gereklidir.

---

## 7. Güvenlik, Failsafe Mekanizmaları ve Yer Kontrol

Sistem yüksek derecede otonom olsa da, güvenlik birincil öncelik olarak ele alınmıştır:

- **Yer Kontrol İstasyonu (GCS)**
  - Mission Planner üzerinden:
    - Görev yükleme,
    - Anlık telemetri takibi (konum, batarya, uçuş modu, durum),
    - Görev ilerleyişinin ve kargo bırakma olaylarının izlenmesi sağlanır.

- **Failsafe Ayarları**
  - Otopilot üzerinde standart failsafe’ler konfigüre edilmiştir, örneğin:
    - Link kaybı durumunda uygulanacak strateji,
    - Düşük batarya seviyesinde alınacak aksiyon,
    - Jeofence ile operasyonel alanın sınırlandırılması.
  - Bu ayarlar, haberleşme kesildiğinde veya kritik bir durum oluştuğunda İHA’nın güvenli bir moda geçmesini sağlar.

- **Manuel Override**
  - Manuel devralma için bir RC verici mevcuttur.
  - Operatör, gerektiğinde otonom moddan çıkabilir, sistemi disarm edebilir veya manuel iniş komutu verebilir.

- **Jetson Servisleri ve Kontrolü**
  - Jetson, açılışta (boot) gerekli servisleri otomatik olarak başlatacak şekilde yapılandırılmıştır (kamera, algılama, görev mantığı, telemetri bağlantıları vb.).
  - Hata ayıklama veya acil durum senaryolarında, operatörler bu servisleri durdurabilir, yeniden başlatabilir veya yeniden konfigüre edebilir.

Bu yapı, normal şartlar altında otonom operasyonu mümkün kılarken, beklenmeyen durumlar için net güvenlik katmanları sağlar.

---

## 8. Veri Kaydı, Haritalama ve FTP İş Akışı

Kargo görevini icra etmenin ötesinde, sistem uçtan uca bir veri işleme hattı da içerir:

1. **Video Kaydı**
   - Kamera aktif olduğu sürece Jetson, uçuşun videosunu kaydeder.
   - Bu kayıtlar:
     - Uçuş sonrası analiz,
     - Tespit olaylarının ve kargo bırakma anlarının gözden geçirilmesi,
     - Yeni veri setleri oluşturma vb. amaçlar için kullanılabilir.

2. **Fotoğraf Çekimi**
   - Sürekli video akışına ek olarak sistem, uçuş sırasında periyodik olarak yüksek çözünürlüklü fotoğraflar alır.
   - Bu fotoğraflar, kargo bırakma alanı ve çevresini kapsar.

3. **OpenDroneMap (ODM) ile Haritalama**
   - Uçuş sonrasında bu fotoğraflar **OpenDroneMap (ODM)** ile işlenerek görev alanının haritası (örneğin ortofoto) üretilir.
   - ODM, hava görüntülerini coğrafi referanslı çıktılara dönüştürerek kargo bırakma alanının detaylı görselleştirilmesini mümkün kılar.

4. **FTP ile Veri Transferi**
   - İşleme tamamlandıktan ve İHA güvenli şekilde iniş yaptıktan sonra, Jetson üretilen harita çıktısını **FTP** aracılığıyla yer kontrol istasyonuna aktarır.
   - Böylece uçuş sırasında toplanan veriden, yer istasyonunda analiz edilebilir harita çıktısına kadar uçtan uca bir iş akışı tamamlanmış olur.

---

## 9. Projenin Gösterdiği Yetkinlikler

Bu proje, aşağıdaki alanlarda somut yetkinlikleri ortaya koymaktadır:

- Özelleştirilmiş bir **YOLOv8** modelinin **NVIDIA Jetson Orin** üzerinde gerçek zamanlı olarak sahada çalıştırılması,
- Edge AI algılama hattının **Cube sınıfı bir uçuş kontrolcüsü** (Pixhawk Cube Orange) ile kapalı döngü karar verme için entegre edilmesi,
- Yüksek seviyeli olayların (işaretleyici tespiti) uçuş davranışına (durma/bekleme, göreve devam) ve aktüatör kontrolüne (kargo vinç motoru) bağlandığı bir **görev mantığı katmanı** tasarımı ve implementasyonu,
- Bir İHA yarışmasının getirdiği **kısıtlar ve güvenlik gereksinimleri** altında sistem tasarımı ve test süreci,
- Uçuş verisinden haritaya uzanan **uçtan uca veri hattı** kurulumu (görüntü toplama → ODM ile haritalama → FTP ile çıktı aktarımı).

İşe alım tarafında bakıldığında, bu depo; **edge AI, İHA entegrasyonu, gerçek zamanlı sistemler ve saha testleri** alanlarında üniversite son sınıf seviyesinde fakat ciddi bir mühendislik tecrübesini kanıtlamayı amaçlamaktadır.

---

## 10. Rolüm ve Katkılarım

Projenin yöneticiliğini, kaptanlığını yaptım.
Bu projeye olan katkılarım özetle şunlardır:

- **Edge AI ve Algılama**
  - Alfasayısal işaretleyiciler için özel veri kümesinin hazırlanması sürecine yöneticilik yaptım.
  - YOLOv8 modelinin eğitimi ve Jetson Orin üzerinde devreye alınması (deployment) süreçlerini yönettim.
  - Kamera akışından tespit çıktısına giden gerçek zamanlı çıkarım hattını (pipeline) geliştirdim.

- **Edge AI performans kazanımı ve Model Optimizasyonu**
  - Cuda etkinleştirme, model optimizasyonu, GPU-CPU kullanım optimizasyonu.

- **Jetson – Otopilot Entegrasyonu**
  - Jetson ile Pixhawk Cube Orange arasındaki haberleşmenin (örneğin MAVLink tabanlı) implementasyonuna katkıda bulundum.
  - Tespit sonuçlarının uçuş ve payload komutlarına dönüştürülmesini sağlayan görev mantığının tasarım ve geliştirme süreçlerinde yer aldım.
  - Jetson’a güç verildiğinde; telemetri bağlantısı, kamera, algılama ve görev mantığı bileşenlerinin otomatik şekilde başlamasını sağlayan başlatma script’leri/servislerini geliştirdim.

- **Kargo Bırakma Mantığı**
  - Payload motor kontrolünün otonomi yığınına entegrasyonunda çalıştım; böylece Jetson’dan gelen komutlara göre uçuş kontrolcüsü kargo vinç motorunu sürerek bırakma ve geri toplama döngüsünü icra edebilir hale geldi.

- **Veri Hattı ve Haritalama**
  - Uçuş sırasında periyodik fotoğraf çekimi iş akışının yapılandırılmasına katkıda bulundum.
  - OpenDroneMap (ODM) ile kargo bırakma alanının haritalandırılması sürecini kurdum ve test ettim.
  - ODM çıktı dosyalarının Jetson’dan yer istasyonuna **FTP** aracılığıyla aktarılması için gerekli iş akışını geliştirdim.

- **Test ve Dokümantasyon**
  - Saha testlerinde görev aldım; sistemin davranışını analiz ederek tespit eşikleri ve görev parametreleri üzerinde iyileştirmeler yaptım.
  - SUAS 2024 video raporunun ve ilgili teknik dokümantasyonun hazırlanmasına katkıda bulundum.

---

### Ekstra Bilgiler

* **Geliştirici**: [Fatih AYIBASAN] (Bilgisayar Mühendisliği Öğrencisi)
* **E-posta**: [fathaybasn@gmail.com]

---