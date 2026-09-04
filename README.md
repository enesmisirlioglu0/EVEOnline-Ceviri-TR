# EVE Online Çeviri TR

EVE Online Çeviri TR, macOS üzerinde çalışan yerel bir oyun ekranı çeviri yardımcısıdır. Öncelikli amacı, çalışan EVE Online istemcisini otomatik bulmak ve oyuncu sol Command tuşuna bastığında farenin yakınındaki İngilizce metni Türkçeye çevirip özgün İngilizce yazıyı kapatmadan yanında tek, okunaklı bir panelde göstermektir.

> **Mevcut durum: Geliştirme aşaması 0.4c, eski asenkron sonuçlara dayanıklı pencere çözümleyici altyapısı hazır.** 0.4a'nın EVE süreç izleyicisi ile 0.4b'nin izin kapılı ScreenCaptureKit değer snapshot'ı ve saf pencere eşleştiricisine, snapshot öncesi/sonrası tam süreç kimliği doğrulaması yapan tek seferlik çözümleyici eklenmiştir. İptal edilen, yeni istekle geçersiz kalan veya çalışma alanı değişen sonuçlar kullanılmaz. Bu altyapı henüz uygulama yaşam döngüsüne bağlanmamış; gerçek `SCShareableContent` çağrısı, EVE/Launcher çalıştırması veya macOS izin istemi yapılmamıştır. Ekran yakalama akışı, sol Command tetikleyicisi, OCR, çeviri ve oyun yanında gösterilecek panel henüz uygulanmadı. Aşama numarası uygulamanın pazarlama sürümü değildir.

## Değişmez çalışma biçimi

- Uygulama EVE oyun istemcisini otomatik bulacak; EVE Launcher ve giriş ekranı hedef alınmayacak.
- EVE penceresi normal, büyütülmüş veya tam ekran durumdayken desteklenmeye çalışılacak.
- Sabit alan seçme modu olmayacak.
- EVE görüntü akışı düşük maliyetle hazır tutulacak; ağır OCR ve çeviri yalnızca kullanıcı sol Command tuşuna bastığında çalışacak.
- Fareye en yakın anlamlı cümle veya paragraf çevrilecek.
- İngilizce metin hiçbir zaman kapatılmayacak. Türkçe sonuç, metnin yanına yerleşen ayrı ve tıklama geçirgen tek panelde gösterilecek.
- Oyuncu ve gemi adları çeviri öncesinde korunup sonuçta özgün biçimleriyle geri yerleştirilecek.
- Varsayılan motor Apple Translation olacak. Mimari, ileride başka cihaz içi çeviri motorlarını ve isteğe bağlı yerel yapay zekâ desteğini ekleyebilecek biçimde kurulacak.
- İlk hedef EVE Online'dır; çekirdek yapı daha sonra başka oyun profillerine açılabilecek şekilde tasarlanacak.
- Oyun dosyalarına, çalışan oyun işlemine, belleğine veya kullanıcı girdilerine müdahale edilmeyecek.

Ayrıntılı ürün ve güvenlik planı için [PLAN.md](PLAN.md), kolaydan zora uygulama sırası için [ROADMAP.md](ROADMAP.md), yalnızca tamamlanmış değişiklikler için [CHANGELOG.md](CHANGELOG.md) dosyasına bakın.

## Küçük açılış penceresi

Uygulamanın 420 × 680 punto boyutundaki ana penceresi i-Panel benzeri kompakt bir yardımcı pencere olarak hazırlanmıştır. Üç bölüm içerir:

1. Üstte uygulamanın amacını anlatan kısa tanıtım.
2. Ortada gerçek durum gösteren izin ve hazırlık satırları.
3. Altta üç kısa kurulum ve kullanım adımı.

