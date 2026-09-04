# Değişiklik Kaydı

Bu dosya yalnızca tamamlanan ve doğrulanan değişiklikleri kaydeder. Planlanan özellikler değişiklik olarak yazılmaz.

## Geliştirme aşaması 0.4e — 2026-09-04

### Düzeltildi

- Gerçek EVE istemcisinin `NSRunningApplication` tarafından `exefile` olarak bildirilen çalışma zamanı yürütülebilirini, doğru `com.ccpgames.eveonline` paket kimliğiyle birlikte exact ve büyük/küçük harfe duyarlı profile ekleme
- İlk süreç keşfi, snapshot öncesi/sonrası çözümleyici ve saf pencere eşleştiricide aynı çalışma adı allowlist'ini kullanma
- `GameProcessIdentity` içinde profil sabiti yerine gerçekten gözlenen yürütülebilir adı koruyarak `exefile` hedefinin yapay snapshot kimliği uyuşmazlığına düşmesini engelleme

### Güvenlik sınırları korundu

- `nil`, boş, kısmi, büyük/küçük harf varyantlı veya yardımcı süreç adları kabul edilmez; görünen uygulama adı kimlik kararına katılmaz
- Launcher'ın ayrı paket kimliği, PID + `launchDate`, öndelik ve ScreenCaptureKit owner PID + paket kimliği çapraz kontrolleri değişmeden kalır
- Mutlak oyun kurulum yolu profile gömülmez; bu metaveri eşleşmesinin yayıncı kod imzası doğrulaması olmadığı açık tutulur

### Doğrulandı

- Gerçek EVE sürecinin güvenli `NSRunningApplication` alanlarında paket kimliği `com.ccpgames.eveonline`, çalışma adı `exefile`, dolu `launchDate` ve arka plan durumu gözlendi. Oyun pikselleri yakalanmadı; süreç argümanları, tam kullanıcı yolu ve ham kimlik değerleri depoya veya kullanıcı belgelerine yazılmadı.
- Exact `EVE`/`exefile` pozitifleri; benzer ad, helper, yanlış paket, görünen ad, sonlandırılmış süreç, Launcher-only, çoklu istemci ve snapshot çalışma adı uyuşmazlığı negatiflerini kapsayan Swift 6 strict harness'i 21 senaryo ve 22/22 kontrolden dört ardışık turda geçti.
- Önceki tam 0.4d sahte entegrasyon harness'i, varsayılan gerçek istemci adı `exefile` olacak biçimde yeniden derlendi; izin/öndelik kapıları, süreç izleme, çözümleyici, iptal ve eski sonuç yollarının tamamı 36 senaryo ve 137/137 kontrolden dört ardışık turda geçti.
- Debug derlemesi, Xcode Analyze ve `SWIFT_VERSION=6` tam sıkı eşzamanlılık/uyarıları hata sayan proje derlemesi başarılı oldu; bağımsız mimari ve UI incelemeleri engelleyici bulgu olmadan **GO** verdi.
- Yeni Xcode derlemesi arka plandaki gerçek EVE istemcisini Launcher'dan ayırdı. Bu derlemede Ekran Kaydı hazır görünmediği için EVE satırı güvenli biçimde **İzin Gerekiyor** gösterdi; uygulama kendiliğinden izin istemedi.

### Canlı doğrulama bekliyor

- Kullanıcının önceki Debug kopyasına verdiği Ekran Kaydı izni yeniden derlenen çalışan kopyada hazır görünmedi. Kullanıcı yeni kopyaya açıkça izin vermeden gerçek `SCShareableContent` envanteri başlatılmayacaktır.
- EVE öndeyken canlı pencere seçimi, normal/borderless/tam ekran sınıflaması ve yeniden arka plana alınca **Öne Getir** geçişi henüz doğrulanmamıştır.
- Görüntü akışı, piksel yakalama, OCR, çeviri ve yan panel bu düzeltmenin parçası değildir.

## Geliştirme aşaması 0.4d — 2026-09-04

### Eklendi

