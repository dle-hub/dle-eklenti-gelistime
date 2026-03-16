# 📘 DataLife Engine (DLE) 19.x+ Eklenti Geliştirme Ansiklopedisi (Nihai Rehber)

Bu döküman, DLE 19.1 ve üzeri sürümlerde profesyonel, güvenli ve sistemle %100 uyumlu (Native) eklentiler geliştirmek için gereken **TÜM** teknik detayları, kod iskeletlerini ve resmi API referanslarını içeren tek dev kaynaktır.

---

## 🏗️ 1. MİMARİ VE DOSYA YAPISI (VFS)

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

### Kritik Güvenlik Satırları
Tüm PHP dosyalarınızın en başına şu satırları eklemek **ZORUNLUDUR**:
```php
if( !defined( 'DATALIFEENGINE' ) ) die( "Hacking attempt!" );

// Sadece Admin Panel dosyaları için ek olarak:
if( !defined( 'LOGGED_IN' ) ) die( "Hacking attempt!" );
```

---

## 📜 2. PLUGIN.XML VE SANAL DOSYA SİSTEMİ

Eklentinin kurulumu ve dosya müdahaleleri burada tanımlanır.

*   **SQL Operasyonları:** Kurulum (`<mysqlinstall>`) ve silme (`<mysqldelete>`) işlemlerinde `{prefix}` ve `{charset}` etiketleri kullanılmalıdır.
*   **Dosya Müdahaleleri:** Kod enjeksiyonu için `after`, `before` veya `replace` eylemleri kullanılır.
*   **Yeni Dosya Oluşturma:** DLE'de olmayan bir dosyayı (Örn: Modül linki) sıfırdan yaratmak için `<operation action="create">` kullanılır.

---

## 🎨 3. YÖNETİM PANELİ (ADMIN PANEL) GELİŞTİRME

Admin modülleri mutlaka `/engine/inc/` dizininde olmalıdır.

### A. Modül Kaydı ve Erişim (PREFIX_admin_sections)
Modülünüzün "Diğer Bölümler" listesinde görünmesi için veritabanına kayıt edilmelidir.

**Tablo Parametreleri:**
*   **name:** Dosya adı (uzantısız, örn: `mymod`).
*   **title:** Modülün paneldeki başlığı.
*   **descr:** Modülün kısa açıklaması.
*   **icon:** İkon ismi. Önerilen boyut **70x70** pikseldir.
*   **allow_groups:** Erişebilecek gruplar (`all` veya `1,2,3`).

**Örnek Kayıt SQL:**
```sql
INSERT INTO `{prefix}_admin_sections` (`name`, `title`, `descr`, `icon`, `allow_groups`) 
VALUES ('mymod', 'Modül Başlığı', 'Modül açıklaması...', 'mymod.png', '1');
```

### B. Dinamik Navigasyon (navbar-collapse collapse)
Mobilde hamburger menüye dönüşen, filtreleme ve araç çubuğu yapısı:
```html
<div class="navbar navbar-default navbar-component navbar-xs systemsettings">
    <ul class="nav navbar-nav visible-xs-block">
        <li class="full-width text-center">
            <a data-toggle="collapse" data-target="#navbar-filter"><i class="fa fa-bars"></i></a>
        </li>
    </ul>
    <div class="navbar-collapse collapse" id="navbar-filter">
        <ul class="nav navbar-nav">
            <li class="active font-bold"><a href="#"><i class="fa fa-cog position-left"></i> Ayarlar</a></li>
            <li><a href="#"><i class="fa fa-list position-left"></i> Kayıtlar</a></li>
        </ul>
        <ul class="nav navbar-nav navbar-right">
            <li>
                <div class="navbar-form">
                    <input type="text" name="search" class="form-control" placeholder="Hızlı Ara...">
                </div>
            </li>
            <li><a href="#" class="bg-primary-600 text-white"><i class="fa fa-plus"></i> Yeni Ekle</a></li>
        </ul>
    </div>
</div>
```

### C. UI Bileşenleri ve Form Standartları
*   **Paneller:** `panel panel-default` (İçerik) veya `panel-flat` (Ayarlar tablosu).
*   **Sekmeler:** `nav nav-tabs nav-tabs-solid` sınıfı ile panel üstüne yerleştirilir.
*   **Form Düzeni:** `form-horizontal` kullanarak Label için `col-md-2`, Input için `col-md-10`.
*   **Kontroller:** `icheck` (kutucuk) ve `switch` (toggled switch).
*   **Yardım Butonu:** `i.help-button` ve `data-rel="popover"` ile native tooltip.
*   **Alert Kutuları:** `alert alert-info alert-styled-left alert-arrow-left alert-component`.

---

## 🌐 4. SİTE ÖN YÜZ (FRONTEND) GELİŞTİRME

