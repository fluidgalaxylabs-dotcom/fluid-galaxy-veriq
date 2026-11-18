# VERIQ – Gizlilik Politikası (Google Play Yayın Hazırlığı)

> **Önemli Durum Notu:** Bu gizlilik politikası `C:\Users\baysa\AndroidStudioProjects\CodeFlow` kaynak kodu incelenerek hazırlanmış ve kodda yer alan gerçek davranışlarla birebir doğrulanmıştır. Google Play Store yayını öncesinde **Kamuya Açık Web URL'si** alanı geliştirici tarafından herkese açık bir HTTPS adresiyle doldurulmalıdır.

---

## 1. Uygulama ve Geliştirici Bilgisi

- **Uygulama Adı:** VERIQ
- **Uygulama Kimliği (`applicationId`):** `com.fluidgalaxy.codeflow`
- **Geliştirici / Şirket:** Fluid Galaxy Labs
- **Resmi İletişim ve Destek E-postası:** [fluidgalaxylabs@gmail.com](mailto:fluidgalaxylabs@gmail.com)
- **Sürüm:** `1.0` (`versionCode: 1`)
- **Kamuya Açık Gizlilik Politikası URL'si:** `[YAYIN ÖNCESİ DOLDURULACAK – Herkese açık HTTPS URL]`

---

## 2. Bildirim Erişimi ve OTP Algılama

VERIQ, `OtpNotificationListenerService` üzerinden Android'in sistem **Bildirim Erişimi (`BIND_NOTIFICATION_LISTENER_SERVICE`)** yetkisiyle yalnızca kullanıcının yetkilendirdiği bildirimlere erişir. **Bildirim erişimi izni verilmeden uygulamanın OTP algılama özelliği çalışmaz.**

- **Desteklenen Kaynaklar (`SourceRegistry`):** SMS/mesajlaşma uygulamaları (Google Messages, Samsung Messages vb.), WhatsApp (`com.whatsapp`, `com.whatsapp.w4b`) ve desteklenen e-posta uygulamaları (Gmail, Outlook, Samsung Email vb.). Sosyal medya, oyun ve diğer desteklenmeyen uygulamaların bildirimleri taranmadan atlanır.
- **Yalnızca Cihaz Üzerinde İşleme (`OtpParser`):** Desteklenen bildirimlerin başlık (`EXTRA_TITLE`) ve metin (`EXTRA_TEXT`, `EXTRA_BIG_TEXT`) alanları yalnızca OTP kodunu ayrıştırmak amacıyla cihazın belleğinde (RAM) anlık olarak işlenir.
- **OTP Dışındaki İçeriklerin Korunması:** Bildirim gövdeleri, telefon numaraları, e-posta adresleri veya OTP dışındaki kişisel metinler hiçbir veritabanına veya kalıcı dosyaya kaydedilmez, başka amaçla kullanılmaz ve hiçbir sunucuya gönderilmez.
- **Kaynak Mesaj Bütünlüğü:** VERIQ kaynak uygulamalardaki (SMS, WhatsApp, Gmail, Outlook vb.) mesajların kendisini **silmez, dışarı göndermez veya içeriğini değiştirmez**.
- **İsteğe Bağlı Otomatik Okundu İşaretleme (`NotificationMarkAsReadHelper`):** Ayarlar ekranındaki **"Otomatik Okundu İşaretle"** (`auto_mark_as_read`) tercihi **varsayılan olarak kapalıdır**. Yalnızca kullanıcı bu ayarı manuel olarak açarsa ve algılanan OTP panoya başarıyla kopyalanırsa, kaynak bildirimde açıkça güvenli `"Okundu"` / `"Mark as read"` `PendingIntent` aksiyonu bulunup bulunmadığı kontrol edilir ve yalnızca bir kez tetiklenir. Yanıtla (`Reply`), Arşivle (`Archive`), Sil (`Delete`) veya Okunmadı (`Unread`) aksiyonları kesinlikle çalıştırılmaz; bildirim kimliği yalnızca RAM içi 128 kapasiteli kümede tutulur, mesaj gövdesi veya OTP kodu saklanmaz.
- **VERIQ Bildirim İşlemleri (`CopyOtpReceiver`):** VERIQ'in gösterdiği kendi OTP bildirimlerindeki **"Bildirimi Kapat"** aksiyonu yalnızca VERIQ bildirimini kapatır; **"Kopyala"** aksiyonu yalnızca o bildirime ait OTP kodunu tekrar panoya yazar. İkisi de kaynak uygulamalara veya VERIQ OTP geçmişine dokunmaz.

