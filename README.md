bu repo ibrahim kavasoğluna aittir.

## Tek iş akışlı çoklu mağaza yayını

Tüm Windows/Android paket üretimi, ortak sürüm doğrulaması, SHA-256 kontrolleri, GitHub Release, `latest.json` ve Android mağaza gönderimleri tek dosyada yürütülür:

`.github/workflows/build-publish-ses-merkezi.yml`

Windows EXE/MSIX ve Android APK/AAB derlemeleri paralel çalışır. Dört paket ve manifest doğrulanmadan yayın adımı başlamaz. Yayın tamamlanınca mağaza gönderimleri ayrı matrix görevleri olarak paralel yürütülür. Bir mağazanın incelemesi diğer mağazanın yüklemesini bekletmez.

### Desteklenen mağazalar

| Mağaza | Paket | Otomasyon durumu |
|---|---|---|
| Samsung Galaxy Store | AAB | Resmî Seller API / Fastlane ile yükleme ve incelemeye gönderme |
| Huawei AppGallery | AAB | AppGallery Connect Publishing API / Fastlane ile yükleme ve incelemeye gönderme |
| Xiaomi GetApps (Redmi/POCO) | APK | Resmî otomatik yayın API'si ile yükleme |
| vivo V-Appstore | APK | Resmî Publishing API ile yükleme ve incelemeye gönderme |
| OPPO App Market | APK | Ücretsiz geliştirici hesabı var; resmî açık binary-yayın API'si doğrulanmadı, Seller Console üzerinden ilk kayıt/yükleme gerekiyor |
| HONOR App Market | APK | Geliştirici kimlik/kurumsal yeterlilik doğrulaması ve manuel ilk kayıt gerekiyor; otomatik binary API'si bu iş akışına eklenmedi |

Mağazaların tamamı uygulamayı ilk kez kaydetme, sahiplik/kimlik doğrulama, içerik derecelendirmesi, gizlilik metni, ekran görüntüleri ve yayın sözleşmesi gibi işlemleri kendi panellerinde isteyebilir. Otomasyon, mağaza hesabı ve uygulama kaydı hazır olduktan sonraki sürüm yükleme/incelemeye gönderme adımlarını kapsar. İnceleme süresi ve mağazanın nihai onayı otomatikleştirilemez.

### Mağaza otomasyonunu açma

GitHub `Stagepulse-Releases` deposunda Settings → Secrets and variables → Actions bölümünde aşağıdaki Repository Variables ve Secrets tanımlanır. Mağaza hesabı hazır değilse ilgili `*_ENABLED` değişkeni `true` yapılmaz; o mağaza işi güvenli biçimde atlanır.

**Repository Variables**

- `SAMSUNG_GALAXY_STORE_ENABLED` = `true`; `SAMSUNG_CONTENT_ID` = Seller Portal'daki 12 haneli Content ID.
- `HUAWEI_APPGALLERY_ENABLED` = `true`; `HUAWEI_APP_ID` = AppGallery Connect App ID; `HUAWEI_PRIVACY_POLICY_URL` = onaylı gizlilik politikası adresi.
- `XIAOMI_GETAPPS_ENABLED` = `true`; `XIAOMI_APP_CATEGORY_ID` = GetApps kategori numarası (varsa).
- `VIVO_APPSTORE_ENABLED` = `true`; `VIVO_NATION_CODES` = örneğin `in,id`; `VIVO_CATEGORY_ID` = V-Appstore kategori numarası; `VIVO_PRIVACY_POLICY_URL` = onaylı gizlilik politikası adresi.

**Repository Secrets**

- Samsung: `SAMSUNG_ACCESS_TOKEN`, `SAMSUNG_SERVICE_ACCOUNT_ID`. Seller Portal'da Publishing kapsamlı Service Account oluşturulmalı; AAB için Samsung tarafında aynı mevcut APK imza anahtarı kullanılmalı veya mevcut imza anahtarı güvenli şekilde aktarılmalı.
- Huawei: `HUAWEI_CLIENT_ID`, `HUAWEI_CLIENT_SECRET`.
- Xiaomi: `XIAOMI_USERNAME`, `XIAOMI_API_PASSWORD`, `XIAOMI_RSA_PUBLIC_KEY`. Buradaki API parolası Xiaomi'nin API için ürettiği anahtardır; hesap giriş parolası kullanılmamalıdır.
- vivo: `VIVO_ACCESS_KEY`, `VIVO_ACCESS_SECRET`. Publishing API yetkisi vivo geliştirici hesabında açılmış olmalıdır.
- Private kaynak erişimi: `SOURCE_REPO_READ_TOKEN`. Fine-grained token, yalnızca `Stagepulse-Ses-Merkezi` için `Contents: Read` yetkisi taşımalıdır.

Kaynak repo `Stagepulse-Ses-Merkezi` Private yapıldığında bu salt-okunur token ile checkout devam eder. Kaynak depodaki `RELEASE_REPO_DISPATCH_TOKEN` ise yalnızca `Stagepulse-Releases` deposuna `repository_dispatch` gönderme yetkisi taşımalıdır. Token'lar workflow dosyalarına yazılmaz.

### Güncelleme bildirimi

Kararlı sürüm yayımlandığında `latest.json` ve Supabase güncelleme servisi aynı sürümle güncellenir. Uygulamanın Hesap menüsündeki mevcut güncelleme kontrolü bu manifesti okur ve yeni sürüm bildirimi verir. Mağazalara yükleme incelemeye gönderilir; mağaza kullanıcılarına mağazanın kendi güncelleme bildirimi ancak ilgili mağaza sürümü onaylayıp yayımladıktan sonra gider. Bu iki bildirim kanalı birbirinden bağımsızdır.

