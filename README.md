# Stagepulse Ses Merkezi Sürümleri

Bu depo, Stagepulse Ses Merkezi'nin herkese açık kurulum paketlerini ve kararlı sürüm kayıtlarını barındırır.

## Dağıtım dosyaları

Her kararlı GitHub Release şu dosyaları içerir:

- `Stagepulse-Ses-Merkezi-Windows-<sürüm>.exe`: Windows kurulum paketi
- `Stagepulse-Ses-Merkezi-Windows-<sürüm>.exe.sha256`: Windows SHA-256 özeti
- `Stagepulse-Ses-Merkezi-Android-<sürüm>.apk`: İmzalı Android APK
- `Stagepulse-Ses-Merkezi-Android-<sürüm>.apk.sha256`: Android SHA-256 özeti
- `latest.json`: Sürüm, platform indirme adresleri ve iki SHA-256 değerini içeren doğrulanmış ortak bildirim

## Güncelleme bildirimi

Uygulamalar sürüm ve dosya bütünlüğü bilgisini Stagepulse Supabase Edge Function üzerinden alır:

`https://itasctyjzyewcykozuex.supabase.co/functions/v1/ses-merkezi-updates`

Bu uç nokta GitHub Release içindeki sabit `latest.json` dosyasını okur ve sürüm, indirme alan adı, dosya adı ve SHA-256 alanlarını doğrular. GitHub API kotasına bağlı değildir. Lisans, kullanıcı veya yönetim verisi içermez.

## Güvenlik

- Dağıtım dosyaları CI tarafından üretilir.
- SHA-256 özetleri her dosya için CI tarafından hesaplanır.
- Android paketi kalıcı Stagepulse imzalama anahtarıyla imzalanmalıdır.
- İmzalama anahtarı veya Supabase servis anahtarı bu depoya kesinlikle eklenmez.


## Otomatik doğrulama

`.github/workflows/test-update-manifest.yml`, Supabase sürüm bildirim uç noktasını kimlik doğrulaması olmadan çağırır; yayın yokken boş sürüm yanıtını, yayın olduğunda Windows/Android indirme adresleriyle SHA-256 alanlarını kontrol eder. Aynı akış HTTP yöntem ve CORS davranışını da denetler.


## İlk kurulum: güvenli yayın yetkileri

Yayın deposunun iş akışları kaynak kodu bu özel depodan salt-okunur olarak alır. Bu nedenle `Stagepulse-Releases` deposunda **Settings → Secrets and variables → Actions** bölümüne aşağıdaki değerler eklenmelidir.

| Secret adı | Kullanım |
|---|---|
| `SOURCE_REPO_READ_TOKEN` | `Stagepulse-Ses-Merkezi` özel deposuna yalnızca **Contents: Read-only** ve Metadata: Read-only erişimli Fine-grained PAT |
| `ANDROID_SIGNING_KEYSTORE_BASE64` | Android uygulamasının kalıcı imzalama anahtar dosyasının Base64 metni |
| `ANDROID_SIGNING_STORE_PASSWORD` | Keystore parolası |
| `ANDROID_SIGNING_KEY_ALIAS` | Keystore içindeki anahtar takma adı |
| `ANDROID_SIGNING_KEY_PASSWORD` | İmzalama anahtarının parolası |
| `SUPABASE_ACCESS_TOKEN` | Yalnızca güncelleme fonksiyonunu yeniden dağıtmak için Supabase erişim belirteci |

`SOURCE_REPO_READ_TOKEN` oluşturulurken yalnızca `ibrahimFOH/Stagepulse-Ses-Merkezi` deposu seçilmeli; yönetim, yazma, Actions veya diğer depo izinleri verilmemelidir. Yayın deposunun kendi Release oluşturma yetkisi, iş akışına sınırlı `GITHUB_TOKEN` izniyle sağlanır.

### Android imza anahtarı için önemli uyarı

Android güncellemelerinin mevcut kurulumun üzerine kurulabilmesi için APK'nın önceki sürümle **aynı imza sertifikası** kullanması gerekir. Daha önce kullanılan kalıcı `.jks` / `.keystore` dosyası varsa onu kullanın. Yeni ve farklı bir anahtar üretmek mevcut uygulamanın üzerine güncelleme kurulmasını engeller; bu durumda kullanıcıların uygulamayı kaldırıp yeniden kurması gerekir. İmza anahtarını kaybetmeyin ve depoya, ZIP dosyasına veya kaynak koda eklemeyin.

Mevcut keystore dosyasını Base64 metnine çevirmek için Windows PowerShell'de (dosya yolunu kendinize göre değiştirerek) şu komut kullanılabilir:

```powershell
[Convert]::ToBase64String([IO.File]::ReadAllBytes("C:\Stagepulse\stagepulse-release.jks")) | Set-Clipboard
```

Panodaki metni yalnızca `ANDROID_SIGNING_KEYSTORE_BASE64` secret alanına yapıştırın. Keystore parolalarını ayrı secret alanlarına kaydedin. Parolaları sohbet veya depoya yazmayın.

### Sürüm yayımlama

1. Tüm zorunlu secrets eklendikten sonra **Actions → Stagepulse Ses Merkezi kararlı sürümünü oluştur ve yayımla → Run workflow** açın.
2. Sürüm numarasını `3.7.1` biçiminde girin. `v` öneki girmeyin.
3. Kaynak dalı olarak PR birleştirilmeden önce `integration/3.7.0-shared-core-demo`, birleştirmeden sonra `main` seçilebilir.
4. Windows testleri, APK imzası ve dosya özetleri doğrulanmadan Release yayımlanmaz.

Güncelleme fonksiyonunun kodu değiştiğinde **Actions → Ses Merkezi güncelleme servisini dağıt** iş akışı ayrıca çalıştırılır. Bu iş akışı mevcut lisans fonksiyonlarına dokunmaz.
