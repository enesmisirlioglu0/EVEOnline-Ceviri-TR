# Plan — EVE Online için Yerel Mac Çeviri Uygulaması

## 1. Amaç

macOS üzerinde çalışan, kullanıcı tarafından seçilen EVE Online penceresindeki İngilizce metinleri Türkçeye çeviren yerel bir yardımcı uygulama geliştirilecek.

Uygulama:

- EVE Online'ın dosyalarını değiştirmeyecek.
- Oyunun belleğine veya çalışan işlemine bağlanmayacak.
- Oyuna komut, tıklama ya da klavye girdisi göndermeyecek.
- Yalnızca kullanıcının ekranda zaten görebildiği metni okuyacak.
- Çeviriyi oyunun arayüzüne uyumlu ve tıklamaları engellemeyen şeffaf bir katmanda gösterecek.
- Mümkün olduğunca tamamen cihaz üzerinde çalışacak.

## 2. Değişmez tasarım kararı

Uygulama, EVE penceresindeki bütün yazıları otomatik olarak çevirmeyecek. Pencere seçimi yalnızca görüntünün hangi uygulamadan alınacağını belirleyecek.

Tam ekran otomatik çeviri, her yazının üzerinde ayrı bir kutu oluşturup oyunu kullanılamaz hâle getirebilir. Bu nedenle çeviri yalnızca kullanıcı istediğinde veya önceden seçilmiş sınırlı bir alanda çalışacak. Aynı anda yalnızca tek bir çeviri paneli gösterilecek.

## 3. Çeviri biçimleri

### 3.1 Fareyle isteğe bağlı çeviri

- Kullanıcı fareyi bir metnin üzerine getirir.
- Ayarlanabilir bir yardımcı tuşa basılı tutar.
- İlk aday tuş Option'dır; EVE kısayollarıyla çakışırsa değiştirilebilir.
- Uygulama yalnızca farenin yakınındaki cümleyi veya paragrafı okur.
- Çeviri tek panelde gösterilir.
- Tuş bırakıldığında veya fare uzaklaştırıldığında çeviri kaybolur.

Bu, varsayılan çalışma biçimi olacaktır.

### 3.2 Sabit alan çevirisi

- Kullanıcı görev açıklaması, bilgi paneli veya sohbet kutusu gibi tek bir alanı çerçeveler.
- Uygulama yalnızca bu alanı takip eder.
- Metin değişmedikçe OCR ve çeviri tekrarlanmaz.
- EVE penceresi taşınır veya yeniden boyutlandırılırsa seçilen alan pencereyle birlikte hareket eder.

### 3.3 Tek seferlik çeviri

- Kullanıcı bir klavye kısayoluna basar.
- Seçilen alan o anda bir kez okunur ve çevrilir.
- Sürekli ekran takibi yapılmaz.

## 4. EVE penceresini seçme

- Uygulama kullanılabilir pencerelerin listesini gösterecek.
- Kullanıcı listeden EVE Online penceresini seçecek.
- Yalnızca seçilen pencere yakalanacak.
- Pencere hareket ettiğinde veya boyutu değiştiğinde çeviri katmanı onu takip edecek.
- EVE arka plana geçtiğinde yakalama ve çeviri duraklatılacak.
- EVE Launcher, hesap giriş ekranı ve diğer uygulamalar kendiliğinden çevrilmeyecek.

## 5. Teknik yapı

- **Swift ve SwiftUI:** Yerel macOS arayüzü
- **ScreenCaptureKit:** Yalnızca seçilen EVE penceresini veya alanı yakalama
- **Vision OCR:** Görüntüdeki İngilizce metni cihaz üzerinde tanıma
- **Apple Translation:** İngilizce metni Türkçeye cihaz üzerinde çevirme
- **Şeffaf macOS paneli:** Çeviriyi oyunun üzerinde, tıklamaları engellemeden gösterme

Minimum hedef macOS 15.0'dır. Bunun nedeni planlanan Apple Translation API'sinin bu sürümden itibaren kullanılabilir olmasıdır.

## 6. EVE arayüzüne uyumlu görünüm

Çevirinin yabancı bir beyaz kutu gibi görünmemesi temel gereksinimdir.

Uygulama:

- EVE'ye benzeyen koyu ve yarı saydam bir arka plan kullanacak.
- Metnin yaklaşık yazı boyutunu, rengini, hizasını ve satır aralığını dikkate alacak.
- Türkçe metni mümkün olduğunca orijinal metnin bulunduğu konumda gösterecek.
- Uzun Türkçe cümleleri yeniden satırlara bölecek; gerekirse yazıyı sınırlı ölçüde küçültecek.
- Her satır için ayrı kutu oluşturmak yerine ilişkili satırları tek paragraf hâlinde birleştirecek.
- Oyuna ait yazı tipi veya başka bir varlığı kopyalamayacak; lisans sorunu oluşturmayan yakın bir sistem yazı tipi kullanacak.

İki görüntüleme biçimi planlanmaktadır:

1. **EVE Stili:** Sabit ve koyu görev/bilgi panellerinde çeviri orijinal metin bölgesinde görünür.
2. **Temiz Panel:** Hareketli görüntü veya karmaşık HUD üzerinde çeviri, metnin yanında tek koyu panelde görünür.

