# Yol Haritası

İş sırası, önce bağımsız ve kolay doğrulanabilen parçaları, ardından oyun yakalama ve çeviri zincirini, en sonda gerçek oyun güvenliği ile dağıtımı tamamlayacak şekilde düzenlenmiştir.

Durum işaretleri:

- ✅ Tamamlandı ve doğrulandı
- 🟡 Sıradaki çalışma
- ⏳ Daha sonra

## 0.1 — Proje temeli ve ürün kararları

- ✅ Kaynak ve dokümantasyon depolarını oluştur
- ✅ Derlenebilir yerel macOS SwiftUI başlangıç projesini oluştur
- ✅ Kaynak ve çalışma dosyalarını `MYPROJECT` altında düzenle
- ✅ Ürünün gizlilik ve oyun istemcisine müdahale etmeme sınırlarını yaz
- ✅ Otomatik EVE bulma, sol Command, tam ekran çalışma ve İngilizceyi örtmeyen yan panel kararlarını plana işle
- ✅ Yerel DMG kopyaları için kaynak projede görünür `DMG` klasörünü hazırla

## 0.2 — Kompakt açılış ve durum ekranı

- ✅ Büyük başlangıç ekranını 420 × 680 punto, tek sütunlu ve dikey kaydırılabilir kompakt pencereye dönüştür
- ✅ Üste ürün amacını; sol Command, cihaz içi çalışma ve tek panel ilkelerini ekle
- ✅ Ekran Kaydı, Giriş İzleme, Türkçe model ve EVE bağlantısı için hazırlık satırlarını tasarla
- ✅ Henüz bağlanmayan bütün işlevleri sahte düğme veya sahte başarı yerine dürüstçe **Sırada** olarak göster
- ✅ Alta üç adımlı kısa kullanım yönergesi ile gizlilik sınırı notunu yerleştir
- ✅ Kararlı satır kimliklerini ve erişilebilirlik başlık/etiketlerini ekle; satırların erişilebilirlik ağacında ayrı öğeler olduğunu, yatay taşmasız görünümü ve kaydırmayı çalışan uygulamada doğrula

## 0.3 — İzin koordinatörü

- ✅ Ekran Kaydı durumunu salt-okunur kontrol et; izin isteğini yalnız açıklamalı düğmeye basıldığında başlat
- ✅ Giriş İzleme durumunu salt-okunur kontrol et; izin isteğini yalnız kendi düğmesine basıldığında başlat
- ✅ İzin yok, Sistem Ayarları gerekiyor, aynı süreçte yeniden kontrol/açılış gerekiyor ve hazır durumlarını anlaşılır göster
- ✅ Sistem Ayarları bağlantısına ek olarak kullanıcıya doğru manuel Gizlilik ve Güvenlik yolunu yaz
- ✅ Gereksiz Erişilebilirlik, mikrofon, sistem sesi, kamera ve Tam Disk Erişimi istemediğini kod ve paket ayarlarında doğrula
- ✅ Planlanan ScreenCaptureKit kullanımı için Apple belgelerinde belirtilen `NSScreenCaptureUsageDescription` değerini pakete ekle; belgelenmemiş `NSInputMonitoringUsageDescription` anahtarı ekleme
- ✅ İzin açıklamalarını kaynak README, uygulama metni ve proje ayarlarıyla eşleştir
- 🟡 Kullanıcı açıkça onayladığında gerçek macOS izin verme, reddetme, Sistem Ayarları'ndan açma ve uygulamaya dönüş turunu bu Mac'te doğrula

## 0.4 — EVE'yi otomatik bulma

