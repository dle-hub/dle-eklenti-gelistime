# DataLife Engine (DLE) 19.x+ Geliştirici Master Rehberi (A'dan Z'ye Tam Kapsam)

Bu rehber, DLE 19.1+ sürümleri için profesyonel eklenti geliştirme standartlarını, çekirdek kod mantığını ve native UI/UX kurallarını içeren **en kapsamlı** teknik dökümandır. Hiçbir detay atlanmadan, tüm bileşenler bir araya getirilmiştir.

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

## 🎨 2. Native Admin Paneli Tasarımı (engine/inc)

Admin paneli **Bootstrap 3** tabanlıdır. Eklentinizin sistemin bir parçası gibi görünmesi için şu yapıları kullanmalısınız:

### A. Sayfa Başlığı ve Giriş Koruması
```php
<?php
if( !defined( 'DATALIFEENGINE' ) OR !defined( 'LOGGED_IN' ) ) die( "Hacking attempt!" );

// echoheader( Modül İkonu ve Başlık, Alt Başlık/Açıklama )
echoheader( "<i class=\"fa fa-cubes position-left\"></i><span class=\"text-semibold\">Eklenti Başlığı</span>", "Modül ayarları ve yönetimi" );

// Kodlarınız...

echofooter();
?>
```

### B. Dinamik Navigasyon ve Filtreleme (navbar-collapse collapse)
Mobilde hamburger menüye dönüşen, en profesyonel native navbar yapısı:
```html
<div class="navbar navbar-default navbar-component navbar-xs systemsettings">
    <!-- Mobil Ekranlar İçin Hamburger Butonu -->
    <ul class="nav navbar-nav visible-xs-block">
        <li class="full-width text-center">
            <a data-toggle="collapse" data-target="#navbar-filter-menu"><i class="fa fa-bars"></i></a>
        </li>
    </ul>
    
    <!-- Menü İçeriği (Masaüstü ve Açılmış Mobil) -->
    <div class="navbar-collapse collapse" id="navbar-filter-menu">
        <ul class="nav navbar-nav">
            <li class="active font-bold"><a href="#"><i class="fa fa-cog position-left"></i> Genel Ayarlar</a></li>
            <li><a href="#"><i class="fa fa-tasks position-left"></i> Görev Listesi</a></li>
            <li><a href="#"><i class="fa fa-shield position-left"></i> Güvenlik Logları</a></li>
        </ul>
        
        <!-- Sağa Dayalı Arama ve Eylem Butonları -->
        <ul class="nav navbar-nav navbar-right">
            <li>
                <div class="navbar-form">
                    <input type="text" name="search" class="form-control" placeholder="Hızlı Ara...">
                </div>
            </li>
            <li><a href="#" onclick="myFunction(); return false;" class="bg-primary-600 text-white"><i class="fa fa-plus"></i> Kayıt Ekle</a></li>
        </ul>
    </div>
</div>
```

### C. Paneller ve Tab (Sekme) Sistemleri
```html
<div class="panel panel-default">
    <div class="panel-heading">
        <ul class="nav nav-tabs nav-tabs-solid">
            <li class="active"><a href="#tab_main" data-toggle="tab"><i class="fa fa-home position-left"></i> Ana Bölüm</a></li>
            <li><a href="#tab_extra" data-toggle="tab"><i class="fa fa-wrench position-left"></i> Gelişmiş</a></li>
        </ul>
    </div>
    
    <div class="panel-tab-content tab-content">
        <div class="tab-pane active" id="tab_main">
            <div class="panel-body">Form veya Liste İçeriği</div>
        </div>
        <div class="tab-pane" id="tab_extra">
            <div class="panel-body">Alternatif Ayarlar</div>
        </div>
    </div>
</div>
```

### D. Form Standartları ve Tooltipler
*   **form-horizontal:** Yatay form düzeni.
*   **col-md-2 / col-md-10:** Label ve Input genişlik oranları.
*   **help-button:** `i.help-button` sınıfı ve `data-rel="popover"` ile native tooltip eklenir.
*   **icheck:** Kare kutucuklu checkboxlar.
*   **switch:** Apple tarzı açma/kapama düğmeleri.
*   **btn-raised:** Yükseltilmiş, gölgeli butonlar (`btn bg-primary-600 btn-raised`).

---

## ⚡ 3. DLEPush & AJAX & Bildirimler (Zorunlu Native JS API)

DLE'de kullanıcı etkileşimi için asla dış kütüphane veya `alert` kullanmayın.

### A. DLEPush (Toast Bildirim Sistemi)
Gelişmiş bildirim sistemidir. `DLEPush.tur(Mesaj, Başlık, Süre);` şeklinde çalışır.
```javascript
DLEPush.success('İşlem başarıyla tamamlandı!', 'Başarılı'); // Yeşil
DLEPush.info('Yeni bir güncelleme mevcut.', 'Bilgi'); // Mavi
DLEPush.warning('Lütfen zorunlu alanları doldurun.', 'Dikkat'); // Sarı
DLEPush.error('Veritabanı bağlantı hatası!', 'Kritik Hata', 5000); // Kırmızı (5 sn ekranda kalır)
```

