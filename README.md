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

---

## 🎨 9. YÖNETİM PANELİ UI STANDARTLARI REHBERİ

DLE Admin paneli, tutarlı bir kullanıcı deneyimi için belirli tasarım kalıpları (Design Patterns) üzerine inşa edilmiştir. Eklentinizin sistemle bütünleşmesi için bu standartları takip edin.

### A. Layout Hiyerarşisi (Sayfa Akışı)
Profesyonel bir DLE sayfası şu silsile ile kurgulanır:
1.  **echoheader():** Modül kimliğini belirler.
2.  **Navbar/Toolbar:** `navbar-collapse` ile filtreleme ve toplu eylemler.
3.  **Alerts:** `alert-info` veya `alert-warning` ile bilgilendirme.
4.  **Ana Panel:** `panel-default` veya sekmeli (tabs) panel.
5.  **Tablolar/Liste:** `panel-flat` veya `media-list`.
6.  **Footer:** `panel-footer` içinde `btn-raised` butonlar.

### B. Semantik Renkler ve İşlevleri
| Renk Kodu | CSS Sınıfı | Kullanım Amacı |
| :--- | :--- | :--- |
| **Primary (Mavi)** | `bg-primary-600` | Ana eylemler, ayarlar başlıkları. |
| **Success (Teal)** | `bg-teal` | Kaydetme, ekleme, aktif/onaylı durumu. |
| **Danger (Kırmızı)** | `bg-danger-600` | Silme, durdurma, pasif/hata durumu. |
| **Warning (Sarı)** | `bg-warning` | Uyarı, kısıtlama veya dikkat çeken notlar. |
| **Info (Koyu Mavi)** | `bg-info-800` | Yardım metinleri ve istatistik başlıkları. |
| **Secondary (Gri)** | `bg-slate-600` | İptal, geri dön, önizleme butonları. |

### C. Spacing (Boşluk) ve Hizalama Yardımcıları
DLE Admin panelinde dikey boşluklar için şu sınıfları kullanın:
*   **Küçük Boşluk:** `mt-10` (10px margin-top).
*   **Orta Boşluk:** `mt-15` (15px margin-top).
*   **Büyük Boşluk:** `mt-20` (20px margin-top).
*   **İkon Hizalama:** İkon metnin solundaysa `position-left`, sağındaysa `position-right`.

### D. İleri Düzey Komponent Örnekleri

**1. Media List (Zarif Linkleme):**
İkonlu ve açıklamalı modül içi linkler için kullanılır.
```html
<div class="panel-body list-bordered">
    <div class="row box-section">
        <div class="col-sm-6 media-list media-list-linked">
            <a class="media-link" href="#">
                <div class="media-left"><img src="public/adminpanel/images/tools.png" class="img-lg"></div>
                <div class="media-body">
                    <h6 class="media-heading text-semibold">Genel Yapılandırma</h6>
                    <span class="text-muted">Eklentinin tüm temel parametrelerini buradan yönetebilirsiniz.</span>
                </div>
            </a>
        </div>
    </div>
</div>
```

**2. Panel Flat (Ayarlar Tablosu):**
Geniş satırlı ve temiz ayar listeleri için tercih edilir.
```html
<div class="panel panel-flat">
    <div class="panel-heading border-bottom">Ayarlar</div>
    <table class="table table-striped">
        <tr>
            <td class="col-md-7">
                <h6 class="text-semibold">Dinamik Önizleme</h6>
                <span class="text-muted">Değişikliklerin anında etkisini gösterir.</span>
            </td>
            <td class="col-md-5">
                <input class="switch" type="checkbox" checked>
            </td>
        </tr>
    </table>
</div>
```

**3. Status Labels (Durum Etiketleri):**
Tablolarda durumu belirtmek için native spanlar:
```html
<span class="label label-success">Yüklendi</span>
<span class="label label-danger">Bekliyor</span>
<span class="label label-info">İşlemde</span>
```

### E. Native Tablo Yapısı (Data Tables)
DLE'de veri listeleme işlemleri için `panel-flat` içine gömülü responsive tablolar kullanılır.

**1. Tablo Sınıfları:**
```html
<div class="panel panel-default">
    <div class="panel-heading">Veri Listesi</div>
    <table class="table table-striped table-xs table-hover">
        <thead>
            <tr>
                <th class="hidden-xs">#</th>
                <th>Başlık</th>
                <th class="text-center">İşlem</th>
                <th style="width: 40px"><input type="checkbox" name="master_box" onclick="ckeck_uncheck_all();" class="icheck"></th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td class="hidden-xs">1</td>
                <td>Örnek Veri</td>
                <td class="text-center"><a href="#"><i class="fa fa-pencil"></i></a></td>
                <td><input name="selected[]" value="1" type="checkbox" class="icheck"></td>
            </tr>
        </tbody>
    </table>
    <!-- Toplu İşlemler -->
    <div class="panel-footer">
        <div class="pull-right">
            <select name="action" class="uniform position-left">
                <option value="">İşlem Seçin</option>
                <option value="delete">Sil</option>
            </select>
            <input class="btn bg-teal btn-sm btn-raised" type="submit" value="Gönder">
        </div>
    </div>
</div>
```