### A. Modül Dahil Etme ({include ...})
Şablon dosyalarına (`.tpl`) PHP dosyalarınızı dahil ederken parametre geçebilirsiniz:
```smarty
{include file="engine/modules/mymod.php?param1=value1&news_id={news-id}"}
```
**Güvenlik Uyarısı:** Modül klasöründe yazma izni (**CHMOD 777**) varsa DLE güvenlik nedeniyle dosyayı çalıştırmaz.

### B. Özel Bölüm Oluşturma (Aviable)
Haber bloğu yerine kendi modülünüzü basmak:
```smarty
[aviable=faq] {include file="engine/modules/faq.php"} [/aviable]
[not-aviable=faq] {content} [/not-aviable]
```

---

## ⚡ 5. JAVASCRIPT, AJAX VE BİLDİRİMLER

### A. DLEPush (Toast Bildirim Sistemi)
```javascript
DLEPush.success('Mesaj', 'Başlık'); // Yeşil
DLEPush.info('Mesaj', 'Başlık');    // Mavi
DLEPush.warning('Mesaj', 'Başlık'); // Sarı
DLEPush.error('Mesaj', 'Başlık', 5000); // Kırmızı (5 sn)
```

### B. AJAX İskeleti ve Spinner
```javascript
function saveAction() {
    tinyMCE.triggerSave(); // Varsa editörü senkronize et
    ShowLoading('');       // Ekranı kilitle
    
    $.post("index.php?controller=ajax&mod=myplugin", { 
        data: 'val', 
        user_hash: dle_login_hash // CSRF Koruması ZORUNLU
    }, function(res) {
        HideLoading('');   // Kilidi kaldır
        if (res.success) DLEPush.success(res.message);
        else DLEPush.error(res.error);
    }, 'json');
}
```

### C. TinyMCE ve Onay Pencereleri
*   `tinymce.get('id').setContent(html)` / `getContent()`
*   `DLEconfirm('Emin misiniz?', 'Başlık', function() { ... });`
*   `DLEconfirmDelete('Silinecek!', 'Onay', function() { ... });`

---

## 📦 6. ÇEKİRDEK SINIFLAR VE GLOBALLER

Dahil edilen PHP dosyalarında şu değişkenler hazır gelir:
*   **$db:** Veritabanı sınıfı (`$db->super_query`, `$db->safesql`).
*   **$tpl:** Şablon motoru sınıfı.
*   **$member_id:** Giriş yapan kullanıcının tüm profil verileri (Array).
*   **$config:** Tüm sistem ayarları.
*   **$is_logged:** Kullanıcı girişi (true/false).
*   **$lang:** Dil paketi dizisi.
*   **$cat_info:** Kategori hiyerarşisi.
*   **$category_id:** Mevcut kategori ID'si.
*   **$_TIME:** Güncel UNIX zamanı.
*   **$smartphone_detected:** Mobil cihaz tespiti (true/false).
*   **$dle_module:** Mevcut sayfa ismi.

---

## 🔌 7. RESMİ DLE API ($dle_api)

API kullanımı, sürüm güncellemelerinden etkilenmemek için tavsiye edilir.
`include ('engine/api/api.class.php');`

**Kritik API Fonksiyonları:**
*   `take_user_by_id(id, fields)` / `take_user_by_name` / `take_user_by_email`
*   `take_users_by_group(id, fields, limit)` / `take_users_by_ip`
*   `change_user_name` / `change_user_pass` / `change_user_email` / `change_user_group`
*   `external_auth(login, pass)` / `external_register(login, pass, email, group)`
*   `send_pm_to_user(id, subject, text, from)`
*   `load_table(table, fields, where, multirow, start, limit, sort, sort_order)`
*   `save_to_cache(name, vars)` / `load_from_cache` / `clean_cache` / `get_cached_files`
*   `edit_config(key, value)`
*   `take_news(cat, fields, start, limit, sort, sort_order)`
*   `checkGroup(id)`
*   `install_admin_module(name, title, descr, icon, perm)` / `uninstall_admin_module` / `change_admin_module_perms`

---

## 🛡️ 8. GÜVENLİK VE KODLAMA STANDARTLARI

1.  **VFS Desteği (EN ÖNEMLİ):** Dosya dahil ederken her zaman:
    `include (DLEPlugins::Check(ENGINE_DIR . '/modules/myscript.php'));`
2.  **SQL Protection:** Kullanıcı verilerini `$db->safesql()` süzgecinden geçirin.
3.  **CHMOD:** Yazma izni olan klasörlerde asla modül dosyası bulundurmayın.
4.  **Hacking Attempt:** Dosya başlarındaki `DATALIFEENGINE` sabit kontrollerini unutmayın.

---
🚀 Bu rehber DLE 19.x mimarisindeki her detayı kapsayan **eksiksiz** teknik iskelettir.