- ✅ `GameApplicationProfile` ile EVE'nin tam oyun ve launcher kimlik alanlarını birbirinden ayır
- ✅ `NSWorkspace` süreç envanterini ekran izni istemeden ve uygulama başlatmadan salt okunur al
- ✅ Yalnız tam `com.ccpgames.eveonline` + `EVE` eşleşmesini kabul et; launcher, helper, Steam kısayolu ve bulanık ad eşleşmelerini dışla
- ✅ Uygulama açılma, kapanma ve etkinleşme bildirimleriyle polling yapmadan durumu yenile
- ✅ Tek istemcide PID'yi seç; çoklu istemcide yalnız tek öndeki adayı kabul et ve belirsizlikte rastgele seçim yapma
- ✅ **Bekleniyor**, **Launcher Açık**, **EVE Bulundu** ve **Seçim Gerekiyor** süreç durumlarını kompakt arayüze bağla
- ✅ Ham ScreenCaptureKit nesnelerini sızdırmayan `Sendable` uygulama, pencere ve ekran değer snapshot'larını hazırla
- ✅ Ekran Kaydı izni yokken yükleyiciyi çağırmayan, yalnız hedef PID + paket kimliğine ait görünür pencere metadatasını saklayan sağlayıcıyı hazırla
- ✅ Kendisine verilen süreç kimliği ile owner PID + paket kimliğini çapraz denetleyen; bozuk/görünmez/küçük pencereleri eleyen saf eşleştiriciyi hazırla
- ✅ Çoklu ekran geometrisi, tam ekran tanısal sınıflaması, yakın eşitlikte belirsizlik ve aynı süreç neslindeki önceki seçimi kapsayan sahte testleri tamamla
- ⏳ Asenkron envanterin öncesi ve sonrasında güncel `NSWorkspace` sürecini denetleyen, eski sonuçları iptal eden koordinatörü ekle
- ⏳ Kullanıcı açıkça onayladığında `SCShareableContent` ile gerçek hedef pencere envanterini al ve sonucu arayüze bağla
- ⏳ Launcher, giriş ve hesap pencerelerini canlı turda doğrula ve hassas ekran güvenlik kapısını tamamla
- ⏳ Belirsizlik sürerse küçük kullanıcı seçimini ekle
- ⏳ EVE arka plandayken ağır işlemleri durdur
- ⏳ Kullanıcı açıkça onayladığında gerçek EVE açılma, kapanma ve öne gelme turunu bu Mac'te doğrula

## 0.5 — Normal ve tam ekran görüntü takibi

- ⏳ EVE penceresi için ScreenCaptureKit akışını başlat
- ⏳ Pencere taşıma, yeniden boyutlandırma, Retina ölçeği ve ekran koordinatlarını doğru eşleştir
- ⏳ macOS tam ekran Space'i ve borderless/pencereli EVE kullanımını dene
- ⏳ Gerekirse EVE uygulamasına filtrelenmiş ekran yakalama yedeğini uygula
- ⏳ Çoklu ekran, Space değiştirme, küçültme ve yeniden öne getirme durumlarını test et
- ⏳ Eski kareleri bırakıp yalnız en güncel kareyi geçici bellekte tut; diske görüntü yazma

## 0.6 — Sol Command tetikleyicisi

- ⏳ Yalnız dinleme yapan `CGEventTap` ile sol ve sağ Command'ı ayır
- ⏳ Sol Command basışını bir istek için yalnız bir kez tetikle
- ⏳ Command ile başka tuş birlikte kullanıldığında çeviriyi iptal et ve normal kısayola karışma
- ⏳ Tetikleyiciyi yalnız hedef oyun öndeyken etkin say
- ⏳ Escape ile panel gizleme ve uygulama içinden genel Duraklat/Sürdür davranışını ekle
- ⏳ Uyumsuz klavyeler için isteğe bağlı yedek kısayol ayarı hazırla; varsayılanı sol Command tut

## 0.7 — Fare yakınında OCR

- ⏳ Fare konumunu güncel oyun karesiyle eşleştir
- ⏳ Vision ile önce küçük bir yakın alanı, gerekirse genişleyen alanı tanı
- ⏳ Birbiriyle ilişkili satırları okuma sırasına göre tek paragrafta birleştir
- ⏳ Düşük güven, yalnız sayı, anlamsız kısa metin, aynı dil ve tekrar filtrelerini ekle
- ⏳ Küçük EVE yazıları, ölçeklendirilmiş arayüz ve hareketli arka planda doğruluk ölç
- ⏳ Sabit alan seçme veya tüm ekranı sürekli OCR'lama kodu ekleme

