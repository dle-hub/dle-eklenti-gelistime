# DLE (DataLife Engine) Eklenti Geliştirme Dokümantasyonu

## İçindekiler

1. [Genel Bakış](#genel-bakış)
2. [Frontend Yapısı](#frontend-yapısı)
3. [Klasör Yapısı ve İşlevleri](#klasör-yapısı-ve-işlevleri)
4. [Modules Klasörü Detaylı İnceleme](#modules-klasörü-detaylı-inceleme)
5. [Ajax Klasörü Detaylı İnceleme](#ajax-klasörü-detaylı-inceleme)
6. [Inc Klasörü Detaylı İnceleme](#inc-klasörü-detaylı-inceleme)
7. [Temel Fonksiyonlar ve Kullanımları](#temel-fonksiyonlar-ve-kullanımları)
8. [Veritabanı Yapısı](#veritabanı-yapısı)
9. [Örnek Eklenti Geliştirme](#örnek-eklenti-geliştirme)

---

## Genel Bakış

**DataLife Engine (DLE)**, PHP ve MySQL ile geliştirilmiş, MODMVC (Model-View-Controller) mimarisine yakın yapıda çalışan bir CMS'dir.

### Kullanılan Teknolojiler

- **Backend**: PHP 7.4+
- **Veritabanı**: MySQL 5.7+ / MariaDB
- **Frontend**: jQuery, Bootstrap (teme bağlı)
- **Cache Sistemi**: File-based, Memcached, Redis
- **Template Engine**: Kendi özel template sistemi (dle_template class)

### Temel Özellikler

- SEO dostu URL yapısı
- Çoklu dil desteği
- Özelleştirilebilir template sistemi
- AJAX tabanlı işlemler
- Gelişmiş önbellekleme sistemi
- Rol tabanlı kullanıcı yetkilendirme

---

## Frontend Yapısı

### Template Sistemi

DLE, kendi özel template motorunu kullanır. Template dosyaları `.tpl` uzantılıdır.

**Temel Template Dosyaları:**

```
templates/
├── main.tpl          # Ana şablon (header, footer, navigation)
├── shortstory.tpl    # Haber önizleme şablonu
├── fullstory.tpl     # Tam haber görüntüleme şablonu
├── comments.tpl      # Yorum şablonu
├── navigation.tpl    # Sayfalama şablonu
├── addnews.tpl       # Haber ekleme formu
├── login.tpl         # Giriş formu
├── registration.tpl  # Kayıt formu
└── ...
```

### Template Etiketleri

#### Değişken Etiketleri

```html
{title}           # Haber başlığı
{short-story}     # Kısa içerik
{full-story}      # Tam içerik
{date}            # Tarih
{author}          # Yazar
{comments-num}    # Yorum sayısı
{views}           # Görüntülenme sayısı
{category}        # Kategori adı
{link-category}   # Kategori linki
{news-id}         # Haber ID'si
{rating}          # Oy verme sistemi
{tags}            # Etiketler
```

#### Koşullu Etiketler

```html
[new] ... [/new]                    # Yeni haberler için
[not-new] ... [/not-new]            # Yeni olmayan haberler için
[fixed] ... [/fixed]                # Sabitlenmiş haberler
[not-fixed] ... [/not-fixed]        # Sabitlenmemiş haberler
[comments] ... [/comments]          # Yorum varsa
[not-comments] ... [/not-comments]  # Yorum yoksa
[full-link] ... [/full-link]        # Tam haber linki
[edit] ... [/edit]                  # Düzenleme linki (yetki varsa)
```

#### Fonksiyonel Etiketler

```html
{date=Y-m-d}              # Özel tarih formatı
{image-1}, {image-2}      # İçerikteki resimler
{xfvalue_alanisimi}       # Özel alan (X-Fields) değeri
{custompages_pageid}      # Özel sayfa içeriği
```

### JavaScript Yapısı

DLE, jQuery kütüphanesini temel alır:

```javascript
// Temel DLE JavaScript Fonksiyonları
doRate('plus', news_id)        // Haber oylama
doRate('minus', news_id)       // Haber olumsuz oylama
doCommentsRate('1', comment_id) // Yorum oylama
dle_news_delete(news_id)       // Haber silme (admin)
ShowProfile(username, url)     // Profil gösterimi
save_last_viewed(news_id)      // Son görüntülenenleri kaydet
```

### CSS Yapısı

```css
/* Temel DLE CSS Sınıfları */
.rating              /* Oy verme sistemi */
.unit-rating         /* Oy verme listesi */
.current-rating      /* Mevcut oy durumu */
.comments            /* Yorum alanı */
.comment             /* Tek yorum */
.full-story          /* Tam haber içeriği */
.short-story         /* Kısa haber içeriği */
.navigation          /* Sayfalama */
```

---

## Klasör Yapısı ve İşlevleri

### Ana Dizin Yapısı

```
dle/
├── engine/                 # Çekirdek motor dosyaları
│   ├── modules/           # Kullanıcı tarafı modülleri
│   ├── ajax/              # AJAX istek işleyicileri
│   ├── inc/               # Yönetim paneli dosyaları
│   ├── classes/           # PHP sınıfları
│   ├── cache/             # Önbellek dosyaları
│   └── data/              # Sistem verileri
├── templates/             # Template dosyaları
├── language/              # Dil dosyaları
├── public/                # Ortak kaynaklar (JS, CSS, resimler)
├── uploads/               # Yüklenen dosyalar
└── admin.php              # Yönetim paneli giriş
```

---

## Modules Klasörü Detaylı İnceleme

### 1. show.full.php - Tam Haber Görüntüleme

**Dosya Yolu:** `engine/modules/show.full.php`

**İşlevi:** Tek bir haberin tam sayfa görüntülenmesini sağlar.

**Özellikler:**
- Haber içeriğini veritabanından çeker
- Kategori izinlerini kontrol eder
- Şifre korumalı haberleri yönetir
- Ülke bazlı erişim kontrolü yapar
- Önbellekleme desteği
- Sayfalama (PAGEBREAK) desteği
- Önceki/Sonraki haber navigasyonu
- Sosyal medya paylaşım linkleri
- Schema.org meta etiketleri

**Örnek Kullanım:**

```php
// URL Yapısı
https://siteadi.com/kategori/2024/01/15/haber-adi.html
https://siteadi.com/?newsid=123
```

**Kullanılan Template Değişkenleri:**

```html
{title}                 # Haber başlığı
{full-story}            # Tam içerik
{short-story}           # Kısa içerik
{date}                  # Tarih
{author}                # Yazar
{comments-num}          # Yorum sayısı
{views}                 # Görüntülenme
{category}              # Kategori
{link-category}         # Kategori linki
{tags}                  # Etiketler
{xfvalue_alanadi}       # Özel alan
{image-1}, {image-2}    # Resimler
[rating]...[/rating]    # Oy sistemi
[comments]...[/comments] # Yorum linki
[next-url]...[/next-url] # Sonraki haber
[prev-url]...[/prev-url] # Önceki haber
```

---

### 2. show.short.php - Haber Listeleme

**Dosya Yolu:** `engine/modules/show.short.php`

**İşlevi:** Haberlerin liste görünümünü sağlar (ana sayfa, kategori sayfaları).

**Özellikler:**
- Çoklu haber listeleme
- Kategori filtreleme
- Sayfalama desteği
- Banner entegrasyonu
- RSS beslemesi oluşturma

**Örnek Kullanım:**

```php
// URL Yapısı
https://siteadi.com/
https://siteadi.com/kategori/
https://siteadi.com/page/2/
```

**Kullanılan Template Değişkenleri:**

```html
{title}                 # Haber başlığı
{short-story}           # Kısa içerik
{date}                  # Tarih
{author}                # Yazar
{comments-num}          # Yorum sayısı
{views}                 # Görüntülenme
{category}              # Kategori
{link-category}         # Kategori linki
{full-link}             # Tam habere link
[rating]...[/rating]    # Oy sistemi
[full-link]...[/full-link] # Tam haber linki
[edit]...[/edit]        # Düzenleme linki
[add-favorites]...[/add-favorites] # Favorilere ekle
[del-favorites]...[/del-favorites] # Favorilerden sil
```

---

### 3. comments.php - Yorum Sistemi

**Dosya Yolu:** `engine/modules/comments.php`

**İşlevi:** Yorumların listelenmesi ve yönetimi.

**Özellikler:**
- Yorum listeleme
- Toplu yorum silme/birleştirme
- Yorum onaylama sistemi
- Ağaç yapısında yorumlar (tree comments)
- Yorum oylama sistemi

**Kullanılan Template Değişkenleri:**

```html
{comment-id}            # Yorum ID
{comment-author}        # Yorum yazarı
{comment-date}          # Yorum tarihi
{comment-text}          # Yorum içeriği
{comment-rating}        # Yorum oyu
[reply]...[/reply]      # Cevap linki
[edit]...[/edit]        # Düzenleme linki
[del]...[/del]          # Silme linki
```

---

### 4. addnews.php - Haber Ekleme

**Dosya Yolu:** `engine/modules/addnews.php`

**İşlevi:** Kullanıcıların haber eklemesini sağlar.

**Özellikler:**
- WYSIWYG editör desteği
- Dosya yükleme
- Özel alanlar (X-Fields)
- Kategori seçimi
- Önizleme özelliği
- SEO ayarları (meta tags)

**Kullanılan Form Alanları:**

```html
<input name="title" />           # Başlık
<textarea name="short_story"></textarea>  # Kısa içerik
<textarea name="full_story"></textarea>   # Tam içerik
<select name="category">         # Kategori
<input name="tags" />            # Etiketler
<input name="alt_name" />        # SEO URL
```

---

### 5. favorites.php - Favoriler Sistemi

**Dosya Yolu:** `engine/modules/favorites.php`

**İşlevi:** Kullanıcıların haberleri favorilere eklemesini sağlar.

**Özellikler:**
- AJAX tabanlı ekleme/çıkarma
- Kullanıcı bazlı favori listesi
- Kategori bazlı filtreleme

**AJAX Kullanımı:**

```javascript
// Favorilere ekle
$.post(dle_root + "index.php", {
    mod: "favorites",
    id: news_id,
    action: "add",
    user_hash: dle_login_hash
});

// Favorilerden sil
$.post(dle_root + "index.php", {
    mod: "favorites",
    id: news_id,
    action: "delete",
    user_hash: dle_login_hash
});
```

---

### 6. search.php - Arama Modülü

**Dosya Yolu:** `engine/modules/search.php`

**İşlevi:** Site içi arama yapar.

**Özellikler:**
- Başlık, içerik, etiket araması
- Kategori filtreleme
- Tarih aralığı filtreleme
- Gelişmiş arama seçenekleri

**URL Yapısı:**

```
https://siteadi.com/?do=search&subaction=search&story=aranacak_kelime
https://siteadi.com/?do=search&subaction=search&story=kelime&category=1
```

---

### 7. profile.php - Profil Sayfası

**Dosya Yolu:** `engine/modules/profile.php`

**İşlevi:** Kullanıcı profil bilgilerini gösterir.

**Özellikler:**
- Kullanıcı bilgileri
- İstatistikler (haber sayısı, yorum sayısı)
- Kullanıcı oylamaları
- Özel alanlar

**Template Değişkenleri:**

```html
{profile-name}          # Kullanıcı adı
{profile-email}         # E-posta
{profile-reg-date}      # Kayıt tarihi
{profile-news-num}      # Haber sayısı
{profile-comm-num}      # Yorum sayısı
{profile-signature}     # İmza
{profile-foto}          # Fotoğraf
{profile-land}          # Konum
{profile-xfields}       # Özel alanlar
```

---

### 8. register.php - Kayıt Sistemi

**Dosya Yolu:** `engine/modules/register.php`

**İşlevi:** Yeni kullanıcı kaydı.

**Özellikler:**
- CAPTCHA doğrulama
- E-posta doğrulama
- Kullanıcı adı kontrolü
- Özel kayıt alanları
- Spam koruması

---

### 9. pm.php - Özel Mesajlaşma

**Dosya Yolu:** `engine/modules/pm.php`

**İşlevi:** Kullanıcılar arası özel mesajlaşma.

**Özellikler:**
- Mesaj gönderme/alma
- Mesaj klasörleri
- Mesaj silme
- Bildirim sistemi

---

### 10. vote.php - Anket Sistemi

**Dosya Yolu:** `engine/modules/vote.php`

**İşlevi:** Anketlere oy verme.

**Özellikler:**
- Çoklu seçim desteği
- IP bazlı oy kontrolü
- Sonuçları gösterme

---

### 11. functions.php - Temel Fonksiyonlar

**Dosya Yolu:** `engine/modules/functions.php`

**İşlevi:** DLE'nin temel fonksiyonlarını içerir.

**Önemli Fonksiyonlar:**

```php
// Cookie ve Session Yönetimi
set_cookie($name, $value, $expires)     // Cookie ayarla
dle_session($sid)                       // Session başlat

// String İşlemleri
totranslit($var, $lower, $punkt)        // URL dostu string
dle_strtolower($str)                    // Küçük harf çevir
dle_strlen($str)                        // String uzunluğu
dle_substr($str, $start, $length)       // String alt dizisi

// Tarih ve Zaman
compare_days_date($news_date)           // Gün karşılaştırma
langdate($format, $stamp)               // Tarih formatlama
difflangdate($format, $stamp)           # Farklı tarih formatı

// Kategori İşlemleri
CategoryNewsSelection($selectedId)      # Kategori seçimi
get_categories($catid, $separator)      # Kategori linki
get_ID($category)                       # Kategori ID bul

// Önbellekleme
dle_cache($prefix, $query)              # Cache'den al
create_cache($name, $content, $prefix)  # Cache oluştur
clear_cache($names)                     # Cache temizle

// Veritabanı
$db->query($sql)                         # SQL sorgu
$db->super_query($sql)                   # Tek satır döndür
$db->get_row($result)                    # Satır al
$db->get_array($result)                  # Dizi al
$db->safesql($string)                    # SQL injection koruması

// Template
$tpl->load_template($template)           # Template yükle
$tpl->set($search, $replace)             # Değer değiştir
$tpl->set_block($pattern, $replace)      # Blok değiştir
$tpl->compile($name)                     # Template derle
$tpl->result['name']                     # Sonuç

// Kullanıcı
get_vars($file)                          # Cache dosyası oku
set_vars($file, $data)                   # Cache dosyası yaz

// Oy Verme
ShowRating($id, $rating, $vote_num)     # Oy gösterimi
ShowCommentsRating($id, $rating)         # Yorum oyu

// Diğer
msgbox($title, $text)                    # Mesaj kutusu
formatsize($file_size)                   # Dosya boyutu formatla
declination($matches)                    # Kelime çoğul ekleri
```

---

## Ajax Klasörü Detaylı İnceleme

### controller.php - AJAX Yönlendirici

**Dosya Yolu:** `engine/ajax/controller.php`

**İşlevi:** Tüm AJAX isteklerini yönlendirir.

**Çalışma Mantığı:**

```php
// URL: index.php?controller=ajax&mod=module_adi
$mod = $_REQUEST['mod'];  // Modül adı

if (file_exists(ENGINE_DIR . '/ajax/' . $mod . '.php')) {
    include(ENGINE_DIR . '/ajax/' . $mod . '.php');
}
```

**Güvenlik Kontrolleri:**
- User hash doğrulama
- IP kontrolü
- Ban kontrolü
- Ülke bazlı erişim

---

### 1. comments.php - AJAX Yorum İşlemleri

**Dosya Yolu:** `engine/ajax/comments.php`

**İşlevi:** Yorumları AJAX ile yükleme.

**Kullanım:**

```javascript
$.get(dle_root + "index.php", {
    controller: "ajax",
    mod: "comments",
    news_id: 123,
    cstart: 1
}, function(data) {
    $("#dle-ajax-comments").html(data.comments);
    $("#navigation").html(data.navigation);
}, "json");
```

**Dönen JSON:**

```json
{
    "navigation": "<div class='navigation'>...</div>",
    "comments": "<div id='comment-1'>...</div>"
}
```

---

### 2. addcomments.php - AJAX Yorum Ekleme

**Dosya Yolu:** `engine/ajax/addcomments.php`

**İşlevi:** AJAX ile yorum ekleme.

**Kullanım:**

```javascript
$.post(dle_root + "index.php", {
    controller: "ajax",
    mod: "addcomments",
    id: news_id,
    text: comment_text,
    name: user_name,
    email: user_email,
    user_hash: dle_login_hash
}, function(data) {
    if (data.success) {
        // Yorum başarıyla eklendi
    } else {
        // Hata mesajı
        alert(data.error);
    }
}, "json");
```

**Dönen JSON:**

```json
{
    "success": true,
    "comment": "<div id='comment-123'>...</div>"
}
```

veya hata durumunda:

```json
{
    "success": false,
    "error": "Hata mesajı"
}
```

---

### 3. favorites.php - AJAX Favori İşlemleri

**Dosya Yolu:** `engine/ajax/favorites.php`

**İşlevi:** Favorilere ekleme/çıkarma.

**Kullanım:**

```javascript
// Favorilere ekle
$.post(dle_root + "index.php", {
    controller: "ajax",
    mod: "favorites",
    id: news_id,
    action: "add",
    user_hash: dle_login_hash
}, function(data) {
    $(".add-favorites").hide();
    $(".del-favorites").show();
}, "json");

// Favorilerden sil
$.post(dle_root + "index.php", {
    controller: "ajax",
    mod: "favorites",
    id: news_id,
    action: "delete",
    user_hash: dle_login_hash
}, function(data) {
    $(".del-favorites").hide();
    $(".add-favorites").show();
}, "json");
```

---

### 4. rating.php - AJAX Oy Verme

**Dosya Yolu:** `engine/ajax/rating.php`

**İşlevi:** Haberlere oy verme.

**Kullanım:**

```javascript
function doRate(type, news_id) {
    $.post(dle_root + "index.php", {
        controller: "ajax",
        mod: "rating",
        id: news_id,
        type: type,  // 'plus' veya 'minus'
        user_hash: dle_login_hash
    }, function(data) {
        if (data.success) {
            // Oy başarıyla verildi
            updateRatingDisplay(news_id, data.rating);
        } else {
            alert(data.error);
        }
    }, "json");
}
```

---

### 5. upload.php - AJAX Dosya Yükleme

**Dosya Yolu:** `engine/ajax/upload.php`

**İşlevi:** Dosya yükleme işlemleri.

**Kullanım:**

```javascript
var formData = new FormData();
formData.append('controller', 'ajax');
formData.append('mod', 'upload');
formData.append('user_hash', dle_login_hash);
formData.append('files', fileInput.files[0]);

$.ajax({
    url: dle_root + "index.php",
    type: "POST",
    data: formData,
    processData: false,
    contentType: false,
    success: function(data) {
        // Dosya başarıyla yüklendi
        console.log(data.url);
    }
});
```

---

### 6. search.php - AJAX Arama

**Dosya Yolu:** `engine/ajax/search.php`

**İşlevi:** AJAX ile canlı arama.

**Kullanım:**

```javascript
$("#search-input").on("input", function() {
    var query = $(this).val();
    
    if (query.length > 2) {
        $.get(dle_root + "index.php", {
            controller: "ajax",
            mod: "search",
            story: query
        }, function(data) {
            $("#search-results").html(data);
        });
    }
});
```

---

### 7. profile.php - AJAX Profil İşlemleri

**Dosya Yolu:** `engine/ajax/profile.php`

**İşlevi:** Profil güncelleme işlemleri.

**Kullanım:**

```javascript
$.post(dle_root + "index.php", {
    controller: "ajax",
    mod: "profile",
    action: "update",
    fullname: fullname,
    land: land,
    signature: signature,
    user_hash: dle_login_hash
}, function(data) {
    if (data.success) {
        // Profil güncellendi
    } else {
        alert(data.error);
    }
}, "json");
```

---

### 8. registration.php - AJAX Kayıt

**Dosya Yolu:** `engine/ajax/registration.php`

**İşlevi:** AJAX ile kullanıcı kaydı.

**Kullanım:**

```javascript
$.post(dle_root + "index.php", {
    controller: "ajax",
    mod: "registration",
    login: username,
    password: password,
    email: email,
    sec_code: captcha,
    user_hash: dle_login_hash
}, function(data) {
    if (data.success) {
        // Kayıt başarılı
    } else {
        // Hata mesajlarını göster
        $.each(data.errors, function(key, value) {
            $("#" + key + "-error").text(value);
        });
    }
}, "json");
```

---

## Inc Klasörü Detaylı İnceleme (Yönetim Paneli)

### 1. addnews.php - Admin Haber Ekleme

**Dosya Yolu:** `engine/inc/addnews.php`

**İşlevi:** Yönetim panelinden haber ekleme.

**Özellikler:**
- Tüm kategori erişimi
- HTML editor desteği
- Dosya yöneticisi entegrasyonu
- SEO ayarları
- Özel alanlar
- Erişim izinleri

---

### 2. editnews.php - Admin Haber Düzenleme

**Dosya Yolu:** `engine/inc/editnews.php`

**İşlevi:** Yönetim panelinden haber düzenleme.

**Özellikler:**
- Toplu haber düzenleme
- Haber onaylama
- Sabitleme
- Kategori değiştirme

---

### 3. xfields.php - Özel Alan Yönetimi

**Dosya Yolu:** `engine/inc/xfields.php`

**İşlevi:** Özel alan (X-Fields) yönetimi.

**Özellikler:**
- Alan ekleme/düzenleme/silme
- Alan tipleri (text, textarea, select, checkbox, etc.)
- Kategori bazlı alan atama
- Zorunlu alan ayarı

**Örnek Özel Alan Tanımı:**

```php
// Alan yapısı
array(
    "name" => "film_adi",
    "descr" => "Film Adı",
    "type" => "text",
    "default" => "",
    "required" => "1",
    "admin" => "1",
    "edit" => "1",
    "search" => "1",
    "filter" => "0"
)
```

**Template Kullanımı:**

```html
{xfvalue_film_adi}      # Alan değeri
{xflabel_film_adi}      # Alan etiketi
[xfvalue_film_adi]...[/xfvalue_film_adi]  # Koşullu gösterim
```

---

### 4. categories.php - Kategori Yönetimi

**Dosya Yolu:** `engine/inc/categories.php`

**İşlevi:** Kategori oluşturma ve yönetimi.

**Özellikler:**
- Ağaç yapısında kategoriler
- SEO ayarları
- Template atama
- Erişim izinleri
- Meta etiketleri

---

### 5. templates.php - Template Yönetimi

**Dosya Yolu:** `engine/inc/templates.php`

**İşlevi:** Template dosyalarını yönetme.

**Özellikler:**
- Template düzenleme
- Yedekleme
- Yeni template oluşturma

---

### 6. options.php - Sistem Ayarları

**Dosya Yolu:** `engine/inc/options.php`

**İşlevi:** Sistem genel ayarları.

**Ayar Grupları:**
- Genel ayarlar
- SEO ayarları
- Yorum ayarları
- Upload ayarları
- Cache ayarları
- Güvenlik ayarları

---

## Temel Fonksiyonlar ve Kullanımları

### Veritabanı İşlemleri

```php
global $db;

// Basit sorgu
$db->query("SELECT * FROM " . PREFIX . "_post WHERE approve='1'");

// Tek satır çekme
$row = $db->super_query("SELECT COUNT(*) as count FROM " . PREFIX . "_post");
echo $row['count'];

// Çoklu satır çekme
$db->query("SELECT id, title FROM " . PREFIX . "_post LIMIT 10");
while ($row = $db->get_row()) {
    echo $row['title'];
}

// Güvenli sorgu (SQL Injection koruması)
$title = $db->safesql($_POST['title']);
$db->query("INSERT INTO " . PREFIX . "_post SET title='{$title}'");

// Güncelleme
$db->query("UPDATE " . PREFIX . "_post SET news_read=news_read+1 WHERE id='{$id}'");

// Silme
$db->query("DELETE FROM " . PREFIX . "_post WHERE id='{$id}'");
```

### Önbellekleme (Cache) İşlemleri

```php
// Cache'den veri çekme
$data = dle_cache("prefix", "cache_name");

// Cache oluşturma
create_cache("cache_name", "içerik", "prefix");

// Cache temizleme
clear_cache(array("news_", "full_", "comm_"));

// Örnek kullanım
$cache_name = "my_cache_" . $id;
$cached_data = dle_cache("my_prefix", $cache_name);

if (!$cached_data) {
    // Cache yok, veritabanından çek
    $data = $db->super_query("SELECT * FROM table WHERE id='{$id}'");
    create_cache($cache_name, json_encode($data), "my_prefix");
} else {
    $data = json_decode($cached_data, true);
}
```

### Template İşlemleri

```php
global $tpl;

// Template yükleme
$tpl->load_template("template_adi.tpl");

// Değer değiştirme
$tpl->set("{degisken}", "değer");
$tpl->set("", array("{a}", "{b}"), array("1", "2"));

// Blok değiştirme
$tpl->set("[blok]", "");
$tpl->set("[/blok]", "");
$tpl->set_block("'\\[blok\\](.*?)\\[/blok\\]'si", "içerik");

// Regex ile değiştirme
$tpl->copy_template = preg_replace("#\{pattern\}#i", "replacement", $tpl->copy_template);

// Template derleme
$tpl->compile("content");

// Sonucu gösterme
echo $tpl->result["content"];

// Template temizleme
$tpl->clear();
```

### Cookie ve Session İşlemleri

```php
// Cookie ayarlama
set_cookie("cookie_adi", "değer", 30);  // 30 gün

// Cookie silme
set_cookie("cookie_adi", "", -1);

// Cookie okuma
$deger = $_COOKIE["cookie_adi"];

// Session başlatma
dle_session();

// Session değişkeni
$_SESSION["degisken"] = "değer";
```

### Kullanıcı ve Yetki Kontrolleri

```php
global $is_logged, $member_id, $user_group;

// Giriş kontrolü
if ($is_logged) {
    // Kullanıcı giriş yapmış
}

// Kullanıcı bilgileri
echo $member_id['name'];      // Kullanıcı adı
echo $member_id['user_group']; // Grup ID
echo $member_id['email'];     // E-posta

// Grup yetkisi kontrolü
if ($user_group[$member_id['user_group']]['allow_edit']) {
    // Düzenleme yetkisi var
}

// Admin kontrolü
if ($member_id['user_group'] < 5) {
    // Admin veya moderatör
}
```

### Kategori İşlemleri

```php
global $cat_info;

// Tüm kategorileri listeleme
foreach ($cat_info as $id => $cat) {
    echo $cat['name'];
    echo $cat['alt_name'];
}

// Kategori seçimi (dropdown)
echo CategoryNewsSelection($selected_id);

// Kategori linki oluşturma
$link = get_categories($catid, ", ");

// Kategori ID bulma
$id = get_ID("kategori-url");
```

### Tarih ve Zaman İşlemleri

```php
global $_TIME;

// Mevcut zaman
$timestamp = $_TIME;

// Tarih formatlama
echo langdate("d.m.Y H:i", $timestamp);

// Gün karşılaştırma
$days = compare_days_date($news_date);  // 0: bugün, 1: dün, 2+: önceki günler

// Özel format
$tpl->copy_template = preg_replace_callback("#\{date=(.+?)\}#i", "formdate", $tpl->copy_template);
```

### URL Oluşturma

```php
// SEO URL oluşturma
$url = DLEUrl::BuildUrl('showfull', [
    'category' => get_url($category_id),
    'year' => date('Y', $timestamp),
    'month' => date('m', $timestamp),
    'day' => date('d', $timestamp),
    'news_name' => $alt_name,
    'newsid' => $id
]);

// Basit URL'ler
DLEUrl::BuildUrl('category', ['category' => 'kategori-url']);
DLEUrl::BuildUrl('user', ['user' => 'kullanici-adi']);
DLEUrl::BuildUrl('tags', ['tag' => 'etiket']);
```

### Güvenlik

```php
// CSRF token kontrolü
if ($_POST['dle_allow_hash'] != $dle_login_hash) {
    die("Hacking attempt!");
}

// XSS koruması
$clean = htmlspecialchars($input, ENT_QUOTES, 'UTF-8');

// Translit (URL dostu)
$url = totranslit($string, true, true);

// IP adresi
$ip = get_ip();

// Flood kontrolü
if (flooder($ip)) {
    die("Çok hızlı işlem!");
}
```

---

## Veritabanı Yapısı

### Ana Tablolar

#### dle_post (Haberler)

```sql
CREATE TABLE `dle_post` (
    `id` INT(11) NOT NULL AUTO_INCREMENT,
    `date` DATETIME DEFAULT NULL,
    `autor` CHAR(40) DEFAULT NULL,
    `short_story` TEXT NOT NULL,
    `full_story` TEXT,
    `xfields` TEXT,
    `title` VARCHAR(250) DEFAULT NULL,
    `descr` VARCHAR(250) DEFAULT NULL,
    `keywords` TEXT,
    `category` VARCHAR(255) DEFAULT NULL,
    `alt_name` VARCHAR(250) DEFAULT NULL,
    `comm_num` INT(11) DEFAULT '0',
    `allow_comm` TINYINT(1) DEFAULT '1',
    `fixed` TINYINT(1) DEFAULT '0',
    `tags` TEXT,
    `news_read` INT(11) DEFAULT '0',
    `approve` TINYINT(1) DEFAULT '1',
    PRIMARY KEY (`id`),
    KEY `date` (`date`),
    KEY `alt_name` (`alt_name`),
    KEY `approve` (`approve`)
);
```

#### dle_post_extras (Haber Ekstra)

```sql
CREATE TABLE `dle_post_extras` (
    `news_id` INT(11) NOT NULL,
    `user_id` INT(11) DEFAULT NULL,
    `rating` INT(11) DEFAULT '0',
    `vote_num` INT(11) DEFAULT '0',
    `access` TEXT,
    `related_ids` TEXT,
    `votes` INT(11) DEFAULT '0',
    PRIMARY KEY (`news_id`)
);
```

#### dle_comments (Yorumlar)

```sql
CREATE TABLE `dle_comments` (
    `id` INT(11) NOT NULL AUTO_INCREMENT,
    `post_id` INT(11) DEFAULT NULL,
    `user_id` INT(11) DEFAULT NULL,
    `date` DATETIME DEFAULT NULL,
    `autor` CHAR(40) DEFAULT NULL,
    `email` VARCHAR(255) DEFAULT NULL,
    `text` TEXT NOT NULL,
    `ip` VARCHAR(50) DEFAULT NULL,
    `is_register` TINYINT(1) DEFAULT '0',
    `rating` INT(11) DEFAULT '0',
    `vote_num` INT(11) DEFAULT '0',
    `parent` INT(11) DEFAULT '0',
    `approve` TINYINT(1) DEFAULT '1',
    PRIMARY KEY (`id`),
    KEY `post_id` (`post_id`),
    KEY `approve` (`approve`)
);
```

#### dle_users (Kullanıcılar)

```sql
CREATE TABLE `dle_users` (
    `user_id` INT(11) NOT NULL AUTO_INCREMENT,
    `password` CHAR(255) DEFAULT NULL,
    `name` CHAR(40) NOT NULL,
    `email` CHAR(255) NOT NULL,
    `news_num` INT(11) DEFAULT '0',
    `comm_num` INT(11) DEFAULT '0',
    `user_group` SMALLINT(5) DEFAULT '5',
    `reg_date` INT(11) DEFAULT NULL,
    `lastdate` INT(11) DEFAULT NULL,
    `signature` TEXT,
    `foto` VARCHAR(255) DEFAULT NULL,
    `fullname` VARCHAR(255) DEFAULT NULL,
    `land` VARCHAR(255) DEFAULT NULL,
    `xfields` TEXT,
    `banned` TINYINT(1) DEFAULT '0',
    PRIMARY KEY (`user_id`),
    UNIQUE KEY `name` (`name`),
    UNIQUE KEY `email` (`email`)
);
```

#### dle_category (Kategoriler)

```sql
CREATE TABLE `dle_category` (
    `id` INT(11) NOT NULL AUTO_INCREMENT,
    `parentid` INT(11) DEFAULT '0',
    `name` CHAR(255) DEFAULT NULL,
    `alt_name` CHAR(255) DEFAULT NULL,
    `icon` VARCHAR(255) DEFAULT NULL,
    `description` TEXT,
    `keywords` TEXT,
    `template` VARCHAR(255) DEFAULT NULL,
    `short_tpl` VARCHAR(255) DEFAULT NULL,
    `full_tpl` VARCHAR(255) DEFAULT NULL,
    `disable_comments` TINYINT(1) DEFAULT '0',
    PRIMARY KEY (`id`),
    KEY `alt_name` (`alt_name`)
);
```

#### dle_usergroups (Kullanıcı Grupları)

```sql
CREATE TABLE `dle_usergroups` (
    `id` SMALLINT(5) NOT NULL AUTO_INCREMENT,
    `group_name` CHAR(255) DEFAULT NULL,
    `allow_cats` TEXT,
    `allow_addc` TINYINT(1) DEFAULT '1',
    `allow_rating` TINYINT(1) DEFAULT '1',
    `allow_edit` TINYINT(1) DEFAULT '0',
    `allow_all_edit` TINYINT(1) DEFAULT '0',
    `del_allc` TINYINT(1) DEFAULT '0',
    `moderation` TINYINT(1) DEFAULT '0',
    PRIMARY KEY (`id`)
);
```

---

## Örnek Eklenti Geliştirme

### Basit Bir Modül Oluşturma

**Dosya:** `engine/modules/my_module.php`

```php
<?php
/*
=====================================================
 My Module - Örnek Eklenti
=====================================================
*/

if (!defined('DATALIFEENGINE')) {
    header("HTTP/1.1 403 Forbidden");
    header('Location: ../../');
    die("Hacking attempt!");
}

// Veritabanından veri çekme
$results = $db->query("SELECT id, title, date FROM " . PREFIX . "_post WHERE approve='1' ORDER BY date DESC LIMIT 10");

$my_data = array();
while ($row = $db->get_row($results)) {
    $row['date'] = strtotime($row['date']);
    $row['link'] = DLEUrl::BuildUrl('showfull', [
        'category' => get_url($row['category']),
        'year' => date('Y', $row['date']),
        'month' => date('m', $row['date']),
        'day' => date('d', $row['date']),
        'news_name' => $row['alt_name'],
        'newsid' => $row['id']
    ]);
    $my_data[] = $row;
}

// Template yükleme
$tpl->load_template("my_module.tpl");

// Verileri template'e aktarma
foreach ($my_data as $item) {
    $tpl->set("{title}", $item['title']);
    $tpl->set("{link}", $item['link']);
    $tpl->set("{date}", langdate("d.m.Y", $item['date']));
    $tpl->compile("content");
}

// Sonucu gösterme
echo $tpl->result["content"];
$tpl->clear();
?>
```

### AJAX Modülü Oluşturma

**Dosya:** `engine/ajax/my_ajax.php`

```php
<?php
if (!defined('DATALIFEENGINE')) {
    header("HTTP/1.1 403 Forbidden");
    header('Location: ../../');
    die("Hacking attempt!");
}

// Güvenlik kontrolü
if ($_POST['user_hash'] !== $dle_login_hash) {
    echo json_encode(array("success" => false, "error" => "Güvenlik hatası"));
    die();
}

// İşlem
$action = $_POST['action'];
$id = intval($_POST['id']);

if ($action == "save") {
    $data = $db->safesql($_POST['data']);
    $db->query("UPDATE " . PREFIX . "_post SET xfields='{$data}' WHERE id='{$id}'");
    
    echo json_encode(array("success" => true));
} else {
    echo json_encode(array("success" => false, "error" => "Geçersiz işlem"));
}
?>
```

### JavaScript ile AJAX Kullanımı

```javascript
function saveData(id, data) {
    $.post(dle_root + "index.php", {
        controller: "ajax",
        mod: "my_ajax",
        action: "save",
        id: id,
        data: data,
        user_hash: dle_login_hash
    }, function(response) {
        if (response.success) {
            alert("Başarıyla kaydedildi!");
        } else {
            alert("Hata: " + response.error);
        }
    }, "json");
}
```

### Özel Template Oluşturma

**Dosya:** `templates/default/my_module.tpl`

```html
<div class="my-module">
    <h3>Özel Modül</h3>
    <ul>
        {content}
    </ul>
</div>
```

**Template İçeriği:**

```html
<li>
    <a href="{link}">{title}</a>
    <span class="date">{date}</span>
</li>
```

---

## Güvenlik En İyi Uygulamaları

### 1. SQL Injection Koruması

```php
// Kötü örnek
$id = $_GET['id'];
$db->query("SELECT * FROM table WHERE id={$id}");

// İyi örnek
$id = intval($_GET['id']);
$db->query("SELECT * FROM table WHERE id={$id}");

// Metin için
$title = $db->safesql($_POST['title']);
```

### 2. XSS Koruması

```php
// Çıktı temizleme
echo htmlspecialchars($user_input, ENT_QUOTES, 'UTF-8');

// Template içinde otomatik temizleme
$tpl->set("{title}", htmlspecialchars($title, ENT_QUOTES, 'UTF-8'));
```

### 3. CSRF Koruması

```php
// Token oluşturma
$dle_login_hash = md5(uniqid($config['http_home_url'], true));

// Formda token
<input type="hidden" name="dle_allow_hash" value="<?php echo $dle_login_hash; ?>">

// Kontrol
if ($_POST['dle_allow_hash'] !== $dle_login_hash) {
    die("CSRF hatası!");
}
```

### 4. Dosya Yükleme Güvenliği

```php
$allowed_ext = array('jpg', 'jpeg', 'png', 'gif');
$ext = strtolower(pathinfo($_FILES['file']['name'], PATHINFO_EXTENSION));

if (!in_array($ext, $allowed_ext)) {
    die("Geçersiz dosya tipi!");
}

// Dosya adını güvenli hale getir
$filename = totranslit(pathinfo($_FILES['file']['name'], PATHINFO_FILENAME), true, false) . '.' . $ext;
```

---

## Performans Optimizasyonu

### 1. Cache Kullanımı

```php
// Cache ile
$cache_data = dle_cache("news", "home_page");
if (!$cache_data) {
    $data = get_expensive_data();
    create_cache("home_page", json_encode($data), "news");
} else {
    $data = json_decode($cache_data, true);
}
```

### 2. Veritabanı Optimizasyonu

```php
// Gerekli sütunları seçme
$db->query("SELECT id, title, date FROM " . PREFIX . "_post");

// JOIN kullanımı
$db->query("SELECT p.*, u.name as user_name 
            FROM " . PREFIX . "_post p 
            LEFT JOIN " . USERPREFIX . "_users u ON p.user_id = u.user_id");

// LIMIT kullanımı
$db->query("SELECT * FROM " . PREFIX . "_post LIMIT 10");
```

### 3. Template Optimizasyonu

```php
// Kötü örnek - Her satırda ayrı compile
foreach ($data as $row) {
    $tpl->set("{title}", $row['title']);
    $tpl->compile("content");
}

// İyi örnek - Tek compile
foreach ($data as $row) {
    $tpl->set("{title}", $row['title']);
    $tpl->parse("content");
}
$tpl->compile("content");
```

---

## Sorun Giderme

### Yaygın Hatalar

**1. "Hacking attempt!" Hatası**

```php
// DLE sabiti tanımlanmamış
if (!defined('DATALIFEENGINE')) {
    die("Hacking attempt!");
}

// Çözüm: Dosyanın başına ekleyin
<?php
if (!defined('DATALIFEENGINE')) {
    header("HTTP/1.1 403 Forbidden");
    header('Location: ../../');
    die("Hacking attempt!");
}
```

**2. Veritabanı Bağlantı Hatası**

```php
// engine/data/dbconfig.php dosyasını kontrol edin
$DB_Host = "localhost";
$DB_Name = "database_adi";
$DB_User = "kullanici_adi";
$DB_Pass = "sifre";
```

**3. Cache Hatası**

```bash
# Cache klasörü izinlerini kontrol edin
chmod -R 777 engine/cache/
```

**4. Template Bulunamadı Hatası**

```php
// Template dosyasının varlığını kontrol edin
if (file_exists(TEMPLATE_DIR . "/template_adi.tpl")) {
    $tpl->load_template("template_adi.tpl");
}
```

---

## Faydalı Kaynaklar

- **Resmi DLE Sitesi:** https://dle-news.ru/
- **DLE Dokümantasyon:** https://dle-news.ru/documentation/
- **DLE Forum:** https://dle-news.ru/forum/
- **DLE Türkçe Destek:** https://dle.net.tr/

---

## Sonuç

Bu dokümantasyon, DLE (DataLife Engine) için eklenti geliştirme konusunda kapsamlı bir rehber sunmaktadır. DLE'nin modüler yapısı, güçlü template sistemi ve geniş fonksiyon kütüphanesi sayesinde hızlı ve güvenli eklentiler geliştirebilirsiniz.

**Önemli Notlar:**

1. Her zaman güvenlik önlemlerini alın (SQL injection, XSS, CSRF)
2. Cache sistemini etkin kullanın
3. Template dosyalarını düzenlerken yedek alın
4. Eklentilerinizi test ortamında deneyin
5. DLE sürüm uyumluluğuna dikkat edin

**İyi Kodlamalar!**
