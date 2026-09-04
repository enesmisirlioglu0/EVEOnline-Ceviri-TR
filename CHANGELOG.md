# Değişiklik Kaydı

Bu dosya yalnızca tamamlanan ve doğrulanan değişiklikleri kaydeder. Planlanan özellikler değişiklik olarak yazılmaz.

## Geliştirme aşaması 0.4c — 2026-09-04

### Eklendi

- Her tek seferlik pencere çözümlemesini ayrı istek ve `NSWorkspace` gözlem oturumu kimliğiyle yöneten `@MainActor` `EVEGameWindowResolver`
- Snapshot alınmadan önce ve sonra PID, paket kimliği, çalıştırılabilir adı, `launchDate`, sonlandırılmamış olma ve öndelik alanlarını yeniden doğrulayan kapalı-güvenli süreç kimliği denetimi
- EVE'nin çalışmaması, yalnız Launcher'ın açık olması, çoklu aday, arka plan, eksik/değişen kimlik, izin gereksinimi, snapshot hatası, iptal, yeni istekle geçersiz kalma ve çalışma alanı değişikliğini birbirinden ayıran değer tipi çözümleme sonuçları
- Yalnız güncel süreç doğrulaması başarılı olduğunda saf `GameWindowMatcher` sonucunu değiştirmeden üst katmana aktaran eşleştirme sınırı
- Yeni isteğin önceki isteği geçersiz kılması; açık iptal, çağıran görev iptali ve çalışma alanı değişikliğinde aktif isteğin sonlandırılıp devam eden snapshot görevine iptal sinyali verilmesi
- Geç tamamlanan eski snapshot sonucu ile eski gözlem oturumundan sıraya alınmış callback'in etkin isteği değiştirmesini engelleyen istek/oturum denetimleri

### Güçlendirildi

- ScreenCaptureKit snapshot sağlayıcısı, asenkron yüklemenin öncesinde ve sonrasında erişimi yeniden denetler; yükleme hatası sırasında Ekran Kaydı izni kaybolmuşsa genel hata yerine açık `screenRecordingPermissionRequired` sonucu üretir
- İptal edilen snapshot işi başarılı snapshot nesli olarak sayılmaz ve generation değerini ilerletmez
- `WorkspaceEVEApplicationMonitor`, durdurulmuş bir gözlem oturumundan geciken callback'in sonraki `start` oturumunu yenilemesini oturum kimliğiyle reddeder

### Doğrulandı

- Debug derlemesi, Xcode Analyze ve `SWIFT_VERSION=6` ile sıkı eşzamanlılık/uyarıları hata sayan derleme başarıyla tamamlandı.
- Gerçek çözümleyici ve süreç monitörü kaynakları; sahte süreç envanteri, gözlemci, snapshot sağlayıcısı ve pencere eşleştiricisiyle snapshot öncesi/sonrası süreç durumları, tam kimlik değişimi, yalnız görünen ad değişimi, izin/hata ayrımı, matcher sonuç aktarımı, açık ve çağıran iptali, yeni isteğin eskisini geçersiz kılması, çalışma alanı değişikliği, geç başarı/hata, eski callback ve gözlemci temizliği dahil 54 senaryo ve 314/314 kontrolden geçti. Aynı binary dört ardışık turda aynı sonucu verdi.
- Gerçek ScreenCaptureKit sağlayıcı dosyasında üç iptal ve üç izin kapısının yükleme/generation işlemine göre güvenli sırası salt-okunur kaynak denetimiyle doğrulandı; canlı ScreenCaptureKit çağrısı yapılmadı.
- Güncel kod incelemesinde çözümleyicinin uygulama yaşam döngüsüne bağlanmadığı, gerçek ScreenCaptureKit sağlayıcısının çağrılmadığı ve istemsiz TCC isteği eklenmediği doğrulandı.

### Canlı doğrulama bekliyor

- `EVEGameWindowResolver`, ScreenCaptureKit sağlayıcısı ve pencere eşleştiricisi henüz `LaunchViewModel` veya uygulama yaşam döngüsüne bağlı değildir.
- Bu alt adımda gerçek `SCShareableContent` çağrısı yapılmamış, EVE veya Launcher çalıştırılmamış, canlı pencere seçilmemiş ve macOS izin istemi açılmamıştır.
- Gerçek hedef pencere envanterini alma ve sonucu arayüze bağlama, kullanıcı açıkça onayladığında ayrı bir canlı doğrulama adımı olacaktır.
- Normal pencere, borderless ve macOS tam ekran Space desteği gerçek EVE turu geçmeden tamamlanmış sayılmayacaktır.

## Geliştirme aşaması 0.4b — 2026-09-04

### Eklendi