**2. Masaüstü/Mobil Görünürlük:**
*   `hidden-xs`: Mobilde sütunu gizler.
*   `hidden-sm`: Tablette sütunu gizler.
*   `text-nowrap`: Sütun içeriğinin kırılmasını önler.

**3. Çoklu Seçim (Multi-Select) Mantığı:**
DLE'de tablo satırlarını seçmek için `master_box` kullanılır. Seçilen satıra otomatik olarak `warning` (sarı zemin) sınıfı atanır:
```javascript
function ckeck_uncheck_all() {
    var frm = document.editnews;
    for (var i=0;i<frm.elements.length;i++) {
        var elmnt = frm.elements[i];
        if (elmnt.type=='checkbox') {
            if(frm.master_box.checked == true) $(elmnt).parents('tr').addClass('warning');
            else $(elmnt).parents('tr').removeClass('warning');
            elmnt.checked = frm.master_box.checked;
        }
    }
}
```

---

## 📜 11. ÖRNEK KURULUM DOSYASI (plugin.xml)

Aşağıdaki örnek, DataLife Engine eklenti sisteminin (`plugin.xml`) tüm yeteneklerini (SQL sorguları, dosya oluşturma ve kod enjeksiyonu) gösteren kapsamlı bir şablondur:

```xml
<?xml version="1.0" encoding="utf-8"?>
<dleplugin>
	<name>Eklenti adı</name>
	<description>Kısa açıklama</description>
	<icon>public/adminpanel/images/reaction.png</icon>
	<version>1.2</version>
	<dleversion>19.0</dleversion>
	<versioncompare>greater</versioncompare>
	<upgradeurl></upgradeurl>
	<filedelete>0</filedelete>
	<needplugin></needplugin>
	<mnotice>0</mnotice>
	<mysqlinstall><![CDATA[Eklentiyi kurarken]]></mysqlinstall>
	<mysqlupgrade><![CDATA[Eklentiyi güncellerken:]]></mysqlupgrade>
	<mysqlenable><![CDATA[Eklentiyi aktif ederken:]]></mysqlenable>
	<mysqldisable><![CDATA[Eklentiyi devredışı bırakırken:]]></mysqldisable>
	<mysqldelete><![CDATA[Eklentiyi kaldırırken:]]></mysqldelete>
	<phpinstall><![CDATA[]]></phpinstall>
	<phpupgrade><![CDATA[]]></phpupgrade>
	<phpenable><![CDATA[]]></phpenable>
	<phpdisable><![CDATA[]]></phpdisable>
	<phpdelete><![CDATA[]]></phpdelete>
	<notice><![CDATA[MYSQL sorguları yazarken aşağıda yer alan servis etiketlerinide kullanabilirsiniz:

{prefix} - bu etiket tabloya ön ek getirir ve bu ön ek ile kullanılmasını sağlar.
{userprefix} - birden fazla site aynı veritabanını kullanıyorsa bu etiketten yararlanılabilirsiniz. -
{charset} - bu etiket ile tabloların kodlama yapısını ayarlayabilirsiniz örneğin utf-8, utf8mb4 gibi.

Lütfen dikkat: Eklentiyi yüklediğinizde, eklenti etkinleştirme tetikleyicisi otomatik olarak etkinleştirilir, ancak eklentiyi kaldırdığınızda eklenti devre dışı bırakma tetikleyicisi kullanılmaz]]></notice>
	<file name="engine/modules/eklenti.php">
		<operation action="after">
			<searchcode><![CDATA[bul ]]></searchcode>
			<replacecode><![CDATA[altına ekle ]]></replacecode>
			<enabled>1</enabled>
		</operation>
		<operation action="before">
			<searchcode><![CDATA[bul ]]></searchcode>
			<replacecode><![CDATA[üstüne ekle]]></replacecode>
			<enabled>1</enabled>
		</operation>
		<operation action="replace">
			<searchcode><![CDATA[bul ]]></searchcode>
			<replacecode><![CDATA[değiştir ]]></replacecode>
			<enabled>1</enabled>
		</operation>
		<operation action="create">
			<replacecode><![CDATA[yeni dosya ]]></replacecode>
			<enabled>1</enabled>
		</operation>
	</file>
	<file name="engine/inc/eklenti.php">
		<operation action="create">
			<replacecode><![CDATA[admin paneli kısmı burada ]]></replacecode>
			<enabled>1</enabled>
		</operation>
	</file>
</dleplugin>
```

---
🚀 Bu rehber DLE 19.x mimarisindeki her detayı kapsayan **eksiksiz** teknik rehberdir. Eklentilerinizi sistem standartlarına uygun şekilde geliştirmek için bu iskeletleri baz alabilirsiniz.
