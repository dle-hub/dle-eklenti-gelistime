# 📘 DataLife Engine (DLE) 19.1+ Eklenti Geliştirme Ansiklopedisi (50 Madde)

Bu rehber, DLE 19.x mimarisini atomlarına ayırarak, bir eklentinin fikir aşamasından yayına alınmasına kadar gereken tüm teknik detayları ve kod iskeletlerini kapsar.

---

## 🏗️ BÖLÜM 1: TEMEL MİMARİ VE DOSYA SİSTEMİ

### 1. DLE 19.x Vizyonu
DLE 19+, modülerlik ve güvenlik üzerine inşa edilmiştir. Eklentiler artık çekirdek dosyaları fiziksel olarak değiştirmez (VFS).

### 2. VFS (Virtual File System) Mantığı
Sistem, `plugin.xml` içindeki talimatları okuyarak dosyaların bellekteki sanal kopyalarını değiştirir. Fiziksel dosyanız bozulsa bile eklenti pasif edildiğinde sistem orijinal haline döner.

### 3. Kritik Klasör Yapısı
*   `engine/inc/`: Admin panel modülleri.
*   `engine/modules/`: Site frontend modülleri.
*   `engine/ajax/`: Arka plan işleyiciler.

### 4. Direct Access Koruması
Tüm PHP dosyalarınızın en başında şu satır ZORUNLUDUR:
```php
if( !defined( 'DATALIFEENGINE' ) ) die( "Hacking attempt!" );
```

### 5. Logged In Koruması
Sadece yetkili adminlerin erişmesini istediğiniz dosyalar için:
```php
if( !defined( 'LOGGED_IN' ) ) die( "Hacking attempt!" );
```

---

## 📜 BÖLÜM 2: PLUGIN.XML USTALIĞI

### 6. XML Metadata Tanımlama
Eklenti adı, sürümü ve benzersiz ID'si (`<id>`) doğru tanımlanmalıdır.

### 7. {prefix} ve {charset} Etiketleri
SQL sorgularında statik `dle_` kullanmayın. Daima `{prefix}` kullanın. Karakter seti için `{charset}` kullanın.

### 8. <mysqlinstall> Prosedürü
İlk kurulumda tabloları oluşturur. `IF NOT EXISTS` kullanmak çakışmaları önler.

### 9. <mysqldelete> ve Temizlik
Eklenti silindiğinde geride çöp bırakmamak için `DROP TABLE` komutlarını içermelidir.

### 10. <file> Hedefleme
Modifikasyon yapılacak dosya yolu tam olarak yazılmalıdır (Örn: `engine/engine.php`).

### 11. <operation action="create">
DLE'de olmayan bir dosyayı (Örn: Modül linki) sıfırdan oluşturmak için en güçlü araçtır.

### 12. searchcode ve replacecode Mantığı
Kodun üstüne (`before`), altına (`after`) ekler veya değiştirir (`replace`). Regex olmayan düz metinler daha stabildir.

---

## 🎨 BÖLÜM 3: NATIVE ADMIN UI STANDARTLARI

### 13. echoheader() Kullanımı
Sayfa başlığını, ikonunu ve açıklamasını sisteme tanıtır. Breadcrumb'ları otomatik hazırlar.

### 14. Layout Flow (Akış Hiyerarşisi)
DLE sayfaları yukarıdan aşağıya: Header -> Navbar -> Alert -> Table/Panel -> Footer silsilesini takip eder.

### 15. panel-default vs panel-flat
*   `panel-default`: Kenarlıklı, bölmeli içerikler için.
*   `panel-flat`: Ayarların listelendiği, tablosal görünümler için.

### 16. Dinamik Navbar (navbar-collapse collapse)
Mobilde hamburger menüye dönüşen tek standart yapıdır. `navbar-xs` ile zarif görünüm sağlar.

### 17. Sağa Dayalı Araçlar (navbar-right)
Arama kutuları ve "Yeni Ekle" butonlarını sağ tarafta toplamak için kullanılır.

### 18. Semantik Renkler (Material Palette)
*   `bg-primary-600`: Ayarlar.
*   `bg-teal`: Kaydet/Başarı.
*   `bg-danger-600`: Silme/Hata.

### 19. Form-Horizontal Düzeni
Label ve giriş alanlarının 2:10 oranında (`col-md-2`, `col-md-10`) hizalandığı profesyonel düzen.

### 20. Tooltips ve Popovers (help-button)
Kullanıcıya ayar hakkında bilgi vermek için `data-rel="popover"` sınıflı soru işareti ikonları.

### 21. Native Checkboxes (icheck)
DLE'nin standart kare checkboxlarını aktif etmek için sadece `class="icheck"` eklemeniz yeterlidir.

### 22. IOS Tarzı Switcher (switch)
Açma/kapatma (Toggle) işlemleri için `class="switch"` kullanılır.

---

## ⚡ BÖLÜM 4: JAVASCRIPT & AJAX (JS API)

### 23. DLEPush: Toast Bildirimleri
`DLEPush.success`, `DLEPush.error`, `DLEPush.info` ve `DLEPush.warning` ile modern bildirimler.