---

## 3. Pano (Clipboard) Kullanımı

- **Panoya Yazma (`OtpClipboardHelper`):** Algılanan OTP kodu, **"Otomatik Kopyalama"** (`auto_copy`) ayarı açıksa veya kullanıcı Ana Sayfa'daki / VERIQ bildirimindeki **Kopyala** butonuna dokunursa `ClipboardManager.setPrimaryClip` aracılığıyla panoya yazılır.
- **Otomatik Pano Temizleme ve Pano Verilerinin Korunması:** Kullanıcının seçtiği süre dolduğunda (`30 Saniye`, `1 Dakika`, `5 Dakika`; `Kapalı` seçeneğinde zamanlayıcı çalışmaz) VERIQ, kullanıcının bu sürede panoya kopyaladığı başka kişisel verileri yanlışlıkla silmemek için bellek içi eşitlik kontrolü amacıyla `ClipboardManager.primaryClip` değerini okuyup son kopyalanan OTP koduyla karşılaştırır. Panoda farklı bir içerik varsa panoya hiç dokunulmaz; panodan okunan diğer kullanıcı verileri asla incelenmez, saklanmaz, loglanmaz veya sunucuya gönderilmez. Yalnızca panoda hâlâ aynı OTP kodu duruyorsa pano boşaltılır (`""`).
- **Android Sistem Davranışları:** Android 10+ sürümlerinde uygulama arka plandayken sistem pano okuma/yazma erişimini kısıtlayabilir; Android 12+ (API 31+) sürümlerinde ise sistem pano erişimi gerçekleştiğinde ekranda sistem bildirim uyarısı gösterebilir. VERIQ sistem kısıtlaması durumunda çökmeden hatayı güvenle yoksayar.

---

## 4. OTP Geçmişi ve Yerel Depolama

VERIQ tüm kalıcı tercihlerini ve yerel kayıtlarını **Android Jetpack Preferences DataStore** (`otp_clipboard_settings.preferences_pb`) içinde yalnızca kullanıcının kendi cihazındaki uygulama özel depolama alanında tutar:

- **OTP Geçmişi (`otp_history_entries`):** Güvenlik gerekçesiyle yalnızca algılama zaman damgasını (`timestampMs`), kaynak kategorisini (`SMS`, `WHATSAPP`, `EMAIL`) ve işlem durumunu (`COPIED`, `AUTO_PASTED`, `DETECTED`) saklar. En fazla `200` kayıt tutulur ve kullanıcının seçtiği süreye göre (`1 Gün`, `7 Gün`, `30 Gün` veya `Süresiz`) otomatik budanır. **OTP geçmişinde gerçek doğrulama kodları veya mesaj içerikleri saklanmaz.**
- **Son Algılanan OTP Alanları (`last_otp_code`, `last_otp_source`, `last_otp_timestamp`):** Ana Sayfa'daki *"Son Algılanan OTP"* kartında görüntüleme ve manuel yeniden kopyalama amacıyla yerel DataStore içinde tutulur; kullanıcı Ana Sayfa'daki **Temizle** butonuna dokunduğunda silinir.
- **Kullanıcı Tercihleri ve Sayaçlar:** `auto_copy`, `show_notification`, `vibration`, `auto_mark_as_read`, `clear_timeout`, `sms_enabled`, `whatsapp_enabled`, `email_enabled`, `daily_count`, `last_count_day`, `history_retention_days`, `has_seen_disclaimer` ve `has_accepted_notification_disclosure` değerleri yerel DataStore'da tutulur.
- **Sunucuya Aktarım Yokluğu:** Kod incelemesiyle doğrulandığı üzere DataStore'da tutulan hiçbir OTP geçmişi kaydı, ayar veya yerel veri hiçbir sunucuya ya da üçüncü tarafa gönderilmez.
- **Bulut Yedekleme ve Aktarım Engeli:** `AndroidManifest.xml` içinde `android:allowBackup="false"` tanımlıdır; `backup_rules.xml` ve `data_extraction_rules.xml` dosyaları üzerinden Android Cloud Backup ve cihazlar arası aktarım (Device-to-Device Transfer) kapsamında tüm uygulama verileri hariç tutulmuştur.
- **Verilerin Silinmesi:**
  - Ana Sayfa > **Temizle** son algılanan OTP kodunu ve kaynağını temizler (`"-"` / `0`).
  - Geçmiş > **Tüm Geçmişi Temizle** tüm OTP geçmişini kalıcı olarak siler.
  - Kullanıcı Android Sistem Ayarları üzerinden **Uygulama Verilerini Temizle** işlemini yaptığında veya **VERIQ'i cihazdan kaldırdığında** DataStore dosyası dahil tüm yerel uygulama verileri kalıcı olarak silinir.

---

## 5. GitHub Üzerinden Uzaktan Sürüm Kontrolü

VERIQ `AndroidManifest.xml` dosyasında yalnızca uzaktan minimum sürüm doğrulaması amacıyla `android.permission.INTERNET` ve `android.permission.ACCESS_NETWORK_STATE` izinlerini içerir (`AppVersionChecker.kt`):

- Uygulama her açılışında ve arka plandan ön plana döndüğünde (`ON_RESUME`) `ConnectivityManager` ile internet bağlantısı kontrol edilir ve aşağıdaki adrese önbelleksiz bir **HTTPS GET** isteği gönderilir:
  - `https://raw.githubusercontent.com/fluidgalaxylabs-dotcom/fluid-galaxy-veriq/main/docs/version.json`
- Bu istek sonucunda sunucudan yalnızca `{ "minimumVersionCode": <pozitif tam sayı> }` bilgisini içeren JSON dosyası okunur ve uygulamanın yüklü `versionCode` değeriyle bellekte karşılaştırılır.
- Bu istek sırasında **hiçbir kullanıcı verisi, bildirim içeriği, OTP kodu, cihaz tanımlayıcısı veya analitik bilgisi sunucuya gönderilmez**; alınan JSON verisi DataStore'a yazılmaz.
- Sistem **fail-closed** çalışır: internet bağlantısı yoksa veya geçerli sürüm doğrulaması alınamazsa uygulama ana ekrana geçişi kilitler.

---

## 6. Üçüncü Taraf Kütüphaneler ve SDK'lar

`gradle/libs.versions.toml` ve `app/build.gradle.kts` incelemesine göre projede yalnızca Kotlin ve resmi Android Jetpack kütüphaneleri (`androidx.core:core-ktx`, `androidx.lifecycle:lifecycle-runtime-ktx`, `androidx.activity:activity-compose`, `androidx.compose.*`, `androidx.datastore:datastore-preferences`) yer almaktadır. **Üçüncü taraf reklam, analitik, telemetri veya hata izleme (crash reporting) SDK'sı bulunmamaktadır.**

---

## 7. Kullanıcının İzinleri Yönetmesi

Kullanıcı dilediği zaman **İzin Merkezi**, **Ayarlar** veya Android Sistem Ayarları üzerinden izinleri yönetebilir:
- **Bildirim Erişimi:** OTP algılamayı açar veya kapatır (zorunludur; öncesinde Prominent Disclosure gösterilir).
- **Bildirim Gösterme İzni (`POST_NOTIFICATIONS`, Android 13+):** VERIQ OTP bildirimlerinin gösterilmesini kontrol eder.
- **Pil ve Arka Plan Ayarları:** İsteğe bağlı sistem ayarıdır; kullanıcı ayarları değiştirmeye zorlanmaz.

