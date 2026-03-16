# DataLife Engine (DLE) 19.x+ Geliştirici Master Rehberi (Tam Kapsamlı)

Bu döküman, DLE 19.1 ve üzeri sürümlerde çekirdek sistemle %100 uyumlu, profesyonel ve modern eklentiler geliştirmek için gereken tüm teknik detayları, kod örneklerini ve tasarım kurallarını tek bir kaynakta toplar.

---

## 🏗️ 1. Mimari ve Dosya Yapısı (VFS)

DLE, **Virtual File System (VFS)** mimarisini kullanır. Eklenti sistemi (`plugin.xml`), çekirdek dosyaların sanal kopyalarını oluşturarak müdahale eder. Fiziksel dosyalara asla dokunulmaz.

### Klasör Organizasyonu
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

Admin paneli **Bootstrap 3** tabanlıdır ve DLE'nin kendine has "Native" tasarım dilini kullanır.

### A. Giriş ve Güvenlik
Her `.php` dosyası şu korumayla başlamalıdır:
```php
if( !defined( 'DATALIFEENGINE' ) OR !defined( 'LOGGED_IN' ) ) die( "Hacking attempt!" );

// Sayfa Başlığı ve İkon (Native DLE)
echoheader( "<i class=\"fa fa-cubes position-left\"></i><span class=\"text-semibold\">Modül Başlığı</span>", "Açıklama Metni" );

// ... İçerik ...

echofooter();
```

### B. Dinamik Navigasyon (navbar-collapse collapse)
Mobilde "Hamburger" menüye dönüşen, filtreleme ve arama araçları için kullanılan yapıdır.
```html
<div class="navbar navbar-default navbar-component navbar-xs systemsettings">
    <ul class="nav navbar-nav visible-xs-block">
        <li class="full-width text-center">
            <a data-toggle="collapse" data-target="#navbar-filter"><i class="fa fa-bars"></i></a>
        </li>
    </ul>
    <div class="navbar-collapse collapse" id="navbar-filter">
        <ul class="nav navbar-nav">
            <li class="active font-bold"><a href="#"><i class="fa fa-cog"></i> Ayarlar</a></li>
            <li><a href="#"><i class="fa fa-list"></i> Kayıtlar</a></li>
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

### C. Panel, Sekme ve Liste Yapısı
*   **Sekmeler (Tabs):** `nav nav-tabs nav-tabs-solid` sınıfı ile panel üstüne yerleştirilir.
*   **Media Listesi:** İkonlu ve açıklamalı linkler için:
    ```html
    <div class="panel-body list-bordered">
        <div class="row box-section">
            <div class="col-sm-6 media-list media-list-linked">
                <a class="media-link" href="#">
                    <div class="media-left"><img src="public/adminpanel/images/tools.png" class="img-lg"></div>
                    <div class="media-body">
                        <h6 class="media-heading text-semibold">Yapılandırma</h6>
                        <span class="text-muted">Açıklama buraya...</span>
                    </div>
                </a>
            </div>
        </div>
    </div>
    ```

### D. Form Standartları (Native Hiyerarşi)
*   **Düzen:** Her zaman `form-horizontal` kullanılmalı.
*   **Oranlar:** Label `col-md-2`, Input kapsayıcı `col-md-10`.
*   **Yardım Butonu:** `help-button` sınıfı ve `data-rel="popover"` ile native tooltip eklenir.
*   **Kontroller:** `icheck` (kutucuk) ve `switch` (toggled switch) sınıfları.

---

## 🌐 3. Site Ön Yüz (Frontend - engine/modules)

### A. Şablona Dahil Etme ve Parametreler
```smarty
{* Modül çağırma *}
{include file="engine/modules/mymod.php"}

{* Parametre gönderimi (Modül içinde $id ve $mode olarak hazır gelir) *}
{include file="engine/modules/mymod.php?id=123&mode=dark"}