Uygulama uygun olmayan bir bölgede orijinal yazıyı örtmeye çalışmayacak.

## 7. Ekran kalabalığını önleyen filtreler

Aşağıdaki içerikler varsayılan olarak çevrilmeyecek:

- Yalnızca sayılardan oluşan metinler
- Çok kısa düğme etiketleri
- Oyuncu, karakter, gemi ve şirket adları
- ISK miktarları ve koordinatlar
- Yeterli güvenle okunamayan yazılar
- Zaten Türkçe olan metinler
- Bir önceki görüntüyle aynı olan metinler
- Kullanıcının seçtiği alanın dışında kalan yazılar

Yeni çeviri gösterilmeden önce önceki panel temizlenecek. Aynı anda çok sayıda çeviri katmanı oluşmasına izin verilmeyecek.

## 8. EVE terimleri sözlüğü

Yerel bir sözlük, makine çevirisinin oyun terimlerini bozmasını önleyecek.

- Gemi ve modül adları korunacak.
- ISK, NPC, corporation, agent ve fitting gibi EVE terimleri için tercih edilen karşılıklar saklanacak.
- Kullanıcı kendi kelimelerini ekleyebilecek.
- Aynı terimin farklı yerlerde farklı çevrilmesi azaltılacak.

İlk sürüm küçük bir temel sözlükle başlayacak ve gerçek kullanım sırasında genişletilecek.

## 9. Güvenlik ve gizlilik sınırları

- Ekran görüntüleri diske kaydedilmeyecek.
- Görüntüler yalnızca geçici bellekte işlenecek.
- Varsayılan çeviri motoru Apple'ın cihaz üzerindeki sistemi olacak.
- Parola, doğrulama kodu veya hesap bilgisi okunmayacak.
- EVE giriş ekranında çeviri otomatik olarak duracak.
- Mikrofon ve sistem sesi izni istenmeyecek.
- Erişilebilirlik izni yalnızca gerçekten zorunluysa istenecek.
- Otomatik tıklama, makro ve oyun kontrolü eklenmeyecek.
- Oyun istemcisine kod enjekte edilmeyecek.
- Gerçek kullanım veya dağıtımdan önce EVE'nin güncel üçüncü taraf yazılım kuralları ayrıca kontrol edilecek.

## 10. Performans yaklaşımı

- Bütün EVE penceresi sürekli ve yüksek hızda analiz edilmeyecek.
- Fareyle çeviri yalnızca kullanıcı tetiklediğinde çalışacak.
- Sabit alan modu düşük yenileme hızında çalışacak.
- Görüntü değişmediyse OCR ve çeviri tekrarlanmayacak.
- Aynı metnin önceki çevirisi yerel önbellekten gösterilecek.
- Küçük ve gereksiz metinler OCR işlemine alınmayacak.
- Kullanıcı tek tuşla bütün işlemi duraklatabilecek.

Amaç, EVE'nin performansını ve Mac'in enerji kullanımını mümkün olduğunca az etkilemektir.

## 11. İlk kullanılabilir sürüm

İlk kullanılabilir sürümde hedeflenenler:

1. EVE Online penceresini seçme
2. Pencereyi hareket ederken takip etme
3. Fare çevresindeki İngilizce paragrafı algılama
4. İngilizce–Türkçe Apple çevirisi
5. Tek, koyu ve tıklama geçirgen çeviri paneli
6. Çeviriyi gösterip gizleyen ayarlanabilir kısayol
7. Sabit bir görev metni alanı seçebilme
8. Aynı metni yeniden çevirmeyi önleyen basit önbellek
9. Temel EVE terimleri sözlüğü
10. Tek tuşla tamamen duraklatma

## 12. İlk sürümde olmayacaklar

- Ses tanıma veya sistem sesi çevirisi
- Toplantı ya da video altyazısı
- Bulut tabanlı yapay zekâ servisi
- Kullanıcı hesabı veya abonelik sistemi
- Otomatik başlangıç
- Birden fazla oyunu aynı anda izleme
- Tam ekrandaki bütün metinleri otomatik kaplama
- Oyuna komut gönderme veya makro
- İlk aşamada App Store yayını

## 13. Tamamlanma ölçütleri

İlk kullanılabilir sürüm şu şartları sağlamalıdır:

- Yalnızca kullanıcının seçtiği EVE penceresini okur.
- Her yazının üzerinde ayrı kutu oluşturmaz.
- Aynı anda yalnızca tek anlaşılır çeviri gösterir.
- Görev paragraflarını doğru sırada okuyabilir.
- Türkçe çeviri EVE arayüzüne görsel olarak uyumludur.
- Çeviri katmanı oyundaki tıklamaları engellemez.
- Tek tuşla anında gizlenebilir.
- EVE arka plandayken ekran okumayı durdurur.
- Hesap ve giriş ekranını yakalamaz.
- Oyunun dosyalarına, belleğine ve kontrollerine dokunmaz.
- Oyunun performansını belirgin biçimde düşürmez.

## 14. Ana ilke

> Uygulama oyunu çeviri kutularıyla kaplamayacak. Çeviri yalnızca kullanıcının istediği metin için, tek ve EVE arayüzüne uyumlu bir panelde gösterilecek.