Ekran Kaydı ve Giriş İzleme satırları gerçek macOS durumuna göre **Eksik**, **Ayar Gerekiyor**, **Yeniden Aç** veya **İzin Hazır** gösterir. **İzin İste** ve **Ayarları Aç** eylemleri birbirinden bağımsızdır; uygulama açılışında yalnız salt-okunur kontrol yapılır. EVE satırı gerçek süreç envanterine göre **Bekleniyor**, **Launcher Açık**, **EVE Bulundu** veya **Seçim Gerekiyor** durumunu gösterir. Türkçe model kendi aşaması gelene kadar **Sırada** kalır.

## Otomatik EVE süreç algılama

0.4a katmanı çalışan uygulamaları `NSWorkspace` ile salt okunur izler. Bir EVE süreç adayı yalnız `com.ccpgames.eveonline` paket kimliği ile `EVE` çalıştırılabilir adı birlikte tam eşleştiğinde kabul edilir. `com.ccpgames.eve-online-launcher`, launcher yardımcıları, Steam kısayolu, bu uygulamanın kendisi ve yalnız adında “EVE” geçen başka süreçler hedef seçilmez.

Sürekli hızlı tarama yapılmaz; uygulama açılma, kapanma ve etkinleşme bildirimlerinde anlık envanter yenilenir. Birden fazla eşleşen EVE süreç adayında yalnız tek bir istemci öndeyse onun PID'si seçilir, aksi durumda rastgele seçim yapılmaz. Bu PID tek başına kalıcı veya kriptografik bir kimlik sayılmaz; sonraki ScreenCaptureKit pencere eşlemesinde güncel paket kimliği ve pencere sahibiyle yeniden çapraz doğrulanacaktır. Bu aşamada EVE veya launcher başlatılmamış, ekran envanteri alınmamış ve macOS izin istemi açılmamıştır.

## Güvenli pencere eşleştirme ve çözümleme altyapısı

0.4b katmanı ham `SCWindow`, `SCRunningApplication` ve `SCDisplay` nesnelerini uygulamanın diğer bölümlerine taşımaz. ScreenCaptureKit sağlayıcısı bunları kendi izolasyon alanında yalnız süreç kimliği, geometri ve sınırlı pencere durumu içeren değişmez `Sendable` değerlere dönüştürür. Canlı yükleyici Ekran Kaydı izni yoksa envanter çağrısına ulaşmadan durur ve kendisi izin istemez.

Snapshot hedef sürece özeldir: yalnız istenen PID + paket kimliğine ait ekranda görünen pencere değerleri saklanır; başka uygulamaların pencere başlıkları veya görünen adları kopyalanmaz. Saf eşleştirici kendisine verilen EVE süreç descriptor'ındaki paket/çalıştırılabilir kimliği ile öndelik durumunu, ScreenCaptureKit tarafında da owner PID + paket kimliğini birlikte denetler. Sahipsiz, yanlış katmanlı, görünmeyen, bozuk geometrili veya çok küçük pencereleri eler; yakın eşitlikte rastgele seçim yapmak yerine belirsizlik üretir. Önceki pencere yalnız aynı, `launchDate` ile ayırt edilebilen süreç neslinde ve hâlâ yeterince büyük/görünürse korunur. `launchDate` yoksa eski seçim tekrar kullanılmaz.

0.4c katmanındaki `EVEGameWindowResolver`, her tek seferlik çözümleme isteği ile onun `NSWorkspace` gözlem oturumunu ayrı kimlikle izler. Yeni istek önceki isteği geçersiz kılar; açık iptal, çağıran görevin iptali veya çalışma alanı değişikliği aktif isteği sonlandırır ve devam eden snapshot görevine iptal sinyali verir. Geç gelen snapshot sonuçları ve durdurulmuş oturumlardan sıraya alınmış callback'ler etkin isteğe karışamaz.

