# Plan — EVE Online için Yerel Mac Çeviri Uygulaması

## 1. Ürünün amacı

macOS üzerinde çalışan, EVE Online oyun istemcisini otomatik bulan ve oyuncu sol Command tuşuna bastığında farenin yakınındaki İngilizce metni Türkçeye çeviren yerel bir yardımcı uygulama geliştirilecek.

Türkçe sonuç, özgün İngilizce metni kapatmayacak. Metnin yanında veya ekran kenarında, EVE arayüzünün koyu görünümüne uyumlu, ayrı ve tıklama geçirgen tek panelde gösterilecek.

İlk ve öncelikli oyun EVE Online'dır. Ekran yakalama, OCR, çeviri ve panel bileşenleri oyundan bağımsız kurulacak; EVE sürümü kararlı hâle geldikten sonra başka oyun profilleri eklenebilecek.

## 2. Değişmez sınırlar

Uygulama:

- EVE Online'ın dosyalarını değiştirmeyecek.
- Oyunun belleğine veya çalışan işlemine bağlanmayacak.
- Oyuna kod enjekte etmeyecek, hook kurmayacak ve ağ paketlerini incelemeyecek.
- Oyuna komut, tıklama ya da klavye girdisi göndermeyecek.
- Makro, bot, otomatik karar veya oyuncuya haksız avantaj sağlayan analiz üretmeyecek.
- Yalnızca kullanıcının ekranda zaten görebildiği pikselleri okuyacak.
- Ekran görüntülerini ve OCR metinlerini diske kaydetmeyecek.
- Mümkün olduğunca tamamen cihaz üzerinde çalışacak.
- İngilizce yazının üstünü Türkçe kutularla kapatmayacak.
- Aynı anda yalnızca tek çeviri paneli gösterecek.

## 3. Ana çalışma akışı

1. Uygulama açılır ve gerekli izinlerin durumunu gösterir.
2. Çalışan süreçler ve ScreenCaptureKit içeriği içinden EVE oyun istemcisi otomatik bulunur.
3. EVE öndeyken oyun görüntü akışı düşük maliyetle hazır tutulur; yalnızca en güncel kare geçici bellekte saklanır.
4. Kullanıcı fareyi öğrenmek istediği İngilizce cümle veya paragrafın üstüne getirir.
5. Kullanıcı klavyenin sol altındaki sol Command tuşuna basar.
6. Uygulama güncel karede fareye en yakın anlamlı metin bloğunu Vision ile okur.
7. Oyuncu ve gemi adları geçici yer tutucularla çeviriden korunur.
8. Metin seçili yerel çeviri motoruyla İngilizceden Türkçeye çevrilir; korunan adlar özgün hâliyle geri eklenir.
9. Türkçe sonuç, İngilizceyi kapatmadan hemen yanında tek panelde görünür.
10. Sonraki çeviri isteği önceki panelin içeriğini değiştirir; kullanıcı Escape veya uygulamadaki duraklatma denetimiyle paneli gizleyebilir.

Sabit alan seçimi ve ekranın tamamındaki her yazıyı sürekli çevirmek bu çalışma biçiminin parçası değildir.

## 4. EVE'yi otomatik bulma ve tam ekran desteği

### 4.1 EVE profili

İlk EVE profili, oyun istemcisini uygulama kimliği ve pencere sahibi bilgisiyle bulacak. Bu Mac'teki mevcut kurulumda oyun istemcisi `com.ccpgames.eveonline`, Launcher ise `com.ccpgames.eve-online-launcher` kimliğini kullanmaktadır. Bu değerler sürüm değişikliklerine karşı tek başına yeterli kabul edilmeyecek; işlem adı ve yakalanabilir pencere bilgisiyle birlikte değerlendirilecektir.

- Launcher, giriş ve hesap pencereleri otomatik olarak dışlanacak.
- Birden fazla EVE istemcisi bulunursa öndeki oyun penceresi seçilecek; belirsizlik varsa küçük bir seçim uyarısı gösterilecek.
- Oyun kapanırsa uygulama bekleme durumuna dönecek ve yeniden açıldığında tekrar bağlanacak.
- EVE arka plandayken OCR ve çeviri duracak.