## 0.8 — Yerel İngilizce–Türkçe çeviri

- ⏳ Motorları değiştirebilen `TranslationEngine` sınırını oluştur
- ⏳ Apple Translation dil desteği ve model durumunu `LanguageAvailability` ile denetle
- ⏳ Gerekirse kullanıcı onaylı model hazırlama/indirme akışını göster
- ⏳ Tek paragraf çevirisi, hata durumları ve yerel önbelleği uygula
- ⏳ Oyuncu ve gemi adlarını yer tutucuyla koruyup sonuçta özgün hâline getir
- ⏳ Temel EVE terimleri sözlüğü ile “bir daha çevirme” kullanıcı listesini ekle
- ⏳ Modül, görev, eşya ve açıklama adlarını genel olarak çevrilebilir bırak

## 0.9 — EVE uyumlu yan panel

- ⏳ Tıklama geçirgen ve odak çalmayan tek macOS paneli oluştur
- ⏳ Paneli kaynak İngilizce metnin sağına; yer yoksa soluna, altına veya güvenli ekran kenarına yerleştir
- ⏳ İngilizce metni hiçbir durumda kapatmayan yerleşim denetimi ekle
- ⏳ EVE hissine uyumlu fakat tamamen özgün koyu tema, sistem yazı tipi ve yüksek karşıtlık kullan
- ⏳ Uzun Türkçe paragrafta satır kırma, boyut sınırı ve kaydırma davranışını doğrula
- ⏳ Pencere/Space/ekran değişiminde hizalamayı koru
- ⏳ Görsel efektlerin gecikme veya GPU etkisi varsa efektleri kademeli azalt

## 0.10 — Performans ve çeviri kalitesi

- ⏳ Görüntü akış hızını ve çözünürlüğü Mac'in yüküne göre dengele
- ⏳ Tetiklemeden panel görünmesine kadar gecikmeyi ölç
- ⏳ CPU, GPU, bellek, enerji ve EVE kare hızı etkisini birlikte ölç
- ⏳ Görüntü/paragraf hash'i ve çeviri önbelleğiyle tekrar işi azalt
- ⏳ İsim koruma hataları ile EVE terimlerini gerçek örneklerle düzelt
- ⏳ Yaygın kısa metinlerde yaklaşık bir saniyelik hedefi ve “belirgin FPS düşüşü yok” koşulunu kanıtla veya yeniden ayarla

## 0.11 — Gerçek EVE doğrulaması ve politika kapısı

- ⏳ Görev açıklaması, bilgi paneli, sohbet, menüler, tooltip ve hareketli HUD üzerinde ayrı testler yap
- ⏳ Normal pencere, borderless ve macOS tam ekran davranışlarını kayıt altına al
- ⏳ Giriş ve kimlik bilgisi ekranı korumasını doğrula
- ⏳ Oyuna girdi gönderilmediğini, istemci/bellek/ağ/önbelleğe erişilmediğini denetle
- ⏳ CCP'nin güncel EULA ve üçüncü taraf uygulama politikasını yeniden kontrol et
- ⏳ Genel yayın öncesi gerekirse CCP desteğine mimariyi açıkça anlatan yazılı uygunluk sorusu hazırla
- ⏳ Bu kapı geçmeden “CCP onaylı”, “tamamen risksiz” veya “her tam ekranda kesin çalışır” iddiasında bulunma

## 0.12 — Yerel DMG ve kişisel kurulum

- ⏳ Universal Release derlemesi ve sürüm/build adlandırmasını hazırla
- ⏳ DMG üretim betiğini ekle ve çıktıyı kaynak projenin `DMG` klasörüne kopyala
- ⏳ Pakette uygulama ile Applications bağlantısını doğrula
- ⏳ `hdiutil verify`, uygulama mimarisi, sürüm ve `PrivacyInfo.xcprivacy` kontrollerini çalıştır
- ⏳ SHA-256 özeti ve imzasız/noterlenmemiş paket için dürüst Gatekeeper notu oluştur
- ⏳ DMG'yi bu Mac'e temiz kurulum adımlarıyla dene; mevcut kaynak/ayarları silme

