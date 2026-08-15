# Glowo'yu iOS uygulamasına (.ipa) çevirme rehberi

Bu klasör, `glowo_app.html` dosyasını gerçek bir iOS uygulamasına
dönüştürmek için hazırlanmış bir **Capacitor** projesidir. Capacitor,
bir web uygulamasını (HTML/CSS/JS) gerçek bir iOS/Android uygulamasının
içine "sarıp" App Store'a/Play Store'a yüklenebilir hale getiren, dünyada
en çok kullanılan açık kaynak araçlardan biridir.

## Önce şunu bilmen önemli

iOS uygulaması **derlemek** (build) ve **imzalamak** (code signing) için
Apple'ın kuralı gereği bir Mac ve Xcode gerekir - bu, Windows'ta hiçbir
şekilde yapılamaz, tek istisnası **bulutta bir Mac kiralamak**tır. Bu yüzden
bu rehber, Windows'tan da kullanabileceğin bulut tabanlı bir servis olan
**Codemagic** üzerinden ilerliyor: sen kodu bir GitHub deposuna yüklüyorsun,
Codemagic'in buluttaki Mac'i uygulamayı derleyip imzalıyor ve sana/App
Store'a teslim ediyor.

Ayrıca, Windows'tan da olsa, App Store'a yayınlamak için mutlaka gerekenler:

- **Apple Developer Program** üyeliği - yıllık 99$ (developer.apple.com)
- Bir **GitHub** hesabı (ücretsiz) - kodu buraya yükleyeceksin
- Bir **Codemagic** hesabı (ücretsiz katmanı var, ayda 500 dakika derleme) - codemagic.io

## Klasördeki dosyalar

- `www/index.html` - Glowo uygulamasının kendisi (tek dosya)
- `package.json`, `capacitor.config.json` - Capacitor proje ayarları
- `resources/icon.png`, `resources/splash.png` - uygulama ikonu ve açılış ekranı kaynak görselleri (bunlardan otomatik olarak tüm boyutlar üretilecek)
- `codemagic.yaml` - Codemagic için hazır bir derleme şablonu (Codemagic kendi sihirbazıyla da benzerini üretebilir, ikisini karşılaştırıp kullan)

## Adım adım

### 1) Kodu GitHub'a yükle

1. github.com'da ücretsiz bir hesap aç (yoksa).
2. Sağ üstten "New repository" ile `glowo` adında yeni, **private** bir depo oluştur.
3. Depo sayfasında "uploading an existing file" linkine tıkla, bu klasördeki
   **tüm dosya ve klasörleri** (www, resources, package.json, capacitor.config.json,
   codemagic.yaml, README.md, .gitignore) sürükleyip bırak, "Commit changes" de.
   (Git komutu bilmene gerek yok, tarayıcıdan sürükle-bırak yeterli.)

### 2) Apple Developer hesabını hazırla

1. developer.apple.com üzerinden Apple Developer Program'a kaydol (99$/yıl,
   kendi Apple ID'inle, kimlik doğrulaması birkaç gün sürebilir).
2. Onay geldikten sonra appstoreconnect.apple.com adresine git,
   **My Apps > + > New App** ile yeni bir uygulama kaydı oluştur.
   - Bundle ID olarak `com.yusuf.glowo` seç (Apple Developer portalında
     önce **Certificates, Identifiers & Profiles > Identifiers** kısmından
     bu Bundle ID'yi kaydetmen gerekebilir - "com.yusuf.glowo" istersen
     kendi tercihine göre değiştirebilirsin, ama o zaman
     `capacitor.config.json` içindeki `appId` alanını da aynı şekilde
     güncellemelisin).
3. App Store Connect içinde **Users and Access > Integrations > App Store
   Connect API** kısmından bir API key oluştur ve indir - Codemagic bunu
   kullanarak senin adına imzalama/yükleme işini otomatik yapacak.

### 3) Codemagic'i bağla

1. codemagic.io adresine git, GitHub hesabınla giriş yap.
2. "Add application" ile yukarıda oluşturduğun `glowo` deposunu seç.
   Codemagic projenin bir Capacitor projesi olduğunu otomatik algılar.
3. Sol menüden **Team settings > Integrations > App Store Connect** kısmına
   girip 2. adımda indirdiğin API key'i yükle, ona bir isim ver
   (örn. `codemagic_api_key` - bu isim `codemagic.yaml` dosyasındaki isimle
   birebir aynı olmalı).
4. Codemagic'in kendi arayüzünden "Capacitor" iş akışı (workflow) şablonunu
   seçebilir ya da bu depodaki `codemagic.yaml` dosyasını kullanabilirsin
   (Codemagic depoda böyle bir dosya bulursa otomatik onu kullanır).
5. **Start new build** ile ilk derlemeni başlat. İlk denemede küçük
   hatalar/uyarılar çıkması normal - Codemagic'in build logu tam olarak
   nerede takıldığını gösterir, o ekran görüntüsünü bana gönderirsen
   düzeltmene yardım ederim.
6. Derleme başarılı olursa Codemagic sana indirilebilir bir `.ipa` dosyası
   verir, istersen otomatik olarak TestFlight'a da yükleyebilir
   (`codemagic.yaml` içinde `submit_to_testflight: true` zaten açık).

### 4) TestFlight ile test et, sonra App Store'a gönder

1. TestFlight uygulamasını bir iPhone'a indirip kendi Apple hesabınla
   giriş yaparak Glowo'yu telefonunda gerçek bir uygulama gibi test edebilirsin.
2. Her şey yolundaysa App Store Connect'te uygulama sayfana girip
   ekran görüntüleri, açıklama, gizlilik politikası linki, yaş sınırı gibi
   bilgileri doldurup **Submit for Review** dersin.

## Apple incelemesi öncesi dikkat etmen gerekenler

Glowo kullanıcıların video paylaştığı bir uygulama olduğu için, Apple
İnceleme Kuralları'nın 1.2 maddesi ("User Generated Content") gereği
şunlar genelde **zorunlu**:

- Uygunsuz içeriği **şikayet etme/bildirme** butonu
- Kullanıcı **engelleme** (block) özelliği
- Bir **gizlilik politikası** sayfası (URL olarak App Store Connect'e eklenmeli)
- Uygulama içi kayıtlı bir **kullanım şartları** metni
- Uygun bir **yaş sınırı** (age rating) seçimi

Şu anki Glowo, bunlar olmadan da TestFlight ile test edilebilir, ama
gerçek App Store yayını için Apple incelemesi muhtemelen bu maddeleri
isteyecektir. İstersen bir sonraki adımda bu özellikleri (bildir/engelle
butonları, gizlilik politikası sayfası vb.) uygulamaya ekleyebilirim.

Ayrıca unutma: şu anki Glowo'da videolar sadece senin cihazının hafızasında
tutuluyor - yani iki farklı telefonda Glowo açan iki kişi birbirinin
videosunu **göremez**. Gerçek bir sosyal uygulama için videoların ortak bir
sunucuda (backend) tutulması gerekir - bu da ayrı bir geliştirme aşaması.
İstersen bunu da birlikte planlayabiliriz.

## Yerel olarak (Windows'ta) deneme

Xcode gerekmeden, sadece web sürümünü doğrudan tarayıcıda test etmek için
`www/index.html` dosyasını çift tıklayıp açman yeterli - bu, App Store'a
gitmeden önce hızlıca kontrol etmek için pratik bir yöntem.