0.4a uygulamasında bunun ilk, TCC gerektirmeyen katmanı tamamlanmıştır. `GameApplicationProfile.eveOnline`, EVE süreç adayını yalnız tam `com.ccpgames.eveonline` paket kimliği ve `EVE` çalıştırılabilir adıyla kabul eder; launcher için `com.ccpgames.eve-online-launcher` ayrı bekleme durumu üretir. Bu metaveri eşleşmesi yayıncı kod imzasını kanıtlamaz. `NSWorkspace` açılma, kapanma ve etkinleşme bildirimleri sürekli polling yerine anlık yenileme sağlar. Birden fazla eşleşen istemcide yalnız tek öndeki aday seçilir, aksi durumda sonuç belirsiz kalır. Seçilen PID sonraki ScreenCaptureKit pencere sahibi eşleşmesinde güncel süreç kimliğiyle yeniden doğrulanacaktır.

### 4.2 Normal ve tam ekran yakalama

Önce ScreenCaptureKit'in EVE'ye ait pencere yüzeyi kullanılacak. Tam ekran modunda ayrı pencere yüzeyi alınamıyorsa EVE uygulamasına filtrelenmiş ekran yakalama yolu denenecek. Normal pencere, macOS tam ekran Space'i, farklı çözünürlükler ve çoklu ekran kullanımı gerçek oyunda ayrı ayrı doğrulanacak.

0.4b uygulamasında canlı bağlantıdan önceki güvenli altyapı tamamlanmıştır. Ham ScreenCaptureKit nesneleri yalnız sağlayıcının izolasyon alanında kalır ve hedef PID + paket kimliğine ait, ekranda görünen pencere değerleri `Sendable` snapshot'a dönüştürülür. Pencere başlıkları ve başka uygulamaların görünen adları saklanmaz. İzin kapısı erişim yokken yükleyiciyi çağırmaz. Saf eşleştirici owner PID + paket kimliğini, kendisine verilen EVE süreç descriptor'ındaki kimlik/öndelik durumunu, görünür geometriyi, layer bilgisini, çoklu ekran kesişimini ve aynı süreç neslindeki önceki seçimi değerlendirir; yakın eşitlikte rastgele seçim yapmaz. Bu kod henüz uygulama yaşam döngüsünden çağrılmadığı için gerçek ScreenCaptureKit envanteri alınmış veya EVE penceresi canlı bulunmuş sayılmaz.

0.4c uygulamasında tek seferlik `EVEGameWindowResolver` koordinasyonu tamamlanmıştır. Çözümleyici snapshot'ın öncesinde ve sonrasında güncel `NSWorkspace` envanterini yeniden okur; PID, paket kimliği, çalıştırılabilir adı, `launchDate`, sonlandırılmamış olma ve öndelik alanlarının aynı kaldığını doğrular. Eksik veya değişmiş kimlik kapalı biçimde reddedilir. Her istek ve gözlem oturumu ayrı kimlikle izlenir; yeni istek, açık iptal, çağıran görevin iptali veya çalışma alanı değişikliği önceki sonucu geçersiz kılar. Geç gelen eski görev ve callback sonuçları etkin isteğe uygulanmaz.

ScreenCaptureKit sağlayıcısı asenkron yükleme öncesinde, sonrasında ve hata yolunda Ekran Kaydı erişimini yeniden kontrol eder; yükleme sırasında izin kaybolursa sonuç açık izin gereksinimi olarak sınıflandırılır. İptal edilmiş iş snapshot neslini ilerletmez. Süreç monitörü de durdurulmuş bir gözlem oturumundan kalan callback'in daha sonraki oturuma karışmasını oturum kimliğiyle engeller.

Bu alt adım yalnız koordinasyon altyapısıdır. Çözümleyici henüz `LaunchViewModel` veya uygulama yaşam döngüsüne bağlanmadı; gerçek `SCShareableContent` çağrısı, EVE/Launcher çalıştırması, canlı pencere seçimi veya TCC izin turu yapılmadı.