{* Dinamik etiket gönderimi *}
{include file="engine/modules/mymod.php?news_id={news-id}"}
```

### B. Bölüm Kontrolü (Aviable)
Haber bloğu yerine kendi modülünüzü basmak için `main.tpl`'de:
```smarty
[aviable=mymod] {include file="engine/modules/mymod.php"} [/aviable]
[not-aviable=mymod] {content} [/not-aviable]
```

### C. PHP Şablon Motoru ($tpl)
```php
$tpl->load_template( 'custom.tpl' );
$tpl->set( '{tag}', "Veri" );
$tpl->set_block( "'\\[block\\](.*?)\\[/block\\]'si", "\\1" );
$tpl->compile( 'content' );
$tpl->clear();
```

---

## ⚡ 4. AJAX ve Bildirim Sistemi (engine/ajax)

DLE 19 AJAX isteklerini merkezi `controller.php` üzerinden yönetir.
**Yol:** `index.php?controller=ajax&mod=modul_adi`

### A. Javascript Tarafı (JS API)
```javascript
function saveAction() {
    tinyMCE.triggerSave(); // Varsa editörü senkronize et (ZORUNLU)
    ShowLoading(''); // Ekranı kilitle & Spinner çıkar
    
    $.post("index.php?controller=ajax&mod=myplugin", { 
        data: 'val', 
        user_hash: dle_login_hash // Güvenlik Hash'i ZORUNLUDUR
    }, function(res) {
        HideLoading(''); // Kilidi aç
        
        if (res.success) {
            DLEPush.success(res.message); // Native Yeşil Bildirim
        } else {
            DLEPush.warning(res.error); // Native Sarı Bildirim
        }
    }, 'json').fail(function(xhr) {
        HideLoading('');
        DLEPush.error('Sistem Hatası: ' + xhr.status); // Native Kırmızı Bildirim
    });
}
```

### B. Onay Pencereleri (Confirm)
*   `DLEconfirm('Devam edilsin mi?', 'Başlık', function() { ... });`
*   `DLEconfirmDelete('Silinecek emin misiniz?', 'Bilgi', function() { ... });`

---

## 📦 5. DLE Global Değişkenleri ve API

Eklentilerde aşağıdaki global verilere doğrudan erişilebilir:

| Değişken | Açıklama |
| :--- | :--- |
| `$db` | Veritabanı sınıfı (`super_query`, `safesql`, `query`) |
| `$config` | Tüm sistem ayarları (`$config['site_url']` vb.) |
| `$member_id` | Oturum açan kullanıcının profil verileri (Array) |
| `$user_group` | Grupların yetki ve ayar haritası |
| `$is_logged` | Kullanıcı girişi kontrolü (true/false) |
| `$tpl` | Şablon motoru sınıfı |
| `$cat_info` | Kategori bilgileri |
| `$lang` | Dil dosyası metinlerinden oluşan dizi |
| `$_TIME` | Sistemin UNIX zamanı (offset dahil) |

### API (engine/api/api.class.php)
API, sürüm güncellemelerinden etkilenmemek için kullanılır:
`include ('engine/api/api.class.php');`
*   `$dle_api->take_user_by_id(int $id);`
*   `$dle_api->send_pm_to_user(int $user_id, string $subject, string $text, string $from);`
*   `$dle_api->load_table(string $table, ...);`

---

## 📜 6. plugin.xml: Sanal Dosya Yönetimi

Eklentinizin kurulumu ve dosya müdahaleleri burada tanımlanır.

### Kritik İşlemler:
*   **`<mysqlinstall>`**: Kurulum SQL'i (`{prefix}` kullanımı zorunlu).
*   **`<mysqldelete>`**: Silme SQL'i (`DROP TABLE IF EXISTS {prefix}_test;`).
*   **`<file name="...">`**: Müdahale edilecek dosya.
*   **`<operation action="create">`**: Yeni bir dosya oluşturur (Özellikle modül linkleri için).
*   **`<operation action="after">`**-**`before`**-**`replace`**: Kod enjeksiyonu.

---

## 🛡️ 7. Güvenlik ve Kod Standartları

1.  **Direct Access Block:** Her PHP dosyasının tepesine ekleyin:
    `if( !defined( 'DATALIFEENGINE' ) OR !defined( 'LOGGED_IN' ) ) die( "Hacking attempt!" );`
2.  **DLEPlugins::Check() (Hayati):** Dosya include ederken her zaman:
    `include (DLEPlugins::Check(ENGINE_DIR . '/modules/myscript.php'));`
3.  **CHMOD Güvenliği:** Yazma izni (777) olan klasörlerde asla PHP dosyası bulundurmayın.
4.  **SQL Protection:** Gelen verileri `$db->safesql()` süzgecinden geçirmeden sorguya sokmayın.

---
Bu döküman, DLE 19.x mimarisindeki tüm kritik çekirdek dosyaların (`inc`, `modules`, `ajax`) ve native fonksiyonların (`DLEPush`, `DLEconfirm`) kusursuz birleşimidir. 🚀
