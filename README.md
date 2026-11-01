# Stagepulse Ses Merkezi Sürümleri

Bu depo, Stagepulse Ses Merkezi'nin herkese açık kurulum paketlerini ve kararlı sürüm kayıtlarını barındırır.

## Dağıtım dosyaları

Her kararlı GitHub Release şu dosyaları içerecek:

- `Stagepulse-Ses-Merkezi-Windows-<sürüm>.exe`: Windows kurulum paketi
- `Stagepulse-Ses-Merkezi-Windows-<sürüm>.exe.sha256`: Windows SHA-256 özeti
- `Stagepulse-Ses-Merkezi-Android-<sürüm>.apk`: İmzalı Android APK
- `Stagepulse-Ses-Merkezi-Android-<sürüm>.apk.sha256`: Android SHA-256 özeti

## Güncelleme bildirimi

Uygulamalar sürüm ve dosya bütünlüğü bilgisini Stagepulse Supabase Edge Function üzerinden alır:

`https://itasctyjzyewcykozuex.supabase.co/functions/v1/ses-merkezi-updates`

Bu uç nokta yalnızca herkese açık GitHub Release varlıklarını okur. Lisans, kullanıcı veya yönetim verisi içermez.

## Güvenlik

- Dağıtım dosyaları CI tarafından üretilir.
- SHA-256 özetleri her dosya için CI tarafından hesaplanır.
- Android paketi kalıcı Stagepulse imzalama anahtarıyla imzalanmalıdır.
- İmzalama anahtarı veya Supabase servis anahtarı bu depoya kesinlikle eklenmez.