0.4d uygulamasında bu çözümleyici `LaunchViewModel` ile kompakt durum ekranına bağlanmıştır. Tek seferlik pencere çözümlemesi ancak süreç izleme oturumu açık, tam EVE oyun istemcisi önde ve taze Ekran Kaydı preflight sonucu izinli olduğunda başlar. Launcher-only, oyun kapalı, istemci belirsizliği, EVE'nin önde olmaması ve izin eksikliği durumlarında çözümleyici ya da gerçek ScreenCaptureKit sağlayıcısı çağrılmaz. Otomatik yol izin istemez; `CGRequest…` çağrıları yalnız kullanıcının ilgili **İzin İste** düğmesindedir.

ViewModel, etkin isteği kendi oturum/istek kimliğiyle ve EVE süreç neslini PID + `launchDate` çiftiyle izler. Aynı PID başka bir açılışta yeniden kullanılsa dahi eski pencere seçimi yeni oyuna taşınmaz. EVE kapanır, değişir veya öndelik kaybederse etkin iş iptal edilir; geç tamamlanan eski sonuçlar yeni görevin durumunu temizleyemez. Snapshot sonrasında değişen süreç kimliği sıradan “pencere bulunamadı” değil, güvenli biçimde eski sonuç olarak gösterilir.

Yardımcı uygulamanın kendi sahnesinin başka bir macOS Space'inde görünmemesi tek başına çözümlemeyi durdurmaz. Tam ekran EVE ayrı bir Space'te öndeyken yardımcı pencere görünmez olabileceğinden kapı, uygulamanın sahne görünürlüğü yerine EVE'nin gerçek öndelik durumuna bağlanmıştır. Bu karar yalnız tek seferlik metadata envanteri içindir; sürekli `SCStream` aşamasında arka plan, Space ve enerji politikası yeniden ele alınacaktır.

İlk canlı 0.4d turunda kullanıcı yalnız EVE Launcher'ı açmıştır. Xcode'dan çalıştırılan güncel uygulama **Launcher Açık** durumunu göstermiş, gerçek oyun istemcisi bulunmadığı için pencere çözümleyicisine/`SCShareableContent` yoluna girmemiş ve macOS izin istemi açmamıştır. Gerçek normal, borderless ve tam ekran EVE pencere seçimi hâlâ canlı test beklemektedir.

“Tam ekranda çalışır” hedefi ürüne dâhildir; ancak her oyun ve macOS sürümündeki tam ekran davranışı aynı olmadığından bu özellik gerçek EVE testi geçmeden tamamlanmış sayılmayacaktır.

## 5. Sol Command tetikleyicisi

- Varsayılan ve ana tetikleyici yalnızca **sol Command** tuşudur; sağ Command aynı işi yapmaz.
- Sol Command tek başına basıldığında çeviri başlatılır.
- Command başka bir tuşla birlikte kullanılırsa normal klavye kısayolu kabul edilir ve çeviri tetiklenmez.
- Uygulama klavyeyi yalnızca dinler; hiçbir tuş olayı üretmez veya engellemez.
- Sol Command bazı klavye düzenlerinde ya da oyun kısayollarında sorun çıkarırsa ayarlardan değiştirilebilen bir yedek tuş sunulabilir; ürünün varsayılanı yine sol Command kalır.

Global tetikleme için ilk tercih, yalnız dinleme yapan Core Graphics olay gözlemcisidir. Bu yaklaşımın ihtiyaç duyduğu Giriş İzleme izni açıkça anlatılacak ve yalnız kullanıcı özelliği etkinleştirdiğinde istenecektir.

## 6. Metni bulma ve OCR

- Bütün ekran sürekli OCR'a gönderilmeyecek.
- Uygulama görüntü akışını hazır tutacak, OCR yalnız sol Command tetiklemesinde çalışacak.
- Fare konumu oyun penceresinin koordinatlarına çevrilecek.
- Önce fare çevresindeki sınırlı görüntü kesiti analiz edilecek; paragraf tamamlanmıyorsa alan kontrollü biçimde genişletilecek.
- Vision tarafından bulunan metin kutuları okuma sırasına göre birleştirilecek.
- Yalnız sayılar, çok kısa ve anlamsız etiketler, çok düşük güvenli sonuçlar, zaten Türkçe içerik ve öncekiyle aynı metin atlanacak.
- Sabit alan çizme, görev kutusu seçme veya bütün ekranı otomatik tarama arayüzü olmayacak.

