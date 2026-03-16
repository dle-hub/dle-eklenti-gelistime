# DataLife Engine (DLE) 19.x+ Geliştirici Master Rehberi (A'dan Z'ye Tam Kapsam)

Bu rehber, DLE 19.1+ sürümleri için profesyonel eklenti geliştirme standartlarını, çekirdek kod mantığını ve native UI/UX kurallarını içeren **en kapsamlı** teknik dökümandır. Tüm bileşenler, modül kayıt sistemleri, frontend entegrasyonları ve **eksiksiz API kataloğu** bir araya getirilmiştir.

---

## 🏗️ 1. Mimari ve Dosya Yapısı (VFS)

DLE, **Virtual File System (VFS)** mimarisini kullanır. Eklentiler çekirdek dosyalara fiziksel olarak dokunmaz; `plugin.xml` üzerinden sanal müdahaleler yapar.

### Standart Klasör Organizasyonu
```text
Eklenti_Paketi/
├── plugin.xml           # Eklentinin beyni: Kurulum, SQL, Dosya Müdahaleleri
└── upload/              # Sunucu kök dizinine atılacak dosyalar
    ├── engine/
    │   ├── inc/         # Yönetim Paneli (Admin) Modülleri
    │   ├── modules/     # Site Ön Yüz (Frontend) Mantığı
    │   ├── ajax/        # Arka Plan (Controller) İşleyicileri
    │   ├── api/         # Dahili API Sınıfları
    │   └── data/        # Ayar dosyaları (dbconfig.php vb.)
    ├── public/          # [v19+] Dışa açık JS, CSS ve Görseller
    └── templates/       # .tpl Şablon Dosyaları
```

---

## 🎨 2. Yönetim Paneli (Admin Panel - engine/inc)

DLE admin modülleri için `/engine/inc/` dizini zorunludur.

### A. Admin Modül Erişimi ve Kayıt Sistemi
1.  **Dosya Konumu:** Modül dosyası mutlaka `/engine/inc/` klasöründe olmalıdır.
2.  **Erişim:** Dosya adınız `mymod.php` ise, modüle `https://site.com/admin.php?mod=mymod` adresinden otomatik olarak erişilir.
3.  **Menü Kaydı (PREFIX_admin_sections):** Modülün panel listesinde görünmesi için veritabanındaki `PREFIX_admin_sections` tablosuna kayıt edilmelidir.

**Tablo Parametreleri:**
*   **name:** Dosya adı (uzantısız, örn: `mymod`).
*   **title:** Modülün paneldeki başlığı.
*   **descr:** Modülün kısa açıklaması.
*   **icon:** İkon ismi ve yolu. Önerilen boyut **70x70** pikseldir.
*   **allow_groups:** Erişebilecek gruplar (`all` veya `1,2,3`).

**Örnek Kayıt Sorgusu:**
```sql
INSERT INTO `{prefix}_admin_sections` (`name`, `title`, `descr`, `icon`, `allow_groups`) 
VALUES ('mymod', 'Test Modülü', 'Modül açıklaması...', 'mymod.png', '1');
```

### B. Ar some UI/UX (navbar-collapse collapse)
Mobilde hamburger menüye dönüşen, en profesyonel native navbar yapısı:
```html
<div class="navbar navbar-default navbar-component navbar-xs systemsettings">
    <ul class="nav navbar-nav visible-xs-block">
        <li class="full-width text-center">
            <a data-toggle="collapse" data-target="#navbar-nav-id"><i class="fa fa-bars"></i></a>
        </li>
    </ul>
    <div class="navbar-collapse collapse" id="navbar-nav-id">
        <ul class="nav navbar-nav">
            <li class="active font-bold"><a href="#"><i class="fa fa-cog position-left"></i> Ayarlar</a></li>
        </ul>
        <ul class="nav navbar-nav navbar-right">
             <li><a href="#" class="bg-primary-600 text-white"><i class="fa fa-plus"></i> Yeni Ekle</a></li>
        </ul>
    </div>
</div>
```

---

## 🎨 3. Yönetim Paneli (Admin UI) Standartları

DLE admin paneli, tutarlı bir kullanıcı deneyimi için belirli tasarım kalıpları (Design Patterns) üzerine inşa edilmiştir.

### A. Layout Hiyerarşisi
Sayfalarınızı şu silsile ile yapılandırın:
1.  **Üst Bilgi:** `echoheader()` (İkon + Başlık + Alt Başlık).
2.  **Navigasyon:** `navbar-collapse` (Filtreler ve Sekmeler).
3.  **İçerik:** `alert` (Bilgi/Uyarı mesajları) -> `panel` (Ana içerik alanı).
4.  **Alt Bilgi:** `panel-footer` (Kaydet/İptal butonları) -> `echofooter()`.