### B. AJAX İskeleti ve Spinner (loading)
AJAX isteklerinde ekranı kilitleyip `ShowLoading` kullanmak DLE standartıdır.
```javascript
function savePlugin() {
    // 1. Varsa editörü senkronize et (ZORUNLU)
    if(typeof tinyMCE !== "undefined") tinyMCE.triggerSave();

    // 2. Ekranı Kilitle ve Spinner Çıkar
    ShowLoading(''); 

    // 3. İstek Gönder
    $.post("index.php?controller=ajax&mod=your_plugin_mod", { 
        title: $('#title').val(), 
        user_hash: dle_login_hash // Güvenlik Hash'i ŞARTTIR
    }, function(res) {
        
        HideLoading(''); // 4. Kilidi Kaldır
        
        if (res.success) {
            DLEPush.success(res.message);
        } else {
            DLEPush.error(res.error);
        }

    }, 'json').fail(function(xhr) {
        HideLoading('');
        DLEPush.error('Sunucu Hatası: ' + xhr.status);
    });
}
```

### C. Onay Pencereleri (Confirm)
```javascript
DLEconfirm('Bu işlemi onaylıyor musunuz?', 'Onay Gerekiyor', function() {
    // Tamam denilince yapılacaklar
});

DLEconfirmDelete('Kalıcı olarak silinecektir, emin misiniz?', 'Silme Onayı', function() {
    // Silme onayı verilince yapılacaklar
});
```

### D. TinyMCE Gelişmiş Manipülasyon
```javascript
// Editöre dışarıdan veri basmak
tinymce.get('short_story').setContent('<p>İçerik...</p>');

// Editördeki HTML'i ham olarak almak
var html = tinymce.get('short_story').getContent();
```

---

## 📦 4. DLE Global Değişkenleri ve API Sistemi

Eklenti dosyalarınızda (`inc`, `modules` veya `ajax`) aşağıdaki değişkenlere **hiçbir tanımlama yapmadan** erişebilirsiniz.

### Global Değişkenler
| Değişken | Açıklama |
| :--- | :--- |
| `$db` | Veritabanı sınıfı (`$db->super_query`, `$db->safesql`, `$db->query`) |
| `$config` | Tüm sistem ayarları (`$config['site_url']`, `$config['version']` vb.) |
| `$member_id` | Giriş yapan kullanıcının tüm veritabanı verileri (Array) |
| `$user_group` | Grupların yetki ve ayar haritası (Array) |
| `$is_logged` | Kullanıcı oturum açmış mı? (true/false) |
| `$tpl` | Şablon motoru sınıfı (`load_template`, `set`, `compile`, `clear`) |
| `$cat_info` | Kategori bilgileri ve hiyerarşisi |
| `$lang` | Dil dosyasındaki çeviri metinleri |
| `$_TIME` | Sistemin güncel UNIX zamanı (offset dahil) |
| `$dle_module` | Mevcut sayfa/bölüm ismi veya URL'deki `do` parametresi |

### DLE API (`engine/api/api.class.php`)
Sürümler arası uyumluluk için API kullanımı tavsiye edilir:
`include ('engine/api/api.class.php');`
*   **Kullanıcı:** `$dle_api->take_user_by_id(int $id, string $fields);`
*   **PM Gönder:** `$dle_api->send_pm_to_user(int $user_id, string $subject, string $text, string $from);`
*   **Veri Çek:** `$dle_api->load_table(string $table, string $fields, ...);`
*   **Önbellek:** `$dle_api->save_to_cache(string $name, mixed $data);`

---

## 🌐 5. Site Ön Yüz (Frontend - engine/modules)

### A. Şablona Dahil Etme ve Parametre Sistemi
```smarty
{* Statik Dahil Etme *}
{include file="engine/modules/mymod.php"}

{* Değişken Gönderme (Modül içinde $id ve $action olarak yakalanır) *}
{include file="engine/modules/mymod.php?id=5&action=list"}

{* Dinamik Etiket Gönderimi *}
{include file="engine/modules/mymod.php?post_id={news-id}"}
```

### B. Bölüm Sayfası Oluşturma (aviable)
`main.tpl`'de haberler yerine kendi modül modülünüzün çıktısını basmak:
```smarty
[aviable=myapp] {include file="engine/modules/myapp.php"} [/aviable]
[not-aviable=myapp] {content} [/not-aviable]
```

---

## 📜 6. plugin.xml: Eklentinin Yaşam Döngüsü

Eklenti kurulumu, veritabanı tabloları ve dosya müdahaleleri burada tanımlanır.

### A. Veritabanı İşlemleri
*   **`<mysqlinstall>`**: Kurulumda çalışacak SQL kodları.
*   **`<mysqldelete>`**: Silme anında çalışacak kodlar (`DROP TABLE IF EXISTS {prefix}_tablo;`).

### B. Dosya Müdahaleleri ve Create Action
*   **`<operation action="create">`**: DLE'de olmayan bir dosyayı (Örn: Modül linki) sıfırdan yaratmak için kullanılır.
*   **`{prefix}` / `{charset}` / `{userprefix}`**: Veritabanı uyumluluğu için bu etiketler zorunludur.

---

## 🛡️ 7. Güvenlik ve Kod Standartları (Altın Kurallar)

1.  **Direct Access Block:** Her PHP dosyasının tepesine ekleyin:
    `if( !defined( 'DATALIFEENGINE' ) ) die( "Hacking attempt!" );`
2.  **DLEPlugins::Check() (HAYATİ):** Bir dosya include/require edilecekse mutlaka:
    `include (DLEPlugins::Check(ENGINE_DIR . '/modules/myscript.php'));`
    *(Bu fonksiyon olmadan VFS üzerinden yapılan XML müdahaleleri çalışmaz).*
3.  **SQL Protection:** Gelen verileri her zaman `$db->safesql()` süzgecinden geçirin.
4.  **CHMOD Güvenliği:** Yazma izni (777) olan klasörlerde asla PHP dosyası barındırmayın.

---
🚀 Bu rehber, profesyonel bir DLE eklentisi geliştirmek için ihtiyacınız olan **tüm** bilgileri tek bir potada eritir.