## 7. İngilizceyi kapatmayan çeviri paneli

Çeviri paneli ayrı bir `NSPanel`/AppKit yardımcı penceresi olarak tasarlanacak:

- İngilizce kaynak metnin üzerine gelmeyecek.
- Önce metnin sağına, yer yoksa soluna, altına veya güvenli ekran kenarına yerleşecek.
- Tek panel kullanılacak; her satır için ayrı kutu oluşturulmayacak.
- Oyundaki tıklamaları ve fare hareketlerini engellemeyecek.
- EVE penceresi hareket ettiğinde, yeniden boyutlandığında veya Space değiştirdiğinde konumu yeniden hesaplanacak.
- Kullanıcı paneli anında gizleyebilecek ve tüm yakalamayı duraklatabilecek.

Tema, EVE'nin okunabilir koyu arayüz hissine uyacak; fakat CCP'ye ait görsel, ikon, yazı tipi veya başka varlık kopyalanmayacak. Özgün tasarımda koyu yarı saydam yüzey, ince soğuk renkli kenarlık, yüksek karşıtlıklı Türkçe metin, rahat satır aralığı ve gerektiğinde büyüyen panel kullanılacak. Performans sorunu görülürse önce gölge, bulanıklık ve animasyonlar azaltılacak; İngilizceyi örtmeme kuralından vazgeçilmeyecek.

## 8. Çeviri motorları

Çeviri katmanı tek bir servise kilitlenmeyecek. Ortak bir `TranslationEngine` arayüzü kullanılacak.

### 8.1 İlk motor: Apple Translation

- İngilizce–Türkçe dil çifti uygulama açılışında `LanguageAvailability` ile denetlenecek.
- Dil modeli kurulu değilse sistemin indirme/hazırlama akışı kullanıcıya gösterilecek.
- Çeviri, model hazır olduğunda cihaz üzerinde yürütülecek.
- Sonuçlar yalnız yerel bellekte ve sınırlı yerel önbellekte tutulacak.

Minimum hedef macOS 15.0'dır; Translation framework bu sürüm ailesiyle kullanılabilir hâle gelmiştir.

### 8.2 Ek yerel motor

Apple motorundan bağımsız ikinci bir cihaz içi motor daha sonra değerlendirilecek. Model kalitesi, Türkçe desteği, uygulama boyutu, bellek/işlemci kullanımı ve lisansı ölçülmeden bağımlılık seçilmeyecek.

### 8.3 İsteğe bağlı yerel yapay zekâ desteği

Kolay ve performanslı olduğu kanıtlanırsa küçük bir yerel yapay zekâ modeli, ham çeviri yapmak yerine bağlama göre terim düzeltme ve cümleyi daha doğal Türkçeleştirme aşamasında kullanılabilir. Bu özellik:

- Varsayılan olmayacak.
- Buluta ekran görüntüsü veya metin göndermeyecek.
- Kullanıcı tarafından açıkça etkinleştirilecek.
- Temel Apple çevirisi olmadan uygulamayı kullanılamaz hâle getirmeyecek.

Bulut tabanlı yapay zekâ ilk sürüm kapsamı dışındadır.

## 9. Oyuncu ve gemi adlarını koruma

Oyuncu adı ve gemi adı çevirilmemesi gereken iki temel özel ad sınıfıdır.

- EVE profilindeki ekran bağlamı, biçim ipuçları ve yerel sözlük kullanılacak.
- Tanınan adlar çeviri motoruna gönderilmeden önce yer tutucularla maskelenecek.
- Çeviri tamamlanınca adlar özgün yazımıyla geri yerleştirilecek.
- Kullanıcı yanlış çevrilen bir adı “bir daha çevirme” listesine ekleyebilecek.
- İsim tanıma OCR üzerinden yüzde yüz hatasız olamayacağı için yanlış pozitif ve yanlış negatif örnekleri gerçek oyunda ölçülecek.