- Tek seferlik `EVEGameWindowResolver` için gerçek uygulama bileşimini ve `LaunchViewModel` yaşam döngüsü bağlantısını kuran üretim başlangıcı
- Süreç izleme oturumu, tam EVE oyun istemcisi, öndelik ve taze Ekran Kaydı preflight sonucu birlikte geçmeden çözümleyiciyi çağırmayan kapılar
- Bağlantı durumunda PID ile birlikte `launchDate` süreç neslini taşıma; etkin istek, önceki seçim, geç sonuç ve dönen seçimde aynı süreç neslini yeniden doğrulama
- Tek EVE hazırlık satırında izin gereksinimi, öne getirme, arama, seçildi, oyun/istemci/pencere belirsizliği, eski kimlik, iptal ve genel hata durumları
- Normal pencere için yalnız **Seçildi**, geometrik tam ekran sonucu için canlı test yapılmadığını belirten **tam ekran adayı** açıklaması

### Güçlendirildi

- Ekran Kaydı gereksinimi öndelik uyarısından önce değerlendirilir; gerekli izin, EVE satırında yanlışlıkla gizlenmez
- EVE PID'si veya `launchDate` süreç nesli değişince etkin eski istek önce iptal edilir ve yeni süreç için tek bir çözümleme başlar
- Eski isteğin geç tamamlanması yeni isteğin görev kaydını temizleyemez; stop sonrasındaki callback ve sonuçlar oturum kimliğiyle reddedilir
- Snapshot sonrasındaki bütün uygulama kimliği değişiklikleri sıradan bekleme yerine eski/güvensiz sonuç olarak sınıflandırılır
- Uygulamanın kendi sahnesi başka bir Space'te görünmüyor diye tek seferlik çözümleme durdurulmaz; tam ekran hedefi için belirleyici, EVE'nin gerçek öndelik durumudur
- Pencere kimliği, PID, `launchDate`, geometri, ekran kimliği ve snapshot nesli arayüz veya erişilebilirlik metnine taşınmaz

### Doğrulandı

- Debug derlemesi, Xcode Analyze ve `SWIFT_VERSION=6` ile sıkı eşzamanlılık/uyarıları hata sayan tam proje derlemesi başarıyla tamamlandı.
- Canlı sistem sağlayıcılarını binary dışında bırakan sahte bağımlılıklı Swift 6 harness'i; Launcher/oyun/izin/öndelik kapıları, single-flight, iptal, stop/eski oturum, PID değişimi, aynı PID + farklı `launchDate`, geç sonuç, izin kaybı, çalışma alanı değişikliği, sonuç eşlemeleri ve otomatik izin istememe dahil 36 senaryo ve 137/137 kontrolden geçti. Aynı binary dört ardışık turda aynı sonucu verdi.
- Harness binary'sinde `SCShareableContent`, `CGRequestScreenCaptureAccess`, `CGRequestListenEventAccess`, `NSWorkspace` veya ScreenCaptureKit bağlantısı bulunmadığı doğrulandı; bütün testler sistem istemi ve gerçek ekran envanteri olmadan çalıştı.
- İki bağımsız salt-okunur mimari/UI incelemesi engelleyici bulgu olmadan **GO** verdi; kaynak farkında pencere başlığı veya capture metadata günlüğü bulunmadı.
- Kullanıcının açtığı gerçek EVE Launcher çalışırken güncel uygulama Xcode'dan başlatıldı. Erişilebilirlik ağacında EVE satırı **Launcher Açık** ve “Launcher açık; EVE oyun istemcisi bekleniyor” gösterdi; iki izin **Eksik** kaldı ve hiçbir izin istemi açılmadı.
- Canlı süreç kontrolünde yalnız `eve-online` Launcher ile Xcode Debug `EVETranslateTR` işlemi görüldü; `EVE` oyun istemcisi çalışmadığından çözümleyici ve gerçek `SCShareableContent` yolu çağrılmadı.

### Canlı doğrulama bekliyor

- Gerçek EVE oyun istemcisi bu turda açılmadı; bu nedenle `SCShareableContent` ile canlı pencere envanteri, seçilmiş normal/borderless pencere ve geometrik tam ekran adayının arayüz sonucu henüz doğrulanmadı.
- macOS Ekran Kaydı ve Giriş İzleme izin verme/reddetme turuna dokunulmadı; izinler kullanıcı düğmeleriyle ayrıca doğrulanacaktır.
- Görüntü akışı, piksel yakalama, OCR, çeviri ve yan panel bu aşamanın parçası değildir.
- Sürekli `SCStream` eklendiğinde macOS Space, uygulama gizleme, arka plan ve enerji politikası yeniden test edilecektir.

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