Snapshot'ın iki yanında PID, paket kimliği, çalıştırılabilir adı, `launchDate`, sonlandırılmamış olma ve öndelik yeniden doğrulanır. Eksik `launchDate` veya değişen süreç kimliği güvenli biçimde reddedilir; yalnız hâlâ aynı süreç olduğu doğrulanan sonuç saf pencere eşleştiriciye gönderilir. Sağlayıcı da yükleme öncesi, sonrası ve hata yolunda Ekran Kaydı iznini yeniden kontrol eder; yükleme sırasında izin kaybolmasını genel hata yerine açık izin gereksinimi olarak bildirir ve iptal edilmiş işi başarılı snapshot nesli saymaz. Süreç monitöründeki oturum kimliği denetimi, `stop` sonrasında geciken eski callback'in yeni `start` oturumunu güncellemesini engeller.

Bu çözümleyici henüz `LaunchViewModel` veya uygulama yaşam döngüsüne bağlanmamıştır. Bu alt adımda gerçek `SCShareableContent` envanteri alınmamış, EVE ya da Launcher çalıştırılmamış, EVE penceresi canlı seçilmemiş ve TCC istemi açılmamıştır. Tam ekran etiketi hâlâ yalnız geometrik bir tanıdır; canlı macOS tam ekran Space desteği kanıtı değildir.

## Gerekli izinler ve nedenleri

| İzin / hazırlık | Neden gerekli? | Sınır |
| --- | --- | --- |
| Ekran Kaydı | EVE penceresindeki, kullanıcının zaten gördüğü yazıyı ScreenCaptureKit ile okuyabilmek için | Yalnız hedef oyun görüntüsü bellekte işlenir; görüntü ve ses diske kaydedilmez |
| Giriş İzleme | EVE öndeyken yalnızca sol Command tetikleyicisini dinleyebilmek için | Uygulama tuş üretmez, tıklama göndermez, makro çalıştırmaz ve yazılan metni kaydetmez |
| İngilizce–Türkçe dil modeli | Apple Translation'ın cihaz içi modeli hazır değilse sistemin modeli indirebilmesi için | Bu bir macOS gizlilik izni değildir; model hazırlama adımıdır |

Mikrofon, sistem sesi, kamera, Tam Disk Erişimi, kişiler, konum, Apple Events/Otomasyon ve oyun arayüzünü kontrol etmeye yarayan Erişilebilirlik izni planlanmamaktadır. Uygulama geliştirilirken yeni bir iznin gerçekten zorunlu olduğu kanıtlanırsa önce nedeni belgelenecek ve kapsam ayrıca değerlendirilecektir.

Planlanan ScreenCaptureKit kullanımı için Apple belgelerinde belirtilen `NSScreenCaptureUsageDescription` uygulama paketine eklenmiştir. Apple tarafından belgelenmiş bir `NSInputMonitoringUsageDescription` anahtarı bulunmadığından böyle bir anahtar uydurulmamış; Giriş İzleme gerekçesi doğrudan uygulama arayüzünde açıklanmıştır. Sistem Ayarları bağlantıları yalnız yardımcıdır; izin kapalı olduğunda kullanıcıya **Sistem Ayarları → Gizlilik ve Güvenlik → Ekran ve Sistem Sesi Kaydı** veya **Giriş İzleme** yolu da gösterilir.

## Teknik yön

- Native macOS, Swift ve SwiftUI
- Minimum macOS 15.0
- İzin kapılı ScreenCaptureKit değer snapshot'ı, saf pencere eşleştiricisi ve henüz yaşam döngüsüne bağlanmamış, eski sonuçlara dayanıklı tek seferlik çözümleyici
- Vision ile cihaz üzerinde OCR
- Değiştirilebilir `TranslationEngine` yapısı; ilk motor Apple Translation
- Sol Command için yalnızca dinleyen global giriş gözlemcisi
- İngilizceyi örtmeyen, kenara göre yer değiştiren, tıklama geçirgen tek çeviri paneli
- Yerel önbellek, EVE terimleri sözlüğü ve özel ad koruma katmanı
- Kullanıcı hesabı, abonelik veya zorunlu ücretli servis yok