Modül, görev, eşya, yetenek, bölge ve açıklama adları kullanıcı bunların anlamını öğrenmek isteyebileceği için genel olarak çevrilebilir kalacak. Gerekli EVE kısaltmaları ile teknik terimler yerel sözlükte özgün bırakılabilir veya tercih edilen Türkçe karşılığıyla gösterilebilir.

## 10. Başka oyunlara genişleme

Çekirdek servisler oyundan bağımsız olacak:

- `GameDetector`: çalışan hedef oyunu bulur.
- `CaptureService`: hedef uygulamanın güncel görüntüsünü sağlar.
- `TextRecognitionService`: fare yakınındaki metni okur.
- `TranslationEngine`: seçili cihaz içi motorla çevirir.
- `NameProtectionService`: oyun profiline göre özel adları korur.
- `TranslationPanelController`: sonucu kaynak metni örtmeden gösterir.

EVE'ye özgü uygulama kimlikleri, tema, terimler ve isim kuralları bir `GameProfile` içinde tutulacak. EVE sürümü kararlı olduktan sonra başka oyun profilleri eklenebilecek. İlk sürüm aynı anda yalnızca bir hedef oyunu izleyecek.

## 11. Kompakt ana uygulama tasarımı

Ana pencere i-Panel benzeri küçük bir yardımcı uygulama olacak; ilk tasarım hedefi yaklaşık 400–430 punto genişliğinde tek sütunlu bir düzendir.

### Üst bölüm — Tanıtım

- Uygulama adı ve özgün küçük simgesi
- “EVE'deki İngilizce metni sol Command ile yanında Türkçe gösterir” biçiminde tek cümlelik amaç
- Durum: EVE bekleniyor, EVE bulundu, hazır, duraklatıldı veya izin eksik

### Orta bölüm — İzin ve hazırlık satırları

1. **Ekran Kaydı:** Eksik / İzin Ver / Ayarları Aç / İzin Hazır
2. **Giriş İzleme:** Eksik / İzin Ver / Ayarları Aç / İzin Hazır
3. **Türkçe Çeviri Modeli:** Hazırlanıyor / İndir / Hazır / Desteklenmiyor
4. **EVE'yi Otomatik Bulma:** Oyun bekleniyor / EVE bulundu / Yakalama hazır

Bu satırlar gerçek sistem durumunu okuyacak. macOS iznini uygulama içinden açıyormuş gibi görünen sahte anahtar kullanılmayacak.

### Alt bölüm — Kısa kullanım

1. EVE Online'ı aç.
2. Fareyi İngilizce metnin üzerine getir.
3. Sol Command'a bas; Türkçe çeviri yanında görünür.

En altta **Çeviriyi Duraklat/Sürdür**, **Paneli Gizle** ve gerektiğinde **Ayarlar** denetimleri bulunacak. Ana pencere açık kalmak zorunda olmayacak; uygulamanın hazır/duraklatılmış durumu menü çubuğundan görülebilecek.

## 12. İzinler ve açıklamaları

### Gerekli

| İzin | Kullanım | Uygulamanın yapmayacağı şey |
| --- | --- | --- |
| Ekran Kaydı | ScreenCaptureKit ile hedef oyunun görünür görüntüsünü almak | Ses kaydetmek, ekran görüntüsü dosyası oluşturmak, başka uygulamaları sürekli arşivlemek |
| Giriş İzleme | EVE öndeyken sol Command tuşunu yalnızca dinlemek | Tuş kaydı tutmak, yazılan içeriği saklamak, tuş/tıklama göndermek veya engellemek |

Uygulama açılışında yalnız `CGPreflightScreenCaptureAccess` ve `CGPreflightListenEventAccess` ile mevcut durum okunur. İzin isteyen `CGRequestScreenCaptureAccess` ve `CGRequestListenEventAccess` çağrıları yalnız kullanıcının kendi satırındaki **İzin İste** düğmesine basmasıyla çalışabilir; iki istem arka arkaya otomatik açılmaz.

