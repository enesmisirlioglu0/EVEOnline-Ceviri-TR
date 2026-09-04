# Değişiklik Kaydı

Bu dosya yalnızca tamamlanan ve doğrulanan değişiklikleri kaydeder. Planlanan özellikler değişiklik olarak yazılmaz.

## Geliştirme aşaması 0.3 — 2026-09-04

### Eklendi

- Ekran Kaydı ile Giriş İzleme için birbirinden bağımsız gerçek izin durumları ve `LaunchViewModel` tabanlı açılış durumu
- İzin durumunu istem açmadan okuyan `CGPreflightScreenCaptureAccess` ve `CGPreflightListenEventAccess` koordinatörü
- Yalnız kullanıcı ilgili **İzin İste** düğmesine bastığında çalışabilen `CGRequestScreenCaptureAccess` ve `CGRequestListenEventAccess` yolları
- İzin kapalıysa **Ayarları Aç** eylemi ile birlikte doğru manuel Sistem Ayarları yolu
- İzin değişikliği aynı süreçte görünmezse yanlış **İzin Hazır** sonucu vermeyen **Yeniden Aç** durumu
- Ekran yakalama gerekçesini içeren `NSScreenCaptureUsageDescription` Info.plist girdisi
- Sahte koordinatörlerle izin mantığını sistem istemi açmadan sınayabilmek için bağımlılık enjeksiyonu

### Doğrulandı

- Temiz macOS arm64 derlemesi başarıyla tamamlandı ve derlenmiş uygulama paketindeki ekran yakalama kullanım açıklaması doğrudan okundu.
- Derlenmiş pakette belgelenmemiş `NSInputMonitoringUsageDescription` anahtarının bulunmadığı doğrulandı.
- Geçici sahte bağımlılık denetimi; salt-okunur kontrol, reddedilen istek, yeniden açma gereksinimi, sonradan izinli durum, gereksiz tekrar istememe ve iki Sistem Ayarları bağlantısı dahil 10 kontrolü geçti.
- Xcode'da çalışan uygulama hiçbir sistem istemi açmadan iki izni **Eksik** gösterdi; iki **İzin İste** düğmesi erişilebilirlik ağacında ayrı adlandırılmış kontroller olarak göründü.
- Türkçe çeviri modeli ve EVE'yi otomatik bulma satırları çalışma zamanı özelliği henüz olmadığı için **Sırada** kalmaya devam etti.

### Canlı doğrulama bekliyor

- Gerçek macOS izin istemine basılmadı; izin verme, reddetme, Sistem Ayarları'ndan açma ve uygulamaya dönüş turu kullanıcı açıkça onayladığında ayrıca denenecek.

## Geliştirme aşaması 0.2 — 2026-09-04

### Eklendi

- 420 × 680 punto boyutunda, tek sütunlu ve dikey kaydırılabilir kompakt macOS açılış penceresi
- EVE hissine uyumlu fakat özgün koyu renk teması, ürün amacı ve sol Command/cihaz içi/tek panel rozetleri
- Ekran Kaydı, Giriş İzleme, Türkçe model ve EVE bağlantısı için hazırlık satırları
- EVE'yi açma, fareyi metne getirme ve sol Command ile çeviri isteme akışını anlatan üç kısa adım
- Erişilebilirlik başlıkları, ayrı durum/adım açıklamaları ve yeniden çizimde odağı koruyan kararlı öğe kimlikleri

### Doğrulandı

- Uygulama macOS arm64 hedefinde başarıyla derlendi ve Xcode'dan temiz süreçle çalıştırıldı.
- Başlık, kartlar ve durum etiketleri 420 punto genişlikte yatay taşma olmadan görüntülendi.
- Alt kullanım kartı ile gizlilik notuna dikey kaydırmayla eksiksiz ulaşılabildi.
- Dört hazırlık satırı ve üç kullanım adımı erişilebilirlik ağacında ayrı, adlandırılmış öğeler olarak doğrulandı; gerçek VoiceOver sesli gezinme testi henüz yapılmadı.
- Henüz uygulanmayan izin, EVE bulma ve çeviri işlevleri yalnız **Sırada** olarak gösterildi; sahte sistem izni veya çalışan düğme sunulmadı.

### Henüz uygulanmadı

- Bu sürüm çalışma zamanı çevirisi yapmaz. Gerçek izin koordinatörü 0.3 aşamasında başlayacaktır.

## 0.1.1 — 2026-09-04

### Değiştirildi

- Ürün amacı kullanıcının son kararlarına göre yeniden yazıldı: EVE'yi otomatik bulma, sol Command ile tetikleme, tam ekranı destekleme ve İngilizceyi örtmeyen ayrı yan panel artık ana çalışma biçimidir.
- Sabit alan modu, Option varsayılanı ve özgün metnin üstüne çeviri yerleştiren görünüm plandan çıkarıldı.
- EVE görüntüsünü düşük maliyetle hazır tutup OCR ve çeviriyi yalnız tetikleme anında çalıştıran performans yaklaşımı belgelendi.
- Çeviri katmanı Apple Translation ile başlayıp başka cihaz içi motorlara ve isteğe bağlı yerel yapay zekâ düzeltmesine açılacak biçimde planlandı.
- Yalnız oyuncu ve gemi adlarının zorunlu olarak özgün kalması; diğer oyun metinlerinin çevrilebilir olması kararlaştırıldı.
- EVE öncelikli çekirdeğin ileride başka oyun profillerine açılma sınırı eklendi.
- Yol haritası kolaydan zora; kompakt arayüz, izinler, otomatik algılama, yakalama, tetikleyici, OCR, çeviri, panel, performans, genişleme, gerçek oyun testi ve dağıtım sırasına geçirildi.

### Eklendi

- Ekran Kaydı ve Giriş İzleme izinlerinin nedenleri ile istenmeyecek izinlerin açık listesi
- Kompakt tanıtım, gerçek izin durumları ve üç adımlı kullanım ekranı tanımı
- Tam ekran, çoklu ekran, Space, özel ad ve Command kısayolu testleri
- Güncel CCP üçüncü taraf uygulama/EULA inceleme kapısı ve bağımsızlık uyarısı
- GitHub Release ile Steam Community Guide dağıtım yaklaşımı; EVE için Workshop'un hedef olmadığı açıklaması
- Kaynak projede gelecekte doğrulanmış yerel kurulum paketlerinin kopyalanacağı `DMG` klasörü

### Henüz uygulanmadı

- Bu sürüm yalnız ürün belgelerini ve çıktı klasörü düzenini günceller; çalışma zamanı özellikleri kaynak koda eklenmedi.

## 0.1.0 — 2026-09-04

### Eklendi

- Derlenebilir macOS SwiftUI başlangıç iskeleti
- Kaynak ve dokümantasyon için ayrı klasör/depo düzeni
- Ürün planı, yol haritası ve kapsam sınırları
- Cihaz üzerinde çalışma, seçici çeviri ve oyun istemcisine müdahale etmeme ilkeleri
- Boş veri toplama bildirimi içeren başlangıç gizlilik manifesti

### O aşamada henüz uygulanmamıştı

- EVE penceresi seçimi ve ekran yakalama
- OCR ve İngilizce–Türkçe çeviri
- Fareyle çeviri, sabit alan ve tek seferlik çeviri
- Oyun üstü tıklama geçirgen çeviri paneli