### B. Semantik Renk Paleti
DLE'de renkler işlevsel anlamlar taşır. `bg-{color}` veya `label-{color}` sınıflarıyla kullanılır:
*   **Primary (Mavi):** `bg-primary-600` - Ana eylemler ve ayarlar.
*   **Success (Yeşil/Teal):** `bg-teal` - Kaydetme, ekleme, "aktif" durumu.
*   **Danger (Kırmızı):** `bg-danger-600` - Silme, kritik hatalar, "pasif" durumu.
*   **Info (Koyu Mavi):** `bg-info-800` veya `label-info` - Bilgilendirme ve ipuçları.
*   **Warning (Sarı/Turuncu):** `alert-warning` - Dikkat gerektiren durumlar.

### C. Tipografi ve Spacing (Boşluk) Kuralları
*   **Vurgu:** `text-semibold`, `text-bold`, `text-size-small`.
*   **Yardımcı:** `text-muted` (Silik gri metin).
*   **İkonlar:** `position-left` (Metnin solundaki ikon için).
*   **Dikey Boşluk:** `mt-10`, `mt-15`, `mt-20` sınıflarını form ve butonlar arasında kullanın.

### D. Gelişmiş Komponentler
**1. Native Alert Kutuları:**
`alert-styled-left` ve `alert-arrow-left` kullanarak estetik uyum sağlayın:
```html
<div class="alert alert-info alert-styled-left alert-arrow-left alert-component">
    <b>Bilgi:</b> Bu modül sadece yöneticiler içindir.
</div>
```

**2. Status Labels:** `span.label.label-success`, `label-danger`, `label-info`.

---

## 🌐 4. Site Ön Yüz (Frontend - engine/modules)

### A. Modül Dahil Etme ({include ...})
```smarty
{include file="engine/modules/mymod.php"}
```
**Kritik Not:** Modülün bulunduğu klasörün yazma izni (**CHMOD 777**) **olmamalıdır**. DLE güvenlik nedeniyle bu tip klasörlerden PHP çalıştırmaz.

### B. Parametre Gönderimi ve Dinamik Etiketler
Modülünüze GET parametresi gibi veri gönderebilirsiniz:
```smarty
{include file="engine/modules/mymod.php?id=123&news_id={news-id}"}
```
*Parametreler URL limitiyle sınırlıdır, devasa veriler (tüm haber içeriği vb.) bu yöntemle gönderilemez.*

### C. Özel Bölüm Oluşturma (Main Block Değişimi)
`main.tpl`'de haberler yerine kendi modül modülünüzün çıktısını basmak:
```smarty
[aviable=faq] {include file="engine/modules/faq.php"} [/aviable]
[not-aviable=faq] {content} [/not-aviable]
```

---

## 🔌 5. DLE API Sistemi (engine/api/api.class.php)

DLE API, hem eski hem de gelecekteki sürümlerle %100 uyumlu kod yazmanızı sağlar. Veritabanı bağlantısı veya sınıfları manuel çağırmanıza gerek kalmaz.

### API'yi Dahil Etme
```php
include ('engine/api/api.class.php');
```

### A. Kullanıcı Yönetimi Fonksiyonları
*   `$dle_api->take_user_by_id( int $id [, string $select_list] );` - ID ile kullanıcı verisi çeker.
*   `$dle_api->take_user_by_name( string $name [, string $select_list] );` - Kullanıcı adı ile veri çeker.
*   `$dle_api->take_user_by_email( string $email [, string $select_list] );` - E-posta ile veri çeker.
*   `$dle_api->take_users_by_group( int $group_id [, string $select_list [, int $limit]] );` - Gruba göre kullanıcı çeker.
*   `$dle_api->take_users_by_ip( string $ip [, bool $like [, string $select_list [, int $limit]]] );` - IP ile kullanıcı çeker.
*   `$dle_api->change_user_name( int $user_id, string $new_name );` - Kullanıcı adını değiştirir.
*   `$dle_api->change_user_pass( int $user_id, string $new_pass );` - Şifre değiştirir.
*   `$dle_api->change_user_email( int $user_id, string $new_email );` - E-posta değiştirir.
*   `$dle_api->change_user_group( int $user_id, int $new_group );` - Kullanıcı grubunu değiştirir.
*   `$dle_api->external_auth( string $login, string $password );` - Giriş doğrulaması yapar.
*   `$dle_api->external_register( string $login, string $password, string $email, int $group );` - Yeni kullanıcı kaydeder.