## Gizlilik ve güvenlik sınırı

- Yakalanan kareler ve OCR metinleri kalıcı olarak kaydedilmeyecek.
- Bulut çevirisi varsayılan olmayacak; yerel yapay zekâ seçeneği eklenirse açıkça isteğe bağlı ve cihaz içi olacak.
- EVE giriş, hesap, parola ve doğrulama ekranlarında çalışma otomatik duracak.
- Uygulama yalnızca okuyacak ve sonuç gösterecek; oyun istemcisine kod enjekte etmeyecek, ağ paketlerini incelemeyecek ve oyun girdisi üretmeyecek.
- Sol Command başka bir tuşla birlikte kullanıldığında normal kısayol kabul edilecek ve çeviri tetiklenmeyecek şekilde tasarlanacak.

## DMG ve tanıtım planı

Kaynak projede yerel kurulum paketlerinin kopyalanacağı görünür bir `DMG` klasörü bulunmaktadır. Çalışan özellikler gerçek EVE oturumunda doğrulandıktan sonra i-Panel'deki düzene benzer bir DMG üretim ve doğrulama akışı eklenecektir.

Genel dağıtım için planlanan sıra GitHub Release üzerinde doğrulanmış DMG, SHA-256 özeti ve kurulum notudur. EVE'nin Steam mağaza sayfası Workshop desteği göstermediği, fakat Community Guides bölümü bulunduğu için Steam tarafındaki tanıtım hedefi şimdilik **Topluluk Rehberi**dir; Workshop hedefi değildir. Yayınlama yalnızca kullanıcı onayı ve güncel platform/CCP kuralları yeniden kontrol edildikten sonra yapılacaktır.

## EVE üçüncü taraf yazılım uyarısı

CCP, üçüncü taraf uygulamaları için kapsamlı bir “izin verilenler” listesi yayımlamadığını ve kullanımın oyuncunun kendi riski altında olduğunu belirtmektedir. Projenin yalnızca ekrandaki metni okuması, oyuna girdi göndermemesi ve ayrı bir panel göstermesi riski azaltır; yine de bu bir CCP onayı anlamına gelmez. Gerçek oyun testi ve genel yayın öncesinde güncel EULA ile üçüncü taraf uygulama politikası yeniden incelenecek; belirsizlik sürerse CCP desteğinden yazılı açıklama istenmesi değerlendirilecektir.

## Resmî teknik ve politika kaynakları

- [Apple ScreenCaptureKit](https://developer.apple.com/documentation/screencapturekit)
- [Apple Quartz Event Services](https://developer.apple.com/documentation/coregraphics/quartz-event-services)
- [Apple Translation](https://developer.apple.com/documentation/translation)
- [CCP — Third party applications](https://support.eveonline.com/hc/en-us/articles/5888034246428-Third-party-applications)
- [CCP — EVE Online EULA](https://support.eveonline.com/hc/en-us/articles/8413329735580-EVE-Online-End-User-License-Agreement)
- [Steam — EVE Online Community Guides](https://steamcommunity.com/app/8500/guides/)

## Proje düzeni

- **Kaynak kod:** Ayrı kaynak deposu; Swift/Xcode projesi ve yerel `DMG` çıktı klasörü
- **Dokümantasyon:** Bu depo; ürün tanımı, yol haritası ve doğrulanmış ilerleme kayıtları

## Bağımsızlık bildirimi

Bu, topluluk tarafından geliştirilen bağımsız ve resmî olmayan bir yardımcı uygulama projesidir. CCP Games ile bağlantılı, onun tarafından desteklenen veya onaylanan bir proje değildir. EVE Online adı ve ilgili işaretler kendi hak sahiplerine aittir. Projede oyuna ait görsel, yazı tipi veya başka varlıklar kullanılmaz; yalnızca benzer okunabilirlik hissi veren özgün bir tema tasarlanır.
