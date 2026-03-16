# DataLife Engine (DLE) 19.1+ Geliştirici Master Rehberi (A'dan Z'ye)

Bu döküman, DLE 19.1 ve üzeri sürümlerde modern, güvenli ve tüm sistemle (Frontend, Admin, API) %100 uyumlu eklentiler geliştirmek isteyenler için hazırlanmış kapsamlı bir teknik kaynaktır.

---

## 🏗️ 1. Sistem Mimarisi ve Dosya Yapısı

DLE bir **VFS (Virtual File System)** yapısı kullanır. Çekirdek dosyalar fiziksel olarak değişmez; eklenti sistemi (`plugin.xml`) bu dosyaların sanal kopyalarını oluşturur ve müdahaleleri yönetir.

### Standart Klasör Düzeni
```text
Eklenti_Klasoru/
├── plugin.xml           # Eklentinin beyni: Kurulum, SQL, Dosya Müdahaleleri
└── upload/              # Sunucu kök dizinine atılacak dosyalar
    ├── engine/
    │   ├── inc/         # Yönetim Paneli (Admin) Modülleri
    │   ├── modules/     # Site Ön Yüz (Frontend) Mantığı
    │   ├── ajax/        # Arka Plan (AJAX) İşleyicileri
    │   └── api/         # [Dahili] DLE API Sınıfları
    ├── public/          # [v19+] JS, CSS, Resimler (Dışa Açık)
    └── templates/       # .tpl Şablon Dosyaları
```

---

## 🌐 2. Site Ön Yüz (Frontend - engine/modules)

Site ön yüzünde modülünüzü şablonlara (`.tpl`) dahil etmek için gelişmiş `{include ...}` etiketi kullanılır.

### A. Şablona Dahil Etme ve Parametre Gönderimi
```smarty
{* Temel dahil etme *}
{include file="engine/modules/mymod.php"}

{* Parametre gönderimi (Değişkenler modül içinde $param1 ve $param2 olarak hazır gelir) *}
{include file="engine/modules/mymod.php?param1=deger&param2=deger2"}

{* Dinamik etiket gönderimi (Sadece main.tpl dışındaki şablonlarda) *}
{include file="engine/modules/mymod.php?news_id={news-id}"}
```
> **Önemli:** Dahil edilen dosyanın bulunduğu klasörün yazma izni (CHMOD 777) **olmamalıdır**. DLE güvenlik gereği bu tip klasörlerden PHP çalıştırmaz.

### B. Özel Sayfa/Bölüm Oluşturma
Haber alanı yerine kendi modülümüzün çıktısını basmak (Örn: FAQ sayfası):
```smarty
[aviable=faq]
    {include file="engine/modules/faq.php"}
[/aviable]
[not-aviable=faq]
    {content}
[/not-aviable]
```
Bu sayfa `site.com/index.php?do=faq` adresinde çalışacaktır.

---

## 🛠️ 3. Yönetim Paneli (Admin Panel - engine/inc)

Admin modülleri `/engine/inc/` klasörüne atılır ve otomatik olarak `admin.php?mod=modul_adi` şeklinde erişilebilir.

### A. Modülü Panelde Kayıt Etme (Menu Entegrasyonu)
Modülünüzün "Diğer Bölümler" listesinde görünmesi için `PREFIX_admin_sections` tablosuna kayıt edilmelidir:
```sql
INSERT INTO `{prefix}_admin_sections` (`name`, `title`, `descr`, `icon`, `allow_groups`) 
VALUES ('mymod', 'Modül Başlığı', 'Modül açıklaması...', 'icon_yolu.png', '1');
```
*   **name:** Dosya ismi (uzantısız).
*   **allow_groups:** Erişebilecek gruplar (Örn: `1,2,3` veya `all`).

---

## 📦 4. DLE Global Değişkenler Listesi

Dahil ettiğiniz modüllerde herhangi bir tanımlama yapmadan şu global değişkenlere erişebilirsiniz:

| Değişken | Açıklama |
| :--- | :--- |
| `$is_logged` | Kullanıcı giriş yapmış mı? (true/false) |
| `$member_id` | Giriş yapan kullanıcının profil verileri (Array) |
| `$db` | DLE Veritabanı sınıfı |
| `$tpl` | DLE Şablon (Template) sınıfı |
| `$config` | Tüm sistem ayarları (Array) |
| `$user_group` | Kullanıcı grupları ve yetkileri (Array) |
| `$cat_info` | Kategori bilgileri ve hiyerarşisi |
| `$lang` | Dil paketi metinleri |
| `$dle_module` | Mevcut sayfa/bölüm ismi (`do` parametresi) |
| `$_TIME` | Sistemin güncel UNIX zamanı (offset dahil) |

---

## 🔌 5. DLE API Sistemi (engine/api)

Sürüm güncellemelerinden etkilenmemek ve standart bir kod yapısı için DLE API kullanılması şiddetle tavsiye edilir.

### API'yi Dahil Etme
```php
include ('engine/api/api.class.php');
```

### Öne Çıkan API Fonksiyonları
*   **Kullanıcı Verisi:** `$dle_api->take_user_by_id( $id, $fields );`
*   **Kullanıcı Güncelleme:** `$dle_api->change_user_group( $id, $new_group );`
*   **Özel Mesaj (PM):** `$dle_api->send_pm_to_user( $target_id, $subject, $text, $from_login );`
*   **Haber Çekme:** `$dle_api->take_news( $cat_ids, $fields, $start, $limit );`
*   **Tablo İşlemleri:** `$dle_api->load_table( 'table', 'fields', 'where', $multirow );`
*   **Önbellek (Cache):** `$dle_api->save_to_cache( $name, $data );`

---

## ⚡ 6. JavaScript, AJAX & Bildirimler (Native API)

### A. Bildirimler ve Onaylar
*   **DLEPush:** `DLEPush.success('Mesaj');` (success, info, warning, error)
*   **Show/Hide Loading:** AJAX işlemi sırasında ekranı kilitlemek için `ShowLoading('');` ve `HideLoading('');`
*   **DLEconfirm:** `DLEconfirm('Soru?', 'Başlık', function() { ... });`

### B. TinyMCE Senkronizasyonu
Editördeki veriyi okumadan önce mutlaka:
```javascript
tinyMCE.triggerSave();
var icerik = $('#short_story').val();
```

---

## 📜 7. plugin.xml ve Sanal Dosya Yönetimi

*   **create action:** Olmayan bir dosyayı (Örn: admin modülü linki) sıfırdan oluşturmak için kullanılır.
*   **SQL Sorguları:** `{prefix}` ve `{charset}` etiketlerini kullanarak veritabanı uyumluluğunu koruyun.
*   **DLEPlugins::Check:** Dosya çağırırken (`include`) her zaman bu fonksiyonu kullanın.

---

## 🛡️ 8. Güvenlik ve Performans Kuralları

1.  **Direct Access:** PHP dosyalarının başına `if( !defined( 'DATALIFEENGINE' ) ) die( "Hacking attempt!" );` ekleyin.
2.  **Relative Paths:** Modül yollarında mutlaka göreceli (relative) yollar kullanın.
3.  **CHMOD:** Yazma izni olan (777) klasörlerde PHP dosyası bulundurmayın.
4.  **SQL Protection:** Kullanıcı verilerini her zaman `$db->safesql()` süzgecinden geçirin.

---
Bu rehber, DLE topluluğu için modern standartları ve çekirdek dökümantasyonu baz alarak hazırlanmıştır. 🚀