- `NSRunningApplication.launchDate` değerini süreç nesli ayırımı için taşıyan salt-değer süreç descriptor'ı; tarih yoksa önceki seçimi yeniden kullanmayan güvenli davranış
- Ham `SCWindow`, `SCRunningApplication` ve `SCDisplay` nesnelerini uygulamanın diğer katmanlarına taşımayan `Sendable` değer snapshot'ları
- Ekran Kaydı izni yoksa gerçek yükleyiciye hiç ulaşmayan ScreenCaptureKit sağlayıcı kapısı
- Yalnız istenen PID + paket kimliğine ait ekranda görünen pencereleri saklayan hedefe özel payload; başka uygulamaların pencere başlığı ve görünen adı kopyalanmaz
- Snapshot'ın hangi süreç niyetiyle üretildiğini taşıyan tam `GameProcessIdentity` ve yanlış süreç snapshot'ını kapalı biçimde reddeden kontrol
- Owner PID + paket kimliği, kendisine verilen descriptor'daki executable/öndelik durumu, pencere katmanı, görünürlük, sonlu geometri, minimum ölçü ve çoklu ekran kesişimini denetleyen saf `GameWindowMatcher`
- Büyük ana pencereyi küçük yardımcı adaydan önceleyen, yakın eşitlikte tek aktif aday yoksa belirsizlik üreten ve dizi sırasına göre rastgele seçim yapmayan kurallar
- Aynı süreç neslindeki yeterince büyük önceki pencereyi koruma ve yalnız tanılama amaçlı geometrik normal/tam ekran sınıflaması

### Doğrulandı

- Debug derlemesi, Xcode Analyze ve `SWIFT_VERSION=6` ile sıkı eşzamanlılık/uyarıları hata sayan derleme başarıyla tamamlandı.
- Hedefe özel sahte sağlayıcı ve saf matcher harness'i; kimlik, izin kapısı, yanlış snapshot, bozuk/görünmez geometri, çoklu ekran, ana pencere/modal sıralaması, belirsizlik, süreç nesli, önceki seçim, tam ekran çıkarımı, dizi sırası bağımsızlığı ve generation davranışı dahil 64 kontrolü geçti.
- İzin reddinde sahte payload yükleyicisinin çağrı sayısının `0` kaldığı ve hedef süreç bilgisinin yükleyiciye aktarılmadığı doğrulandı.
- Güncel kod incelemesinde ScreenCaptureKit nesnesi sızıntısı, uygulama yaşam döngüsünden canlı sağlayıcı çağrısı veya istemsiz TCC isteği bulunmadı.

### Canlı doğrulama bekliyor

- Sağlayıcı ve eşleştirici henüz uygulama yaşam döngüsüne bağlı değildir; gerçek `SCShareableContent` çağrısı yapılmamış ve EVE penceresi canlı seçilmemiştir.
- Asenkron envanterin öncesi ve sonrasında güncel süreç/öndelik doğrulaması, eski sonuçları iptal eden koordinatör ve canlı tam ekran EVE turu sonraki adımlardır.

## Geliştirme aşaması 0.4a — 2026-09-04

### Eklendi

- Başka oyun profillerine genişleyebilen tam kimlik alanlı `GameApplicationProfile` değeri ve EVE profili
- Çalışan uygulamaları Ekran Kaydı izni istemeden değer tipi anlık görüntülere dönüştüren `NSWorkspace` sağlayıcısı
- EVE süreç adayını yalnız `com.ccpgames.eveonline` paket kimliği ile `EVE` çalıştırılabilir adı birlikte eşleştiğinde kabul eden saf eşleştirici; bu metaveri eşleşmesi yayıncı imzası doğrulaması değildir
- Launcher'ı, yardımcı süreçleri, Steam kısayolunu, çeviri uygulamasının kendisini ve bulanık ad benzerliklerini dışlayan kurallar
- Uygulama açılma, kapanma ve etkinleşme bildirimleriyle çalışan; sürekli tarama yapmayan süreç monitörü
- Tek eşleşen istemci adayını PID'siyle seçme; çoklu adayda yalnız tek öndeki adayı kabul etme ve rastgele seçimden kaçınma
- Arayüzde **Bekleniyor**, **Launcher Açık**, **EVE Bulundu** ve **Seçim Gerekiyor** durumları

### Doğrulandı

- Debug derlemesi ve Xcode Analyze başarıyla tamamlandı.
- Güncel sahte süreç sağlayıcılı matcher/monitor harness'i; kesin oyun/launcher kimliği, yanlış eşleşmeler, sonlandırılmış süreçler, çoklu istemci seçimi, farklı oyun profili, idempotent başlatma/durdurma, bildirim güncellemesi ve gözlemci iptali dahil 25 kontrolü geçti.
- `LaunchViewModel` entegrasyon harness'i; başlatma, ilk durum aktarımı, launcher ve seçilen PID güncellemesi, yenileme, durdurma ve callback bırakma dahil 11 kontrolü geçti.
- Xcode'dan çalışan uygulama, EVE kapalıyken gerçek `NSWorkspace` envanterinden **Bekleniyor** sonucunu üretti.
- Süreç algılama klasöründe polling, uygulama başlatma, ScreenCaptureKit veya TCC istek çağrısı bulunmadığı doğrulandı.

### Canlı doğrulama bekliyor

- EVE veya launcher bu aşamada başlatılmadı. Gerçek açılma, kapanma, öne gelme ve çoklu istemci turu kullanıcı açıkça onayladığında ayrıca denenecek.
- ScreenCaptureKit pencere envanteri ve seçilen PID'nin pencere sahibiyle çapraz doğrulanması 0.4'ün sonraki alt adımıdır.

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
