# DLE 19.x+ Eklenti Geliştirme Master Rehberi

Bu proje, **DataLife Engine (DLE) 19.1** ve üzeri sürümler için %100 uyumlu, yüksek performanslı ve native görünümlü eklentiler geliştirmek isteyenler için hazırlanmış kapsamlı bir teknik dökümantasyondur.

---

## 🗂️ 1. Eklenti Anatomisi ve Dosya Yapısı

Profesyonel bir DLE eklentisi, çekirdek dosyalara asla dokunmaz. **Sanal Dosya Sistemi (VFS)** üzerinden çalışır. Klasör düzeniniz şu şekilde olmalıdır:

```text
Eklenti_Paketi/
├── plugin.xml           # Eklentinin kalbi (Kurulum, SQL ve Dosya Müdahaleleri)
└── upload/              # Sunucuya yüklenecek statik ve PHP dosyaları
    ├── engine/
    │   ├── inc/         # Yönetim Paneli (Admin) dosyaları
    │   ├── modules/     # Site Ön Yüz (Frontend) dosyaları
    │   └── ajax/        # Arka plan (Controller) işlemleri
    ├── public/          # [v19+] Dışa açık JS, CSS ve Görseller
    └── templates/       # TPL (Şablon) dosyaları
```

---

## 📜 2. `plugin.xml` Standartları

Sizin eklentinizin nasıl kurulacağını DLE'ye XML söyler. Aşağıdaki yapı, v19.x için en güncel ve güvenli örnektir (`{prefix}` kullanımı zorunludur).

```xml
<?xml version="1.0" encoding="utf-8"?>
<dleplugin>
	<name>MWS Master Plugin</name>
	<version>1.2</version>
	<dleversion>19.1</dleversion>
	<versioncompare>greater</versioncompare>
	<upgradeurl>https://api.siteniz.com/update</upgradeurl>
	<filedelete>1</filedelete>

	<!-- Veritabanı Kurulumu -->
	<mysqlinstall><![CDATA[
		CREATE TABLE IF NOT EXISTS `{prefix}_mws_data` (
			`id` INT NOT NULL AUTO_INCREMENT,
			`title` VARCHAR(255) NOT NULL,
			PRIMARY KEY (`id`)
		) ENGINE=InnoDB DEFAULT CHARSET={charset};
	]]></mysqlinstall>

	<!-- Dosya Müdahalesi (Virtual File System) -->
	<file name="engine/modules/main.php">
		<operation action="after">
			<searchcode><![CDATA[include_once (DLEPlugins::Check(ENGINE_DIR . '/modules/eklenti.php'));]]></searchcode>
			<replacecode><![CDATA[// Kendi kodunuzu buraya enjekte edin]]></replacecode>
			<enabled>1</enabled>
		</operation>
	</file>
</dleplugin>
```

---

## 🎨 3. Admin Panel UI/UX Kuralları (Native Tasarım)

DLE 19.1 yönetimi **Bootstrap 3** tabanlı, özel olarak özelleştirilmiş bir iskelet kullanır. Eklentinizin native görünmesi için aşağıdaki sınıfları kullanmalısınız:

### A. Panel ve Kart Yapısı
Sayfalarınızı `panel-flat` veya `panel-default` içine sarın:
```html
<div class="panel panel-default">
    <div class="panel-heading border-bottom">
        <h6 class="panel-title text-semibold">Genel Yapılandırma</h6>
    </div>
    <div class="panel-body">
        <!-- İçerik -->
    </div>
</div>
```

### B. Yatay Formlar (Horizontal Forms)
Form elemanları için `col-md-2` (label) ve `col-md-10` (input) düzenini izleyin:
```html
<div class="form-group">
    <label class="control-label col-md-2 col-sm-3">Ayar İsmi</label>
    <div class="col-md-10 col-sm-9">
        <input type="text" name="set" class="form-control width-400">
        <i class="help-button text-primary-600 fa fa-question-circle position-right" data-rel="popover" data-trigger="hover" data-content="Açıklama metni"></i>
    </div>
</div>
```

### C. Native Buton Ailesi
DLE "Raised" (yükseltilmiş) butonları sever:
- **Primary (Mavi):** `btn bg-primary-600 btn-sm btn-raised`
- **Success (Yeşil):** `btn bg-teal btn-sm btn-raised`
- **Danger (Kırmızı):** `btn bg-danger-600 btn-sm btn-raised`
- **Info (Açık Mavi):** `btn bg-info-800 btn-sm btn-raised`

