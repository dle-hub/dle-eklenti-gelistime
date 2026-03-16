# DataLife Engine (DLE) 19.x+ Geliştirici Master Rehberi (A'dan Z'ye)

Bu rehber, DLE çekirdek kodları (`inc`, `modules`, `ajax`) incelenerek hazırlanmış, %100 native uyumlu bir eklenti geliştirme iskeletidir.

---

## 🏗️ 1. DLE Eklenti Mimarisi (Kutsal Üçlü)

Profesyonel bir eklenti şu üç sac ayağı üzerine kurulur:
1.  **Yönetim Paneli (`engine/inc/`):** Ayarların yapıldığı görsel arayüz.
2.  **Site Ön Yüz (`engine/modules/`):** Ziyaretçilerin gördüğü içerik mantığı.
3.  **AJAX İşleyici (`engine/ajax/`):** Veritabanı ile arka planda konuşan motor.

---

## 🛠️ 2. YÖNETİM PANELİ İSKELETİ (`engine/inc/yourplugin.php`)

Bu dosya DLE Admin panelinin bir parçası gibi görünmeli ve davranmalıdır.

```php
<?php
if( !defined( 'DATALIFEENGINE' ) OR !defined( 'LOGGED_IN' ) ) die( "Hacking attempt!" );

// 1. Başlık ve İkon (Native DLE Tasarımı)
echoheader( "<i class=\"fa fa-cubes position-left\"></i><span class=\"text-semibold\">Master Plugin v1.0</span>", "Eklenti yönetim merkezi" );

// 2. Dinamik Navbar (navbar-collapse collapse) - AYARLAR VE FİLTRELER İÇİN
echo <<<HTML
<div class="navbar navbar-default navbar-component navbar-xs systemsettings">
    <ul class="nav navbar-nav visible-xs-block">
        <li class="full-width text-center"><a data-toggle="collapse" data-target="#nav-menu"><i class="fa fa-bars"></i></a></li>
    </ul>
    <div class="navbar-collapse collapse" id="nav-menu">
        <ul class="nav navbar-nav">
            <li class="active"><a href="#"><i class="fa fa-cog position-left"></i> Ana Ayarlar</a></li>
            <li><a href="#"><i class="fa fa-list position-left"></i> Kayıtlar</a></li>
        </ul>
        <ul class="nav navbar-nav navbar-right">
            <li><a href="#" onclick="myAjaxAction(); return false;" class="bg-primary-600 text-white"><i class="fa fa-bolt"></i> AJAX Tetikle</a></li>
        </ul>
    </div>
</div>
HTML;

// 3. Form ve Panel Yapısı (Native Hiyerarşi)
echo <<<HTML
<div class="panel panel-default">
    <div class="panel-heading border-bottom">
        <h6 class="panel-title text-semibold">Yapılandırma Formu</h6>
    </div>
    <div class="panel-body">
        <form class="form-horizontal">
            <div class="form-group">
                <label class="control-label col-md-2">Ayar Başlığı</label>
                <div class="col-md-10">
                    <input type="text" id="plugin_title" class="form-control width-450 position-left">
                    <i class="help-button text-primary-600 fa fa-question-circle position-right" data-rel="popover" data-trigger="hover" data-content="Buraya yazdığınız veri AJAX ile gönderilecektir."></i>
                </div>
            </div>
            <div class="form-group">
                <label class="control-label col-md-2">Durum</label>
                <div class="col-md-10">
                    <input class="switch" type="checkbox" id="plugin_status" checked>
                </div>
            </div>
        </form>
    </div>
</div>
HTML;

echofooter();
?>
```

---

## 🌐 3. SİTE ÖN YÜZ İSKELETİ (`engine/modules/yourplugin.php`)

Frontend modülleri şablon motoruyla konuşur.