---

## 8. Gizlilik Politikası Güncellemeleri ve İletişim

Uygulamanın veri işleme pratiklerinde değişiklik olduğunda bu belge güncellenir. Sorular ve talepler için:

- **Geliştirici / Şirket:** Fluid Galaxy Labs
- **Resmi İletişim ve Destek E-postası:** [fluidgalaxylabs@gmail.com](mailto:fluidgalaxylabs@gmail.com)
- **Resmi Gizlilik Politikası URL'si:** `[YAYIN ÖNCESİ DOLDURULACAK – Herkese açık HTTPS URL]`

---

# VERIQ – Privacy Policy (English Version)

## 1. Application and Developer Information

- **Application Name:** VERIQ
- **Application ID (`applicationId`):** `com.fluidgalaxy.codeflow`
- **Developer / Company:** Fluid Galaxy Labs
- **Official Contact and Support Email:** [fluidgalaxylabs@gmail.com](mailto:fluidgalaxylabs@gmail.com)
- **Version:** `1.0` (`versionCode: 1`)
- **Public Privacy Policy URL:** `[TO BE FILLED BEFORE PUBLISHING – Public HTTPS URL]`

---

## 2. Notification Access and OTP Detection

VERIQ accesses notifications authorized by the user through Android's system **Notification Access (`BIND_NOTIFICATION_LISTENER_SERVICE`)** via `OtpNotificationListenerService`. **Without Notification Access permission, the app's notification-based OTP detection cannot function.**

- **Supported Sources (`SourceRegistry`):** SMS/messaging apps (e.g., Google Messages, Samsung Messages), WhatsApp (`com.whatsapp`, `com.whatsapp.w4b`), and supported email apps (e.g., Gmail, Outlook, Samsung Email). Notifications from social media, games, and other unsupported apps are ignored without scanning.
- **Strictly On-Device Processing (`OtpParser`):** Notification titles (`EXTRA_TITLE`) and text fields (`EXTRA_TEXT`, `EXTRA_BIG_TEXT`) of supported apps are processed transiently in device memory (RAM) solely to detect verification codes (OTP).
- **Protection of Non-OTP Content:** Notification bodies, phone numbers, email addresses, or non-OTP personal messages are never saved to any database or persistent file, used for any secondary purpose, or sent to any external server.
- **Source Message Integrity:** VERIQ does **not delete, forward, or alter** the underlying messages in source apps (SMS, WhatsApp, Gmail, Outlook, etc.).
- **Optional Auto Mark as Read (`NotificationMarkAsReadHelper`):** The **"Otomatik Okundu İşaretle"** (`auto_mark_as_read`) preference in Settings is **disabled (`false`) by default**. Only when manually enabled by the user—and only after an OTP is detected and copied to the clipboard—does VERIQ inspect notification actions for an explicit, safe `"Okundu"` / `"Mark as read"` `PendingIntent` and trigger it once. Actions such as Reply, Archive, Delete, or Mark as unread are strictly rejected. Tracking keys are kept solely in a volatile in-memory set (`128` capacity) without storing notification bodies or OTP codes.
- **VERIQ Notification Actions (`CopyOtpReceiver`):** Within VERIQ's own OTP notification, **"Bildirimi Kapat"** closes only that VERIQ notification and **"Kopyala"** copies only that notification's OTP code again. Neither modifies source app notifications or OTP history.

---

## 3. Clipboard Usage