## 0.13 — İkinci yerel motor ve isteğe bağlı yapay zekâ

- ⏳ Türkçe kalitesi olan cihaz içi model adaylarını araştır
- ⏳ Model lisansı, kaynağı, hash'i, boyutu, RAM/CPU kullanımı ve Apple motoruna göre kalitesini karşılaştır
- ⏳ Uygun aday bulunursa ikinci `TranslationEngine` olarak ekle
- ⏳ Küçük yerel yapay zekâ ile yalnız isteğe bağlı terim/akıcılık düzeltmesini prototiple
- ⏳ Yapay zekâ kapalıyken temel uygulamanın eksiksiz çalıştığını doğrula
- ⏳ Ekran görüntüsü veya OCR metnini buluta gönderen varsayılan özellik ekleme

## 0.14 — Başka oyunlara genişleme

- ⏳ EVE'ye özgü algılama, tema, terim ve ad kurallarını `GameProfile` içinde ayır
- ⏳ Genel oyun penceresi profili ve güvenli tek seferlik seçim yedeği ekle
- ⏳ İkinci bir oyunla yakalama–OCR–çeviri–panel zincirinin yeniden kullanılabildiğini kanıtla
- ⏳ Her oyun için izin, kullanım şartı, tam ekran ve özel ad filtrelerini ayrı doğrula
- ⏳ Aynı anda yalnızca bir hedef oyun izleme sınırını koru

## 0.15 — Genel yayın ve tanıtım

- ⏳ Kaynak ve dokümantasyon depolarını gerçek özellik durumuyla eşleştir
- ⏳ Kullanıcı açıkça onaylarsa doğrulanmış DMG, SHA-256 ve sürüm notunu GitHub Release'e yükle
- ⏳ Gizlilik, izinler, kaldırma, CCP bağımsızlık/risk uyarısı ve sorun giderme metnini yayımla
- ⏳ Kullanıcı açıkça onaylarsa Steam Community Guide üzerinde tanıtım ve GitHub indirme bağlantısı hazırla
- ⏳ EVE'nin Steam özelliklerinde Workshop desteği görünmediği sürece Workshop yayını planlama
- ⏳ App Store veya otomatik güncelleme kararını ancak kararlı genel sürümden sonra ayrıca değerlendir

## Doğrulama ve 10 saniyelik ilerleme kuralı

Bir aşama, ilgili özellik gerçek derleme ve uygun testlerle doğrulanmadan ✅ işaretlenmeyecektir. Her bağımsız yenilik tamamlandığında kullanıcıya kısa rapor verilecek ve durdurma veya düzeltme için 10 saniye beklenecektir. Kullanıcı bu sürede durdurmazsa ve doğrulamada sorun yoksa değişiklik ayrı, açık kapsamlı bir commit olarak GitHub'a gönderilecek ve sıradaki özelliğe geçilecektir.

Kaynak ile dokümantasyon değişiklikleri kendi depolarında ayrı commitlerle izlenecektir. GitHub Release yayımlamak, DMG'yi genel dağıtıma açmak, CCP ile iletişim kurmak, Steam Community Guide veya App Store yayını yapmak ve veri silen işlemler otomatik ilerleme kapsamında değildir; bunlar ayrıca açık kullanıcı onayı gerektirir.

On saniyelik sessizlik; macOS gizlilik/TCC istemi açma, gerçek ekran yakalama, EVE'yi çalıştırma, Sistem Ayarlarını değiştirme, paket veya araç kurma, ücretli servis kullanma, remote/dal değiştirme ya da yeni ürün kapsamına geçme izni sayılmaz. Böyle bir adım gerektiğinde kullanıcıdan ayrıca açık onay alınır.