### B. İçerik ve İletişim Fonksiyonları
*   `$dle_api->send_pm_to_user ( int $user_id, string $subject, string $text, string $from );` - Özel mesaj gönderir.
*   `$dle_api->take_news ( string $cat [, string $fields [, int $start [, int $limit [, string $sort [, string $sort_order]]]]] );` - Haberleri çeker.

### C. Veritabanı ve Önbellek (Cache) Fonksiyonları
*   `$dle_api->load_table ( string $table [, string $fields [, string $where [, bool $multirow ...]]] );` - Tablodan veri çeker.
*   `$dle_api->save_to_cache ( string $fname, mixed $vars );` - Veriyi önbelleğe yazar.
*   `$dle_api->load_from_cache ( string $fname [, int $timeout [, string $type]] );` - Önbellekten veri okur.
*   `$dle_api->clean_cache ( [string $name] );` - Önbelleği temizler.
*   `$dle_api->get_cached_files();` - Önbellek dosya listesini döndürür.

### D. Sistem ve Ayarlar
*   `$dle_api->edit_config ( mixed $key [, string $new_value] );` - Sistem ayarlarını günceller ve kaydeder.
*   `$dle_api->checkGroup ( int $group );` - Grubun varlığını kontrol eder.
*   `$dle_api->install_admin_module ( string $name, string $title, string $descr, string $icon [, string $perm] );` - Admin modülü yükler.
*   `$dle_api->uninstall_admin_module ( string $name );` - Admin modülü siler.
*   `$dle_api->change_admin_module_perms ( string $name, string $perm );` - Modül erişim yetkilerini değiştirir.

---

## 📦 6. Modüllerde Kullanılabilen Global Değişkenler

Dahil edilen (include) PHP dosyalarında hiçbir tanımlama yapmadan şu değişkenlere erişebilirsiniz:

| Değişken | Açıklama |
| :--- | :--- |
| `$db` | DLE Veritabanı sınıfı (`$db->super_query`, `$db->safesql`) |
| `$is_logged` | Ziyaretçi üye mi yoksa misafir mi? (true/false) |
| `$member_id` | Üyenin tüm profil verilerini içeren dizi (Array) |
| `$tpl` | DLE Şablon motoru sınıfı |
| `$cat_info` | Tüm kategorilerin bilgilerini içeren dizi |
| `$config` | Tüm sistem ayarları (site_url, version vb.) |
| `$user_group` | Tüm kullanıcı grupları ve ayarları |
| `$category_id` | Ziyaretçinin o an görüntülediği kategori ID'si |
| `$_TIME` | Sistemin güncel UNIX zamanı (offset dahil) |
| `$lang` | Dil paketindeki metinleri içeren dizi |
| `$smartphone_detected` | Mobil cihaz tespiti durumu (true/false) |
| `$dle_module` | Mevcut bölüm ismi (URL'deki `do` parametresi) |

---

## ⚡ 7. DLEPush & AJAX & Bildirimler (Native JS API)

### A. DLEPush (Toast Bildirim Sistemi)
`DLEPush.tur(Mesaj, Başlık, Süre);`
*   `DLEPush.success('İşlem tamam!');`
*   `DLEPush.error('Hata oluştu!', 'Sistem', 5000);`

### B. AJAX ve Spinner (ShowLoading)
AJAX isteklerinde ekranı kilitli tutmak DLE standartıdır:
```javascript
ShowLoading(''); // Spinner Başlat
$.post("index.php?controller=ajax&mod=test", { user_hash: dle_login_hash }, function(res) {
    HideLoading(''); // Spinner Durdur
    DLEPush.success(res.message);
}, 'json');
```

---

## 📜 8. plugin.xml ve Güvenlik Kuralları

1.  **Direct Access Block:** `if( !defined( 'DATALIFEENGINE' ) ) die( "Hacking attempt!" );`
2.  **VFS Desteği (DLEPlugins::Check):** Dosya dahil ederken **ZORUNLUDUR**:
    `include (DLEPlugins::Check(ENGINE_DIR . '/modules/script.php'));`
3.  **SQL Protection:** Sorgularda tabloları `{prefix}_users` şeklinde yazın ve her zaman `$db->safesql()` kullanın.

---
🚀 Bu dev döküman, DLE dünyasındaki tüm teknik ve API detaylarını kapsayan en güncel rehberdir.