- **Writing to Clipboard (`OtpClipboardHelper`):** Detected OTP codes are written to the clipboard via `ClipboardManager.setPrimaryClip` when **Auto Copy** (`auto_copy`) is enabled or when the user taps **Kopyala** on the Home screen or VERIQ notification.
- **Auto Clipboard Clear and Protection:** When the configured timeout expires (`30s`, `1m`, `5m`; disabled when set to `Off`), VERIQ checks `ClipboardManager.primaryClip` strictly in memory for equality against the last copied OTP code (`currentClipboardText == code`) so user-copied content is not accidentally erased. If another value is present, VERIQ leaves the clipboard untouched; other clipboard contents are never inspected, stored, logged, or transmitted. If the same OTP code remains, the clipboard is cleared (`""`).
- **Android System Behavior:** Depending on the Android OS version (Android 10+ background clipboard limits and Android 12+ clipboard access system toasts), system restrictions are handled gracefully without crashing.

---

## 4. OTP History and Local Storage

VERIQ stores all persistent preferences and local records strictly on the user's device inside Android Jetpack Preferences DataStore (`otp_clipboard_settings.preferences_pb`):

- **OTP History (`otp_history_entries`):** Stores only the timestamp (`timestampMs`), source category (`SMS`, `WHATSAPP`, `EMAIL`), and status (`COPIED`, `AUTO_PASTED`, `DETECTED`), capped at `200` entries and pruned according to user retention settings (`1 Day`, `7 Days`, `30 Days`, or `Unlimited`). **Actual OTP codes and message bodies are never saved in OTP history.**
- **Last Detected OTP (`last_otp_code`, `last_otp_source`, `last_otp_timestamp`):** Stored locally in DataStore solely to display and re-copy the latest code on the Home card; cleared when tapping **Temizle**.
- **User Preferences & Counters:** `auto_copy`, `show_notification`, `vibration`, `auto_mark_as_read`, `clear_timeout`, `sms_enabled`, `whatsapp_enabled`, `email_enabled`, `daily_count`, `last_count_day`, `history_retention_days`, `has_seen_disclaimer`, and `has_accepted_notification_disclosure`.
- **No Server Transmission:** As verified in source code, none of the OTP history entries, settings, or local DataStore files are ever uploaded or sent to any server or third party.
- **Backup & Device Transfer Disabled:** `android:allowBackup="false"` alongside `backup_rules.xml` and `data_extraction_rules.xml` excludes all app data from Android Cloud Backup and Device-to-Device Transfer.
- **Data Deletion:** Users can clear the last code via Home > **Temizle**, delete all OTP history via History > **Tüm Geçmişi Temizle**, or permanently erase all local DataStore data via Android App Settings ("Clear storage") or by uninstalling VERIQ.

---

## 5. Remote Minimum Version Check via GitHub

VERIQ declares `android.permission.INTERNET` and `android.permission.ACCESS_NETWORK_STATE` in `AndroidManifest.xml` solely for fail-closed remote minimum version verification (`AppVersionChecker.kt`):

- At startup and whenever returning to the foreground (`ON_RESUME`), VERIQ checks network availability via `ConnectivityManager` and performs an uncached **HTTPS GET** request to:
  - `https://raw.githubusercontent.com/fluidgalaxylabs-dotcom/fluid-galaxy-veriq/main/docs/version.json`
- Only `{ "minimumVersionCode": <positive integer> }` is read and compared in memory against the installed app `versionCode`.
- **No personal data, notification text, OTP code, analytics event, or device identifier is sent in this request**, and the response is not persisted to DataStore.

---

## 6. Third-Party Libraries and SDKs

Per `gradle/libs.versions.toml` and `app/build.gradle.kts`, VERIQ uses only Kotlin and official Android Jetpack / Material3 / Compose / DataStore libraries. **No third-party advertising, analytics, telemetry, or crash-reporting SDK is included.**

---

## 7. Contact Information

For questions or requests regarding this Privacy Policy or local data handling in VERIQ:

- **Developer / Company:** Fluid Galaxy Labs
- **Official Contact and Support Email:** [fluidgalaxylabs@gmail.com](mailto:fluidgalaxylabs@gmail.com)
- **Public Privacy Policy URL:** `[TO BE FILLED BEFORE PUBLISHING – Public HTTPS URL]`