### D. Dinamik Navigasyon (Navbar Collapse)
Mobilde "Hamburger" menüye dönüşen, sekmeli geçişler için:
```html
<div class="navbar navbar-default navbar-component navbar-xs">
    <ul class="nav navbar-nav visible-xs-block">
        <li class="full-width text-center">
            <a data-toggle="collapse" data-target="#navbar-filter"><i class="fa fa-bars"></i></a>
        </li>
    </ul>
    <div class="navbar-collapse collapse" id="navbar-filter">
        <ul class="nav navbar-nav">
            <li class="active"><a href="#"><i class="fa fa-cog"></i> Ayarlar</a></li>
            <li><a href="#"><i class="fa fa-shield"></i> Güvenlik</a></li>
        </ul>
    </div>
</div>
```

---

## ⚡ 4. JavaScript & AJAX & Bildirimler

DLE'nin yerleşik JS kütüphanesini kullanmak, eklentinin stabilitesini artırır.

### DLEPush (Toast Bildirimleri)
Gelişmiş bildirim altyapısıdır. `DLEPush.tur('Mesaj', 'Başlık', Süre);` şeklinde kullanılır.
```javascript
// Temel bildirimler
DLEPush.success('Kayıt başarılı!'); // Yeşil
DLEPush.info('Lütfen bekleyin...'); // Mavi
DLEPush.warning('Dikkatiniz gerekiyor!'); // Sarı
DLEPush.error('Hata oluştu!', 'Sistem', 5000); // Kırmızı (5 saniye ekranda kalır)

// Pratik Uygulama (Form Validasyonu)
function validate() {
    if ($('#title').val() == '') {
        DLEPush.error('Başlık alanı boş bırakılamaz!');
        return false;
    }
    return true;
}
```

### AJAX ve Loading
Veri çekerken ekranı kilitleyip spinner göstermek standarttır:
```javascript
ShowLoading(''); // Ekranı kilitle & Spinner başlat
$.post('index.php?controller=ajax&mod=test', { data: 'val', user_hash: dle_login_hash }, function(res) {
    HideLoading(''); // Ekranı aç
    DLEPush.success(res.message);
}, 'json').fail(function(xhr) {
    HideLoading('');
    DLEPush.error('HTTP Error: ' + xhr.status);
});
```

### TinyMCE Senkronizasyonu (Veri Gönderme/Alma)
Eğer formunuzda zengin metin editörü varsa, veriyi okumadan önce mutlaka senkronize etmelisiniz:
```javascript
// 1. Editördeki veriyi textarea'ya aktar (ZORUNLU)
tinyMCE.triggerSave();

// 2. Editöre dışarıdan veri basmak
tinymce.get('short_story').setContent('<p>Bot ile çekilen içerik...</p>');

// 3. Editördeki HTML içeriği JS ile almak
var editor_html = tinymce.get('short_story').getContent();
```

### Ek Alanlar (xfields) Kullanımı
DLE'nin ek alanlarını PHP tarafında hazırlayıp forma basmak için:
```php
$xfields = DLEXFields::FieldsList();
$xf_root = array();
foreach ($xfields['fields'] as $value) {
    $xf_root[] = $value; // DLE her alanı bir form-group olarak hazırlar
}
echo implode('', $xf_root); // Formun içine enjekte edin
```

---

## 🛡️ 5. Güvenlik ve En İyi Uygulamalar (Best Practices)

1.  **Hacking Attempt:** Her PHP dosyasının başına güvenliği tetiklemek için şunu ekleyin:
    `if( !defined( 'DATALIFEENGINE' ) OR !defined( 'LOGGED_IN' ) ) die( "Hacking attempt!" );`
2.  **DLEPlugins::Check() (ZORUNLU):** DLE'de bir dosyayı `include` veya `require` ile çağırırken asla doğrudan yol yazmayın. DLE, eklenti müdahalelerini **Virtual File System (VFS)** üzerinden yönetir. Bu fonksiyonu kullanmazsanız yaptığınız XML modifikasyonları çalışmaz ve eklentiniz pasifken bile dosya okunur.
    *   ❌ **Yanlış:** `include (ENGINE_DIR . '/modules/dosya.php');`
    *   ✅ **Doğru:** `include (DLEPlugins::Check(ENGINE_DIR . '/modules/dosya.php'));`
3.  **Cross-Site Request Forgery (CSRF):** AJAX isteklerinde mutlaka `user_hash: dle_login_hash` gönderin.
4.  **Veritabanı:** Sorgularda tabloları `{prefix}_users` şeklinde yazın, asla statik `dle_` kullanmayın.
5.  **Temizlik:** `mysqldelete` etiketinde mutlaka `DROP TABLE` komutunuzu bulundurun ki eklenti silindiğinde geride çöp bırakmasın.

---

Bu döküman, DLE topluluğu için modern standartları teşvik etmek amacıyla hazırlanmıştır. İyi kodlamalar! 🚀