```php
<?php
if( !defined( 'DATALIFEENGINE' ) ) die( "Hacking attempt!" );

// DLE Global Değişkenleri Hazırdır: $db, $tpl, $config, $is_logged, $member_id, $lang

$tpl->load_template( 'yourplugin.tpl' );

// Dinamik Veri Basma
$tpl->set( '{title}', "Merhaba " . ($is_logged ? $member_id['name'] : $lang['all_guest']) );

// Koşullu Bloklar
if( $config['allow_comments'] ) {
    $tpl->set_block( "'\\[comments\\](.*?)\\[/comments\\]'si", "\\1" );
} else {
    $tpl->set_block( "'\\[comments\\](.*?)\\[/comments\\]'si", "" );
}

$tpl->compile( 'content' );
$tpl->clear();
?>
```

---

## ⚡ 4. AJAX SİSTEMİ VE DLEPush (`engine/ajax/yourplugin.php`)

AJAX motoru, eklentinin arka plandaki beynidir.

### A. Javascript Tarafı (Admin Paneline Eklenir)
```javascript
function myAjaxAction() {
    var title = $('#plugin_title').val();
    
    // 1. Ekranı Kilitle (Spinner)
    ShowLoading(''); 

    // 2. AJAX İstek (Controller üzerinden)
    $.post("index.php?controller=ajax&mod=yourplugin", { 
        title: title, 
        user_hash: dle_login_hash // Güvenlik Hash'i ZORUNLUDUR
    }, function(data) {
        
        HideLoading(''); // 3. Kilidi Kaldır
        
        if (data.success) {
            // DLEPush: Native Bildirim Sistemi
            DLEPush.success(data.message, 'Başarılı'); 
        } else {
            DLEPush.warning(data.error, 'Uyarı');
        }

    }, 'json').fail(function(xhr) {
        HideLoading('');
        DLEPush.error('Hata: ' + xhr.status, 'Sunucu Hatası');
    });
}
```

### B. PHP Tarafı (engine/ajax/yourplugin.php)
```php
<?php
if( !defined( 'DATALIFEENGINE' ) ) die( "Hacking attempt!" );

// Güvenlik: Hash Kontrolü
if( $_POST['user_hash'] == "" OR $_POST['user_hash'] != $dle_login_hash ) {
    die( json_encode( array( 'success' => false, 'error' => 'Geçersiz Hash!' ) ) );
}

// Veritabanı İşlemi ($db sınıfı hazırdır)
$title = $db->safesql( $_POST['title'] );

header('Content-Type: application/json; charset=' . $config['charset']);
echo json_encode( array( 'success' => true, 'message' => 'Veri Kaydedildi: ' . $title ) );
?>
```

---

## 📦 5. DLE DAHİLİ ARAÇLARI (Global Veriler)

Aşağıdaki verilere modülleriniz içinden doğrudan erişebilirsiniz:

| Veri | Açıklama |
| :--- | :--- |
| `$db` | Veritabanı Sınıfı. `$db->super_query("...")` ile tek, `$db->query("...")` ile çoklu işlem yapılır. |
| `$tpl` | Şablon Sınıfı. `.tpl` dosyalarını işlemek için kullanılır. |
| `$config` | DLE genel ayarları (site_url, charset, version vb.) |
| `$member_id` | Giriş yapan kullanıcının veritabanı satırı (Array). |
| `$user_group` | Kullanıcı gruplarının yetki haritası. |
| `$lang` | Dil dosyasındaki çeviriler. |
| `DLEPlugins::Check()` | **ZORUNLU:** Dosya include edilirken sanal sistemi aktif eder. |

---

## 📜 6. plugin.xml STANDARTLARI

```xml
<dleplugin>
    <name>Eklenti</name>
    <mysqlinstall><![CDATA[CREATE TABLE ...]]></mysqlinstall>
    <mysqldelete><![CDATA[DROP TABLE IF EXISTS {prefix}_test;]]></mysqldelete>
    
    <!-- Sanal Dosya Enjeksiyonu -->
    <file name="engine/engine.php">
        <operation action="after">
            <searchcode><![CDATA[case "stats" :]]></searchcode>
            <replacecode><![CDATA[case "myplugin" : include (DLEPlugins::Check(ENGINE_DIR . '/modules/myplugin.php')); break;]]></replacecode>
        </operation>
    </file>
</dleplugin>
```

---
Bu döküman DLE 19.x mimarisindeki tüm kritik yolları (`inc`, `modules`, `ajax`) ve `DLEPush` gibi native fonksiyonları kapsar. 🚀
