# EVE Online Çeviri TR

EVE Online Çeviri TR, macOS üzerinde çalışan yerel ve isteğe bağlı bir ekran çeviri yardımcısı olarak planlanmaktadır. Amaç, seçilen EVE Online penceresindeki İngilizce metni Türkçeye çevirirken oyunun arayüzünü çeviri kutularıyla kaplamamaktır.

> **Mevcut durum: Başlangıç iskeleti.** Derlenebilir macOS SwiftUI projesi oluşturuldu. Pencere seçimi, ekran yakalama, OCR, Apple çevirisi ve oyun üstü panel henüz uygulanmadı.

## Temel yaklaşım

- Kullanıcı çevrilecek EVE Online penceresini kendisi seçecek.
- Varsayılan mod bütün ekranı otomatik çevirmeyecek.
- Çeviri, kullanıcı bir yardımcı tuşa basılı tuttuğunda farenin yakınındaki tek paragraf için gösterilecek.
- Görev açıklaması gibi bölümler için isteğe bağlı sabit alan modu bulunacak.
- Aynı anda yalnızca tek, koyu ve tıklama geçirgen çeviri paneli kullanılacak.
- Temel OCR ve çeviri işlemleri Apple çerçeveleriyle cihaz üzerinde yapılacak.
- Oyun dosyalarına, çalışan oyun işlemine ve kullanıcı girişlerine müdahale edilmeyecek.

Ayrıntılı ürün ve güvenlik planı için [PLAN.md](PLAN.md), aşamalar için [ROADMAP.md](ROADMAP.md), kayıt altına alınan değişiklikler için [CHANGELOG.md](CHANGELOG.md) dosyasına bakın.

## Proje düzeni

- **Kaynak kod:** Ayrı, başlangıçta özel tutulan kaynak deposu
- **Dokümantasyon:** Bu depo; plan, yol haritası ve doğrulanmış ilerleme kayıtları

## Teknik yön

- Native macOS ve SwiftUI
- Minimum macOS 15.0
- ScreenCaptureKit ile seçili pencere/alan
- Vision ile cihaz üzerinde OCR
- Apple Translation ile İngilizce–Türkçe çeviri
- Oyunun üzerinde tıklamaları engellemeyen şeffaf panel
- Haricî paket, kullanıcı hesabı, abonelik veya ücretli servis yok

## Bağımsızlık bildirimi

Bu, topluluk tarafından geliştirilen bağımsız ve resmî olmayan bir yardımcı uygulama projesidir. CCP Games ile bağlantılı, onun tarafından desteklenen veya onaylanan bir proje değildir. EVE Online adı ve ilgili işaretler kendi hak sahiplerine aittir. Projede oyuna ait görsel, yazı tipi veya başka varlıklar kullanılmaz.