Planlanan ScreenCaptureKit kullanımı için Apple belgelerinde belirtilen `NSScreenCaptureUsageDescription` uygulama paketinde bulunur. Apple tarafından belgelenmiş bir `NSInputMonitoringUsageDescription` anahtarı olmadığı için böyle bir anahtar eklenmez; Giriş İzleme gerekçesi uygulama içinde görünür metinle açıklanır. Sistem Ayarları deep linkleri yalnız kolaylık amaçlıdır ve gerçek izin kaynağı sayılmaz; manuel Gizlilik ve Güvenlik yolu da her zaman gösterilir.

### İzin olmayan hazırlık

Apple'ın İngilizce–Türkçe modeli hazır değilse Translation framework kullanıcıdan model indirmesini isteyebilir. Bu, macOS Gizlilik ve Güvenlik izni değil; cihaz içi model hazırlığıdır.

### İstenmeyecek

- Mikrofon ve sistem sesi
- Kamera
- Tam Disk Erişimi
- Konum, kişiler, fotoğraflar ve takvim
- Apple Events/Otomasyon
- Oyun arayüzünü kontrol etmeye yarayan Erişilebilirlik

Geliştirme sırasında yeni bir iznin zorunlu olduğu kanıtlanırsa izin eklenmeden önce kullanım amacı, veri sınırı ve kaldırma yolu README'de açıklanacak.

## 13. Performans yaklaşımı

- EVE öndeyken ScreenCaptureKit akışı düşük ve dengeli kare hızında hazır tutulacak.
- En güncel kare bellekte tutulacak; eski kareler hemen bırakılacak.
- OCR ve çeviri yalnız tetikleme anında çalışacak.
- Önce farenin yakınındaki küçük görüntü kesiti analiz edilecek.
- Aynı görüntü ve metin için OCR/çeviri tekrarlanmayacak.
- Aynı metnin önceki çevirisi yerel önbellekten gösterilecek.
- Panel bulanıklığı ve animasyonları performans bütçesine göre ayarlanacak.
- EVE arka plandaysa ağır iş tamamen duracak.

İlk gerçek oyun ölçümlerinde tetiklemeden panel görünmesine kadar geçen süre, CPU, bellek ve enerji etkisi kaydedilecek. Hedef, oyunda hissedilir kare hızı düşüşü yaratmadan yaygın kısa metinlerde yaklaşık bir saniye içinde kullanılabilir sonuç göstermektir; ölçüm yapılmadan bu hedef “başarıldı” sayılmayacaktır.

## 14. Güvenlik, gizlilik ve CCP sınırı

- Giriş, hesap, parola ve doğrulama kodu ekranları otomatik dışlanacak.
- OCR çıktısı günlük dosyalarına yazılmayacak.
- Uygulama kendi ekran/OCR telemetrisini veya kullanıcı analitiğini toplamayacak. Apple Translation sistem çerçevesinin işletimsel kullanımı Apple'ın geçerli gizlilik koşullarına tabi olacak.
- Uygulama ücretsiz, hesapsız ve aboneliksiz çalışacak.
- Üçüncü taraf model eklenirse lisans, dosya kaynağı ve hash doğrulaması belgelenecek.
- Gerçek kullanım ve her genel sürüm öncesinde CCP'nin güncel EULA ve üçüncü taraf uygulama politikası yeniden kontrol edilecek.

CCP, üçüncü taraf araçların kullanımını kullanıcının kendi riski olarak tanımlar ve kapsamlı bir izin listesi sunmaz. Ayrı ekran paneli, yerel OCR/çeviri, kullanıcı tetiklemesi ve oyun girdisi üretmeme yaklaşımı riski azaltır; fakat CCP onayı anlamına gelmez. İlk genel yayından önce mimarinin CCP desteğine somut biçimde sorulması değerlendirilecektir.

## 15. İlk kullanılabilir EVE sürümü

İlk kullanılabilir sürümde hedeflenenler:

1. EVE oyun istemcisini otomatik bulma ve Launcher'ı dışlama
2. Normal pencere ve tam ekran EVE yüzeyini takip etme
3. Sol Command'ı tek başına global tetikleyici olarak algılama
4. Fare çevresindeki İngilizce paragrafı Vision ile okuma
5. Oyuncu ve gemi adlarını koruma
6. Apple Translation ile cihaz üzerinde İngilizce–Türkçe çeviri
7. İngilizceyi örtmeyen, EVE uyumlu, tıklama geçirgen tek yan panel
8. Aynı metni yeniden işlemeyi önleyen yerel önbellek
9. Temel EVE terimleri sözlüğü ve kullanıcı istisna listesi
10. Kompakt tanıtım, izin durumu ve kısa kullanım penceresi
11. Tek denetimle paneli gizleme ve bütün işlemi duraklatma

## 16. İlk sürümde olmayacaklar

- Sabit alan seçimi
- Ekrandaki bütün yazıları otomatik ve sürekli çevirme
- İngilizce metnin üstünü kapatma
- Ses tanıma veya sistem sesi çevirisi
- Toplantı ya da video altyazısı
- Bulut tabanlı yapay zekâ servisi
- Kullanıcı hesabı, abonelik veya ücretli API zorunluluğu
- Otomatik tıklama, makro, oyun komutu veya karar desteği
- Birden fazla oyunu aynı anda izleme
- İlk aşamada App Store yayını

Ek cihaz içi motor, yerel yapay zekâ düzeltmesi ve başka oyun profilleri temel EVE sürümü doğrulandıktan sonraki aşamalardır.

## 17. Tamamlanma ölçütleri

İlk kullanılabilir EVE sürümü şu şartları sağlamalıdır:

- EVE istemcisini kullanıcıya her açılışta seçtirmeden bulur.
- Launcher ve giriş/kimlik bilgisi ekranını yakalamaz.
- Normal pencere ve doğrulanmış tam ekran EVE kullanımında çalışır.
- Sol Command tek başına basıldığında doğru metni çevirir; Command kısayollarına karışmaz.
- İngilizce kaynak metni görünür bırakır ve yalnızca tek Türkçe panel gösterir.
- Panel oyundaki tıklamaları engellemez ve ekran kenarlarında doğru yere taşınır.
- Oyuncu ve gemi adlarını doğrulanmış test örneklerinde özgün bırakır.
- Görev ve açıklama paragraflarını doğru sırada okuyabilir.
- Temel çeviri ve OCR internet bağlantısı olmadan çalışır; yalnız ilk model hazırlığı internet gerektirebilir.
- Yakalanan görüntüleri ve tanınan metni diske kaydetmez.
- Oyunun dosyalarına, belleğine, ağına ve kontrollerine dokunmaz.
- Ölçülen performans EVE'nin kare hızını belirgin biçimde düşürmez.

## 18. DMG ve yayın yaklaşımı

- Yerel DMG kopyaları kaynak projenin `DMG` klasöründe tutulacak.
- Paket adı sürüm ve build numarası içerecek.
- DMG açılma, içerik, mimari, sürüm, gizlilik manifesti ve `hdiutil verify` ile doğrulanacak.
- Genel sürüme SHA-256 özeti ve Gatekeeper kurulum notu eklenecek.
- GitHub Release ana indirme kaynağı olacak; ikili dosya doğrudan Git geçmişine eklenmeyecek.
- Steam tarafında, EVE için mevcut olan Community Guides alanında tanıtım ve güvenli kullanım rehberi hazırlanacak.
- EVE mağaza sayfası Workshop desteği göstermediğinden Steam Workshop bu uygulamanın dağıtım hedefi değildir.
- Genel yayın, GitHub değişikliği ve Steam rehberi kullanıcıdan ayrıca açık onay alınmadan yapılmayacak.

## 19. Ana ilke

> EVE'yi otomatik bul; görüntüyü hafifçe hazır tut; yalnız sol Command istendiğinde çevir; İngilizceyi kapatma; Türkçeyi yanında tek, hızlı ve EVE'ye uyumlu panelde göster.