### 24. DLEPush Parametreleri
`DLEPush.error(mesaj, baslik, sure)` formülünde süre milisaniye cinsindendir.

### 25. ShowLoading() ve HideLoading()
AJAX isteği başladığında ekranı karartıp "Bekleyin" spinner'ı çıkaran hayati fonksiyonlar.

### 26. dle_login_hash (CSRF Koruması)
Her AJAX POST isteğinde `user_hash` parametresi olarak bu değişkeni göndermek ZORUNLUDUR.

### 27. tinyMCE.triggerSave() Senkronu
Eğer formda editör varsa, veriyi okumadan önce editor içeriğini textarea'ya aktarır.

### 28. DLEconfirm Onay Pencereleri
Silme veya kritik işlemlerden önce `DLEconfirm` ile kullanıcı onayı alınması native bir kuraldır.

### 29. DLEconfirmDelete Özelleşmiş Sınıfı
Sadece silme işlemleri için optimize edilmiş, kırmızı temalı onay penceresi.

### 30. AJAX .fail() Hata Yönetimi
İnternet koptuğunda veya sunucu hata verdiğinde `jqXHR` üzerinden hatayı `DLEPush.error` ile basma pratiği.

---

## 🌐 BÖLÜM 5: FRONTEND (MODULES) VE TEMPLATE

### 31. {include} Etiketi ve Güvenlik
`{include file="engine/modules/modul.php"}` kullanımı en güvenli dahil etme yöntemidir.

### 32. Parametre Gönderimi (Query String)
`modul.php?act=view&id=5` şeklinde gönderilen veriler PHP'de direkt değişken (`$act`, `$id`) olur.

### 33. {news-id} Verisini Modüle Taşımak
`fullstory.tpl` içinde `{include file="...php?id={news-id}"}` ile dinamik haber verisi taşınır.

### 34. CHMOD 777 Güvenlik Yasağı
Modül PHP dosyalarının bulunduğu klasörde yazma yetkisi varsa DLE o dosyayı "Hacking Attempt" diyerek çalıştırmaz.

### 35. [aviable] ile Özel Sayfalar
`main.tpl`'de sadece belirli bölümlerde (Örn: `?do=faq`) modülü gösterme mantığı.

### 36. $tpl->load_template() ve Derleme
Bir `.tpl` dosyasını belleğe yükleyip, içindeki etiketleri doldurup `compile` ile basma süreci.

---

## 💾 BÖLÜM 6: ÇEKİRDEK SINIFLAR VE GLOBALLER

### 37. $db Sınıfı ve super_query
Tek satır sonuç dönen (Örn: ID ile üye seçme) sorgular için `super_query` performansı artırır.

### 38. $db->safesql() Koruması
SQL Injection yememek için kullanıcıdan gelen her veriyi bu süzgeçten geçirin.

### 39. $member_id Dizisi
Giriş yapan kullanıcının adından, şifresine, e-postasından grubuna kadar tüm veriler bu dizidedir.

### 40. $config Dizisi
Daima `$config['site_url']` veya `$config['version']` gibi verilere buradan erişin.

### 41. $user_group Yetki Haritası
Belirli bir işlemin belirli bir grup tarafından yapılıp yapılamayacağını kontrol eder.

### 42. $lang Dil Paketi Dizisi
Metinlerinizi asla PHP'ye hard-code yazmayın. `$lang['mesaj_adi']` şeklinde dil dosyasından çekin.

### 43. $cat_info Kategori Verisi
Sitedeki tüm kategorilerin isim, URL ve ID hiyerarşisini tutan devasa dizi.

---

## 🔌 BÖLÜM 7: RESMİ DLE API (engine/api)

### 44. API Sürüm Uyumluluğu Garantisi
API üzerinden yapılan işlemler, sistem güncellense bile PHP kodunuzun bozulmamasını sağlar.

### 45. take_user_by_id() ile Veri Çekme
Kullanıcı verilerini veritabanına sorgu atmadan, API'nin güvenli metodlarıyla çekme.

### 46. change_user_group() ile Yetki Yönetimi
Kullanıcı gruplarını, veritabanı yollarını düşünmeden tek satırla değiştirme.

### 47. send_pm_to_user() (Özel Mesaj API)
Standartlara uygun, bildirimli özel mesaj gönderme motoru.

### 48. load_table() ile Gelişmiş Filtreleme
Tablolardan veri çekerken sayfalama, sıralama ve WHERE koşullarını tek fonksiyonda toplama.

### 49. save_to_cache() ve Performans
Ağır sorguları önbelleğe alarak sitenin hızını (ve sunucu yükünü) optimize etme.

### 50. DLEPlugins::Check() (ALTIN KURAL)
Bir dosyayı `include` veya `require` edecekseniz, VFS'nin onu tanıması için bu fonksiyona sokmak ZORUNLUDUR.

---
🚀 **Tebrikler!** Bu 50 maddeyi hazmettiyseniz, artık DataLife Engine için profesyonel eklentiler geliştirmeye hazırsınız.
