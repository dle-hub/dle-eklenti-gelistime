# DLE (DataLife Engine) Eklenti Geliştirme Dokümantasyonu

## İçindekiler

1. [Genel Bakış](#genel-bakış)
2. [Frontend Yapısı](#frontend-yapısı)
3. [HTML Etiketleri ve Kullanımları](#html-etiketleri-ve-kullanımları)
4. [Yönetim Paneli (Admin UI) Standartları](#yönetim-paneli-admin-ui-standartları)
5. [Klasör Yapısı ve İşlevleri](#klasör-yapısı-ve-işlevleri)
6. [Modules Klasörü Detaylı İnceleme](#modules-klasörü-detaylı-inceleme)
7. [Ajax Klasörü Detaylı İnceleme](#ajax-klasörü-detaylı-inceleme)
8. [Inc Klasörü Detaylı İnceleme](#inc-klasörü-detaylı-inceleme)
9. [Temel Fonksiyonlar ve Kullanımları](#temel-fonksiyonlar-ve-kullanımları)
10. [Veritabanı Yapısı](#veritabanı-yapısı)
11. [Örnek Eklenti Geliştirme](#örnek-eklenti-geliştirme)

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

```
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

```
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

```
{date=Y-m-d}              # Özel tarih formatı
{image-1}, {image-2}      # İçerikteki resimler
{xfvalue_alanisimi}       # Özel alan (X-Fields) değeri
{custompages_pageid}      # Özel sayfa içeriği
```

### JavaScript Yapısı

DLE, jQuery kütüphanesini temel alır.

**ÖNEMLI:** DLE'de AJAX istekleri her zaman `index.php?controller=ajax&mod=modul_adi` formatında yapılır!

```javascript
// Temel DLE JavaScript Fonksiyonları
doRate('plus', news_id)        // Haber oylama
doRate('minus', news_id)       // Haber olumsuz oylama
doCommentsRate('1', comment_id) // Yorum oylama
dle_news_delete(news_id)       // Haber silme (admin)
ShowProfile(username, url)     // Profil gösterimi
save_last_viewed(news_id)      // Son görüntülenenleri kaydet

// AJAX İstek Formatı
$.post(dle_root + "index.php?controller=ajax&mod=modul_adi", {
    user_hash: dle_login_hash,
    // diğer parametreler
}, function(data) {
    // callback
}, "json");
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

## HTML Etiketleri ve Kullanımları

### 1. Haber Listeleme (Short Story) HTML Yapısı

**Kullanılan HTML Etiketleri:**

```html
<!-- Ana Haber Konteyneri -->
<div class="news-item" data-news-id="{news-id}">
    
    <!-- Kategori -->
    <div class="news-category">
        [has-category]
        <a href="{link-category}" class="category-link">{category}</a>
        [/has-category]
    </div>
    
    <!-- Resim -->
    <div class="news-image">
        [image-1]
        <a href="{full-link}">
            <img src="{image-1}" alt="{title}" title="{title}" />
        </a>
        [/image-1]
        [not-image-1]
        <img src="{THEME}/dleimages/no_image.jpg" alt="{title}" />
        [/not-image-1]
    </div>
    
    <!-- Başlık -->
    <h2 class="news-title">
        <a href="{full-link}">{title}</a>
    </h2>
    
    <!-- Meta Bilgiler -->
    <div class="news-meta">
        <span class="news-date">
            <i class="icon-calendar"></i>
            <time datetime="{date=Y-m-d}">{date=d.m.Y H:i}</time>
        </span>
        
        <span class="news-author">
            <i class="icon-user"></i>
            [profile]
            <a href="{author-url}" onclick="ShowProfile('{login}', '{author-url}'); return false;">{author}</a>
            [/profile]
        </span>
        
        <span class="news-views">
            <i class="icon-eye"></i>
            {views} Görüntülenme
        </span>
        
        <span class="news-comments">
            <i class="icon-comment"></i>
            [comments]
            <a href="{full-link}#comment">{comments-num} Yorum</a>
            [/comments]
            [not-comments]Yorum Yok[/not-comments]
        </span>
    </div>
    
    <!-- Kısa İçerik -->
    <div class="news-short">
        {short-story}
    </div>
    
    <!-- Özel Alanlar (X-Fields) -->
    <div class="news-xfields">
        [xfvalue_yonetmen]
        <div class="director">
            <label>Yönetmen:</label>
            <span>{xfvalue_yonetmen}</span>
        </div>
        [/xfvalue_yonetmen]
        
        [xfvalue_oyuncular]
        <div class="actors">
            <label>Oyuncular:</label>
            <span>{xfvalue_oyuncular}</span>
        </div>
        [/xfvalue_oyuncular]
        
        [xfvalue_yapim_yili]
        <div class="year">
            <label>Yapım Yılı:</label>
            <span>{xfvalue_yapim_yili}</span>
        </div>
        [/xfvalue_yapim_yili]
        
        [xfvalue_imdb]
        <div class="imdb">
            <label>IMDB:</label>
            <span class="imdb-score">{xfvalue_imdb}</span>
        </div>
        [/xfvalue_imdb]
    </div>
    
    <!-- Oy Verme Sistemi -->
    <div class="news-rating">
        [rating]
        <div class="rating-container">
            {rating}
            <span class="rating-score">{ratingscore}</span>
        </div>
        [rating-plus]
        <button class="btn-rating-plus" onclick="doRate('plus', {news-id}); return false;">
            <i class="icon-thumbs-up"></i> Beğen
        </button>
        [/rating-plus]
        [rating-minus]
        <button class="btn-rating-minus" onclick="doRate('minus', {news-id}); return false;">
            <i class="icon-thumbs-down"></i> Beğenmedim
        </button>
        [/rating-minus]
        [/rating]
    </div>
    
    <!-- Favori Butonu -->
    <div class="news-favorites">
        [is-logged]
        [add-favorites]
        <button class="btn-add-favorite" onclick="addToFavorites({news-id}); return false;">
            <i class="icon-heart"></i> Favorilere Ekle
        </button>
        [/add-favorites]
        [del-favorites]
        <button class="btn-del-favorite" onclick="delFromFavorites({news-id}); return false;">
            <i class="icon-heart-broken"></i> Favorilerden Sil
        </button>
        [/del-favorites]
        [/is-logged]
    </div>
    
    <!-- Tam Haber Linki -->
    <div class="news-footer">
        [full-link]
        <a href="{full-link}" class="btn-read-more">
            <i class="icon-arrow-right"></i> Devamını Oku
        </a>
        [/full-link]
    </div>
    
    <!-- Düzenleme Linki (Admin) -->
    [edit]
    <div class="news-admin">
        {edit}
        [del]
        <button class="btn-delete" onclick="dle_news_delete({news-id}); return false;">
            Sil
        </button>
        [/del]
    </div>
    [/edit]
    
    <!-- Şikayet Butonu -->
    [complaint]
    <button class="btn-complaint" onclick="AddComplaint({news-id}, 'news')">
        <i class="icon-flag"></i> Şikayet Et
    </button>
    [/complaint]
    
</div>
```

---

### 2. Tam Haber Sayfası (Full Story) HTML Yapısı

```html
<!-- Ana İçerik -->
<article class="full-news" itemscope itemtype="http://schema.org/Article">
    
    <!-- Başlık -->
    <header class="news-header">
        <h1 class="news-title" itemprop="headline">{title}</h1>
        
        <!-- Kategori -->
        <div class="news-categories">
            [has-category]
            <span class="category-label">Kategori:</span>
            {link-category}
            [/has-category]
        </div>
        
        <!-- Meta -->
        <div class="news-meta">
            <meta itemprop="datePublished" content="{date=Y-m-d\TH:i:s}" />
            <meta itemprop="dateModified" content="{edit-date=Y-m-d\TH:i:s}" />
            
            <time datetime="{date=Y-m-d}" itemprop="dateCreated">{date=d.m.Y H:i}</time>
            
            <span class="author" itemprop="author">
                [profile]
                <a href="{author-url}">{author}</a>
                [/profile]
            </span>
            
            <span class="views" itemprop="interactionCount">{views} Görüntülenme</span>
        </div>
    </header>
    
    <!-- Resim Galeri -->
    <div class="news-gallery">
        [image-1]
        <figure class="gallery-item">
            <a href="{image-1}" class="fancybox" data-fancybox="gallery">
                <img src="{image-1}" alt="{title}" itemprop="image" />
            </a>
            [xfvalue_resim_aciklama]
            <figcaption>{xfvalue_resim_aciklama}</figcaption>
            [/xfvalue_resim_aciklama]
        </figure>
        [/image-1]
        
        [image-2]
        <figure class="gallery-item">
            <a href="{image-2}" class="fancybox" data-fancybox="gallery">
                <img src="{image-2}" alt="{title}" />
            </a>
        </figure>
        [/image-2]
        
        [image-3]
        <figure class="gallery-item">
            <a href="{image-3}" class="fancybox" data-fancybox="gallery">
                <img src="{image-3}" alt="{title}" />
            </a>
        </figure>
        [/image-3]
    </div>
    
    <!-- Video -->
    <div class="news-video">
        [xfvalue_video]
        <div class="video-container">
            {xfvalue_video}
        </div>
        [/xfvalue_video]
        
        [xfvalue_youtube]
        <div class="youtube-container">
            <iframe src="https://www.youtube.com/embed/{xfvalue_youtube}" 
                    frameborder="0" 
                    allowfullscreen>
            </iframe>
        </div>
        [/xfvalue_youtube]
    </div>
    
    <!-- Kısa İçerik -->
    <section class="short-story" itemprop="description">
        {short-story}
    </section>
    
    <!-- Tam İçerik -->
    <section class="full-story" itemprop="articleBody">
        {full-story}
        
        <!-- Sayfalama -->
        [pages]
        <div class="news-pages">
            <span class="pages-label">Sayfalar:</span>
            {pages}
        </div>
        [/pages]
    </section>
    
    <!-- Etiketler -->
    <footer class="news-tags">
        [tags]
        <div class="tags-container">
            <span class="tags-label">Etiketler:</span>
            {tags}
        </div>
        [/tags]
    </footer>
    
    <!-- Sosyal Paylaşım -->
    <div class="social-share">
        <span class="share-label">Paylaş:</span>
        
        [vk]
        <a href="https://vk.com/share.php?url={full-link}&title={title}" 
           class="share-vk" 
           target="_blank" 
           rel="nofollow">
            <i class="icon-vk"></i> VK
        </a>
        [/vk]
        
        [odnoklassniki]
        <a href="https://connect.ok.ru/offer?url={full-link}&title={title}" 
           class="share-ok" 
           target="_blank" 
           rel="nofollow">
            <i class="icon-ok"></i> OK
        </a>
        [/odnoklassniki]
        
        [facebook]
        <a href="https://www.facebook.com/sharer/sharer.php?u={full-link}" 
           class="share-facebook" 
           target="_blank" 
           rel="nofollow">
            <i class="icon-facebook"></i> Facebook
        </a>
        [/facebook]
        
        [twitter]
        <a href="https://twitter.com/intent/tweet?url={full-link}&text={title}" 
           class="share-twitter" 
           target="_blank" 
           rel="nofollow">
            <i class="icon-twitter"></i> Twitter
        </a>
        [/twitter]
        
        [google]
        <a href="https://plus.google.com/share?url={full-link}" 
           class="share-google" 
           target="_blank" 
           rel="nofollow">
            <i class="icon-google"></i> Google+
        </a>
        [/google]
        
        [mailru]
        <a href="https://connect.mail.ru/share?url={full-link}&title={title}" 
           class="share-mailru" 
           target="_blank" 
           rel="nofollow">
            <i class="icon-mailru"></i> Mail.ru
        </a>
        [/mailru]
        
        [whatsapp]
        <a href="whatsapp://send?text={title} - {full-link}" 
           class="share-whatsapp" 
           target="_blank" 
           rel="nofollow">
            <i class="icon-whatsapp"></i> WhatsApp
        </a>
        [/whatsapp]
        
        [telegram]
        <a href="https://t.me/share/url?url={full-link}&text={title}" 
           class="share-telegram" 
           target="_blank" 
           rel="nofollow">
            <i class="icon-telegram"></i> Telegram
        </a>
        [/telegram]
    </div>
    
    <!-- Önceki / Sonraki Haber -->
    <nav class="news-navigation">
        [prev-url]
        <div class="prev-news">
            <a href="{prev-url}" rel="prev">
                <i class="icon-arrow-left"></i>
                <span class="nav-label">Önceki:</span>
                <span class="nav-title">{prev-title}</span>
            </a>
        </div>
        [/prev-url]
        
        [next-url]
        <div class="next-news">
            <a href="{next-url}" rel="next">
                <span class="nav-label">Sonraki:</span>
                <span class="nav-title">{next-title}</span>
                <i class="icon-arrow-right"></i>
            </a>
        </div>
        [/next-url]
    </nav>
    
    <!-- Yorumlar -->
    <section class="news-comments" id="comments">
        <h3 class="comments-title">Yorumlar ({comments-num})</h3>
        <div id="dle-ajax-comments"></div>
    </section>
    
    <!-- Yorum Ekleme Formu -->
    <section class="add-comment" id="add-comment">
        [allow-comments]
        <h4>Yorum Yaz</h4>
        <form id="addCommentForm" method="post">
            <input type="hidden" name="dle_allow_hash" value="{dle_login_hash}" />
            <input type="hidden" name="news_id" value="{news-id}" />
            
            [is-logged]
            <div class="form-group">
                <label>Adınız:</label>
                <input type="text" name="name" value="{login}" readonly class="form-control" />
            </div>
            [/is-logged]
            
            [not-logged]
            <div class="form-group">
                <label>Adınız:</label>
                <input type="text" name="name" required class="form-control" />
            </div>
            
            <div class="form-group">
                <label>E-posta:</label>
                <input type="email" name="email" required class="form-control" />
            </div>
            [/not-logged]
            
            <div class="form-group">
                <label>Yorumunuz:</label>
                <textarea name="text" required class="form-control" rows="5"></textarea>
            </div>
            
            [not-logged]
            <div class="form-group captcha">
                <label>Güvenlik Kodu:</label>
                <div class="captcha-box">
                    <span id="dle-captcha">{security-code}</span>
                    <input type="text" name="sec_code" required class="form-control" />
                    <a href="#" onclick="reloadCaptcha(); return false;">
                        <i class="icon-refresh"></i> Yenile
                    </a>
                </div>
            </div>
            [/not-logged]
            
            <div class="form-actions">
                <button type="submit" class="btn-submit">
                    <i class="icon-send"></i> Yorum Gönder
                </button>
            </div>
        </form>
        [/allow-comments]
        
        [not-allow-comments]
        <div class="comments-closed">
            <p>Bu habere yorum yapılamaz.</p>
        </div>
        [/not-allow-comments]
    </section>
    
</article>

<!-- İlgili Haberler -->
<aside class="related-news">
    <h3 class="related-title">İlgili Haberler</h3>
    <div class="related-grid">
        {related-news}
    </div>
</aside>
```

---

### 3. Yorum (Comments) HTML Yapısı

```html
<!-- Yorum Listesi -->
<div class="comments-list" id="comments-list">
    
    <!-- Tek Yorum -->
    <div class="comment" id="comment-{comment-id}" data-comment-id="{comment-id}">
        
        <!-- Yorum Yazarı -->
        <div class="comment-author">
            [is-registered]
            <a href="{profile-url}" class="author-avatar">
                [user-foto]
                <img src="{user-foto}" alt="{comment-author}" />
                [/user-foto]
                [not-user-foto]
                <img src="{THEME}/dleimages/avatar.jpg" alt="{comment-author}" />
                [/not-user-foto]
            </a>
            [/is-registered]
            
            [not-registered]
            <div class="author-avatar guest">
                <i class="icon-user"></i>
            </div>
            [/not-registered]
            
            <div class="author-info">
                <span class="author-name">
                    [is-registered]
                    <a href="{profile-url}" onclick="ShowProfile('{comment-author}', '{profile-url}'); return false;">
                        {comment-author}
                    </a>
                    [/is-registered]
                    [not-registered]
                    {comment-author}
                    [/not-registered]
                </span>
                
                <span class="comment-date">
                    <time datetime="{comment-date=Y-m-d\TH:i:s}">{comment-date=d.m.Y H:i}</time>
                </span>
                
                [user-group]
                <span class="user-group-badge group-{user-group-id}">{user-group-name}</span>
                [/user-group]
            </div>
        </div>
        
        <!-- Yorum İçeriği -->
        <div class="comment-content">
            <div class="comment-text" id="comment-text-{comment-id}">
                {comment-text}
            </div>
            
            <!-- İmza -->
            [user-signature]
            <div class="comment-signature">
                ---
                {user-signature}
            </div>
            [/user-signature]
            
            <!-- Özel Alanlar -->
            [user-xfields]
            <div class="comment-xfields">
                {user-xfields}
            </div>
            [/user-xfields]
        </div>
        
        <!-- Yorum Footer -->
        <div class="comment-footer">
            <!-- Oy Verme -->
            <div class="comment-rating">
                [rating]
                <span class="rating-count">{comment-rating}</span>
                [rating-plus]
                <button class="btn-rating-plus" onclick="doCommentsRate('plus', {comment-id}); return false;">
                    <i class="icon-thumbs-up"></i>
                </button>
                [/rating-plus]
                [rating-minus]
                <button class="btn-rating-minus" onclick="doCommentsRate('minus', {comment-id}); return false;">
                    <i class="icon-thumbs-down"></i>
                </button>
                [/rating-minus]
                [/rating]
            </div>
            
            <!-- İşlemler -->
            <div class="comment-actions">
                [reply]
                <button class="btn-reply" onclick="replyToComment({comment-id}, '{comment-author}')">
                    <i class="icon-reply"></i> Cevapla
                </button>
                [/reply]
                
                [quote]
                <button class="btn-quote" onclick="quoteComment({comment-id})">
                    <i class="icon-quote"></i> Alıntıla
                </button>
                [/quote]
                
                [complaint]
                <button class="btn-complaint" onclick="AddComplaint({comment-id}, 'comment')">
                    <i class="icon-flag"></i> Şikayet
                </button>
                [/complaint]
                
                [edit]
                <button class="btn-edit" onclick="editComment({comment-id})">
                    <i class="icon-edit"></i> Düzenle
                </button>
                [/edit]
                
                [del]
                <button class="btn-delete" onclick="deleteComment({comment-id})">
                    <i class="icon-trash"></i> Sil
                </button>
                [/del]
            </div>
        </div>
        
        <!-- Cevaplar -->
        [has-replies]
        <div class="comment-replies" id="replies-{comment-id}">
            {comment-replies}
        </div>
        [/has-replies]
        
    </div>
    
</div>

<!-- Sayfalama -->
<div class="comments-navigation" id="comments-navigation">
    {comments-navigation}
</div>
```

---

### 4. Kullanıcı Profili HTML Yapısı

```html
<!-- Profil Sayfası -->
<div class="profile-page" itemscope itemtype="http://schema.org/Person">
    
    <!-- Profil Header -->
    <header class="profile-header">
        <div class="profile-avatar">
            [user-foto]
            <img src="{user-foto}" alt="{profile-name}" itemprop="image" />
            [/user-foto]
            [not-user-foto]
            <img src="{THEME}/dleimages/avatar.jpg" alt="{profile-name}" />
            [/not-user-foto]
        </div>
        
        <div class="profile-info">
            <h1 class="profile-name" itemprop="name">{profile-name}</h1>
            
            <div class="profile-meta">
                <span class="user-group">{user-group-name}</span>
                <span class="register-date">
                    Kayıt: {profile-reg-date=d.m.Y}
                </span>
                <span class="last-date">
                    Son Giriş: {profile-lastdate=d.m.Y H:i}
                </span>
            </div>
            
            <!-- İstatistikler -->
            <div class="profile-stats">
                <div class="stat-item">
                    <span class="stat-value">{profile-news-num}</span>
                    <span class="stat-label">Haber</span>
                </div>
                <div class="stat-item">
                    <span class="stat-value">{profile-comm-num}</span>
                    <span class="stat-label">Yorum</span>
                </div>
                <div class="stat-item">
                    <span class="stat-value">{profile-files-num}</span>
                    <span class="stat-label">Dosya</span>
                </div>
                <div class="stat-item">
                    <span class="stat-value">{profile-favorites-num}</span>
                    <span class="stat-label">Favori</span>
                </div>
            </div>
            
            <!-- Oy Puanı -->
            <div class="profile-rating">
                <span class="rating-label">Kullanıcı Puanı:</span>
                {user-rating}
            </div>
        </div>
    </header>
    
    <!-- Profil İçerik -->
    <div class="profile-content">
        <!-- Kişisel Bilgiler -->
        <section class="profile-personal">
            <h3>Kişisel Bilgiler</h3>
            
            <dl class="personal-list">
                [profile-fullname]
                <dt>Ad Soyad:</dt>
                <dd itemprop="name">{profile-fullname}</dd>
                [/profile-fullname]
                
                [profile-email]
                <dt>E-posta:</dt>
                <dd>{profile-email}</dd>
                [/profile-email]
                
                [profile-land]
                <dt>Konum:</dt>
                <dd itemprop="homeLocation">{profile-land}</dd>
                [/profile-land]
                
                [profile-icq]
                <dt>ICQ:</dt>
                <dd>{profile-icq}</dd>
                [/profile-icq]
                
                [profile-skype]
                <dt>Skype:</dt>
                <dd>{profile-skype}</dd>
                [/profile-skype]
                
                [profile-site]
                <dt>Web Site:</dt>
                <dd>
                    <a href="{profile-site}" rel="nofollow" target="_blank">
                        {profile-site}
                    </a>
                </dd>
                [/profile-site]
            </dl>
        </section>
        
        <!-- Özel Alanlar -->
        <section class="profile-xfields">
            <h3>Ek Bilgiler</h3>
            {profile-xfields}
        </section>
        
        <!-- İmza -->
        [profile-signature]
        <section class="profile-signature">
            <h3>İmza</h3>
            <div class="signature-content">
                {profile-signature}
            </div>
        </section>
        [/profile-signature]
        
        <!-- Son Haberler -->
        <section class="profile-news">
            <h3>Son Haberleri</h3>
            <ul class="news-list">
                {profile-news-list}
            </ul>
        </section>
        
        <!-- Son Yorumlar -->
        <section class="profile-comments">
            <h3>Son Yorumları</h3>
            <ul class="comment-list">
                {profile-comments-list}
            </ul>
        </section>
    </div>
    
    <!-- Profil İşlemleri (Kendi Profiliniz) -->
    [is-owner]
    <div class="profile-actions">
        <a href="{profile-edit-url}" class="btn-edit-profile">
            <i class="icon-edit"></i> Profili Düzenle
        </a>
        <a href="{password-change-url}" class="btn-change-password">
            <i class="icon-lock"></i> Şifre Değiştir
        </a>
    </div>
    [/is-owner]
    
</div>
```

---

### 5. Kayıt Formu HTML Yapısı

```html
<!-- Kayıt Formu -->
<div class="registration-page">
    <h1 class="page-title">Kayıt Ol</h1>
    
    <form id="registrationForm" method="post" class="registration-form">
        <input type="hidden" name="dle_allow_hash" value="{dle_login_hash}" />
        <input type="hidden" name="action" value="register" />
        
        <!-- Kullanıcı Adı -->
        <div class="form-group">
            <label for="login">Kullanıcı Adı: <span class="required">*</span></label>
            <input type="text" 
                   id="login" 
                   name="login" 
                   required 
                   minlength="3" 
                   maxlength="40"
                   class="form-control"
                   placeholder="Kullanıcı adınızı girin" />
            <span class="help-text">En az 3 karakter olmalıdır</span>
            <span class="error-text" id="login-error"></span>
        </div>
        
        <!-- E-posta -->
        <div class="form-group">
            <label for="email">E-posta: <span class="required">*</span></label>
            <input type="email" 
                   id="email" 
                   name="email" 
                   required 
                   class="form-control"
                   placeholder="ornek@email.com" />
            <span class="error-text" id="email-error"></span>
        </div>
        
        <!-- Şifre -->
        <div class="form-group">
            <label for="password">Şifre: <span class="required">*</span></label>
            <input type="password" 
                   id="password" 
                   name="password" 
                   required 
                   minlength="6"
                   class="form-control"
                   placeholder="******" />
            <span class="help-text">En az 6 karakter olmalıdır</span>
            <span class="error-text" id="password-error"></span>
        </div>
        
        <!-- Şifre Tekrar -->
        <div class="form-group">
            <label for="password2">Şifre Tekrar: <span class="required">*</span></label>
            <input type="password" 
                   id="password2" 
                   name="password2" 
                   required 
                   class="form-control"
                   placeholder="******" />
            <span class="error-text" id="password2-error"></span>
        </div>
        
        <!-- Güvenlik Kodu -->
        <div class="form-group captcha">
            <label for="sec_code">Güvenlik Kodu: <span class="required">*</span></label>
            <div class="captcha-box">
                <span id="dle-captcha">{security-code}</span>
                <input type="text" 
                       id="sec_code" 
                       name="sec_code" 
                       required 
                       class="form-control"
                       placeholder="Kodu girin" />
                <a href="#" onclick="reloadCaptcha(); return false;" class="captcha-reload">
                    <i class="icon-refresh"></i> Yenile
                </a>
            </div>
            <span class="error-text" id="sec_code-error"></span>
        </div>
        
        <!-- Özel Kayıt Alanları -->
        {registration-xfields}
        
        <!-- Kullanıcı Sözleşmesi -->
        <div class="form-group checkbox">
            <label>
                <input type="checkbox" name="agreement" required />
                <a href="{agreement-url}" target="_blank">Kullanıcı Sözleşmesi</a>'ni okudum ve kabul ediyorum.
            </label>
            <span class="error-text" id="agreement-error"></span>
        </div>
        
        <!-- Gizlilik Politikası -->
        <div class="form-group checkbox">
            <label>
                <input type="checkbox" name="privacy" required />
                <a href="{privacy-url}" target="_blank">Gizlilik Politikası</a>'nı kabul ediyorum.
            </label>
            <span class="error-text" id="privacy-error"></span>
        </div>
        
        <!-- Gönder Butonu -->
        <div class="form-actions">
            <button type="submit" class="btn-submit">
                <i class="icon-user-plus"></i> Kayıt Ol
            </button>
        </div>
        
        <!-- Giriş Linki -->
        <div class="form-footer">
            <p>Zaten hesabınız var mı? 
                <a href="{login-url}">Giriş Yapın</a>
            </p>
        </div>
    </form>
</div>
```

---

### 6. Haber Ekleme Formu HTML Yapısı

```html
<!-- Haber Ekleme Formu -->
<div class="add-news-page">
    <h1 class="page-title">Haber Ekle</h1>
    
    <form id="addNewsForm" method="post" enctype="multipart/form-data" class="news-form">
        <input type="hidden" name="dle_allow_hash" value="{dle_login_hash}" />
        <input type="hidden" name="action" value="addnews" />
        
        <!-- Başlık -->
        <div class="form-group">
            <label for="title">Başlık: <span class="required">*</span></label>
            <input type="text" 
                   id="title" 
                   name="title" 
                   required 
                   maxlength="250"
                   class="form-control"
                   placeholder="Haber başlığını girin" />
        </div>
        
        <!-- Kategori -->
        <div class="form-group">
            <label for="category">Kategori: <span class="required">*</span></label>
            <select id="category" name="category" required class="form-control">
                <option value="">Seçiniz...</option>
                {category-options}
            </select>
        </div>
        
        <!-- Kısa İçerik -->
        <div class="form-group">
            <label for="short-story">Kısa İçerik: <span class="required">*</span></label>
            <div class="editor-toolbar">{short-story-editor}</div>
            <textarea id="short-story" 
                      name="short_story" 
                      required 
                      class="form-control editor"
                      rows="10"></textarea>
        </div>
        
        <!-- Tam İçerik -->
        <div class="form-group">
            <label for="full-story">Tam İçerik:</label>
            <div class="editor-toolbar">{full-story-editor}</div>
            <textarea id="full-story" 
                      name="full_story" 
                      class="form-control editor"
                      rows="20"></textarea>
        </div>
        
        <!-- Özel Alanlar -->
        <div class="xfields-container">
            {xfields}
        </div>
        
        <!-- Etiketler -->
        <div class="form-group">
            <label for="tags">Etiketler:</label>
            <input type="text" 
                   id="tags" 
                   name="tags" 
                   class="form-control"
                   placeholder="virgülle ayırarak yazın" />
        </div>
        
        <!-- SEO URL -->
        <div class="form-group">
            <label for="alt-name">SEO URL:</label>
            <input type="text" 
                   id="alt-name" 
                   name="alt_name" 
                   maxlength="250"
                   class="form-control"
                   placeholder="otomatik-olusturulur" />
            <span class="help-text">Boş bırakılırsa otomatik oluşturulur</span>
        </div>
        
        <!-- Meta Açıklama -->
        <div class="form-group">
            <label for="descr">Meta Açıklama:</label>
            <textarea id="descr" 
                      name="descr" 
                      maxlength="250"
                      class="form-control"
                      rows="3"
                      placeholder="SEO açıklaması"></textarea>
        </div>
        
        <!-- Meta Anahtar Kelimeler -->
        <div class="form-group">
            <label for="keywords">Meta Anahtar Kelimeler:</label>
            <textarea id="keywords" 
                      name="keywords" 
                      class="form-control"
                      rows="3"
                      placeholder="anahtar, kelimeler"></textarea>
        </div>
        
        <!-- Dosya Yükleme -->
        <div class="form-group">
            <label>Dosya Yükle:</label>
            <div class="file-uploader" id="file-uploader">
                <input type="file" name="files[]" multiple class="file-input" />
                <div class="upload-progress"></div>
                <div class="uploaded-files"></div>
            </div>
        </div>
        
        <!-- Resim Yükleme -->
        <div class="form-group">
            <label>Resim Yükle:</label>
            <div class="image-uploader" id="image-uploader">
                <input type="file" name="images[]" accept="image/*" multiple class="image-input" />
                <div class="upload-progress"></div>
                <div class="uploaded-images"></div>
            </div>
        </div>
        
        <!-- Gönder Butonları -->
        <div class="form-actions">
            <button type="submit" name="submit" value="publish" class="btn-submit">
                <i class="icon-publish"></i> Yayınla
            </button>
            <button type="submit" name="submit" value="draft" class="btn-draft">
                <i class="icon-save"></i> Taslak Kaydet
            </button>
            <button type="button" onclick="previewNews()" class="btn-preview">
                <i class="icon-eye"></i> Önizle
            </button>
        </div>
    </form>
</div>
```

---

### 7. Giriş Formu HTML Yapısı

```html
<!-- Giriş Formu -->
<div class="login-page">
    <h1 class="page-title">Giriş Yap</h1>
    
    <form id="loginForm" method="post" class="login-form">
        <input type="hidden" name="dle_allow_hash" value="{dle_login_hash}" />
        <input type="hidden" name="action" value="login" />
        <input type="hidden" name="login" value="submit" />
        
        <!-- Kullanıcı Adı / E-posta -->
        <div class="form-group">
            <label for="login_username">Kullanıcı Adı veya E-posta:</label>
            <input type="text" 
                   id="login_username" 
                   name="login_username" 
                   required 
                   class="form-control"
                   placeholder="Kullanıcı adı veya e-posta" />
        </div>
        
        <!-- Şifre -->
        <div class="form-group">
            <label for="login_password">Şifre:</label>
            <input type="password" 
                   id="login_password" 
                   name="login_password" 
                   required 
                   class="form-control"
                   placeholder="******" />
        </div>
        
        <!-- Beni Hatırla -->
        <div class="form-group checkbox">
            <label>
                <input type="checkbox" name="login_store" value="1" />
                Beni Hatırla
            </label>
        </div>
        
        <!-- Güvenlik Kodu (Opsiyonel) -->
        [need-captcha]
        <div class="form-group captcha">
            <label for="login_sec_code">Güvenlik Kodu:</label>
            <div class="captcha-box">
                <span id="dle-captcha">{security-code}</span>
                <input type="text" 
                       id="login_sec_code" 
                       name="sec_code" 
                       class="form-control" />
                <a href="#" onclick="reloadCaptcha(); return false;">
                    <i class="icon-refresh"></i> Yenile
                </a>
            </div>
        </div>
        [/need-captcha]
        
        <!-- Gönder Butonu -->
        <div class="form-actions">
            <button type="submit" class="btn-submit">
                <i class="icon-sign-in"></i> Giriş Yap
            </button>
        </div>
        
        <!-- Ek Linkler -->
        <div class="form-footer">
            <p>
                <a href="{register-url}">Kayıt Ol</a> | 
                <a href="{lostpassword-url}">Şifremi Unuttum</a>
            </p>
        </div>
    </form>
    
    <!-- Sosyal Giriş -->
    [social-login]
    <div class="social-login">
        <span class="separator">veya</span>
        
        <a href="{vk-login-url}" class="btn-social vk">
            <i class="icon-vk"></i> VK ile Giriş
        </a>
        
        <a href="{facebook-login-url}" class="btn-social facebook">
            <i class="icon-facebook"></i> Facebook ile Giriş
        </a>
        
        <a href="{google-login-url}" class="btn-social google">
            <i class="icon-google"></i> Google ile Giriş
        </a>
    </div>
    [/social-login]
</div>
```

---

### 8. Sayfalama (Navigation) HTML Yapısı

```html
<!-- Sayfalama -->
<div class="pagination" id="pagination">
    
    <!-- Önceki -->
    [prev-page]
    <a href="{prev-page-url}" class="page-prev" rel="prev">
        <i class="icon-arrow-left"></i> Önceki
    </a>
    [/prev-page]
    
    [not-prev-page]
    <span class="page-prev disabled">
        <i class="icon-arrow-left"></i> Önceki
    </span>
    [/not-prev-page]
    
    <!-- Sayfa Numaraları -->
    <div class="page-numbers">
        [first-page]
        <a href="{first-page-url}" class="page-first">1</a>
        [/first-page]
        
        [left-separator]
        <span class="page-separator">...</span>
        [/left-separator]
        
        {pages}
        <!-- Örnek: <a href="..." class="page-num">2</a> -->
        <!-- Örnek: <span class="page-num active">3</span> -->
        
        [right-separator]
        <span class="page-separator">...</span>
        [/right-separator]
        
        [last-page]
        <a href="{last-page-url}" class="page-last">{total-pages}</a>
        [/last-page]
    </div>
    
    <!-- Sonraki -->
    [next-page]
    <a href="{next-page-url}" class="page-next" rel="next">
        Sonraki <i class="icon-arrow-right"></i>
    </a>
    [/next-page]
    
    [not-next-page]
    <span class="page-next disabled">
        Sonraki <i class="icon-arrow-right"></i>
    </span>
    [/not-next-page]
    
</div>

<!-- Sayfa Bilgisi -->
<div class="page-info">
    Toplam {total-pages} sayfadan {current-page}. sayfa
</div>
```

---

### 9. Arama Formu HTML Yapısı

```html
<!-- Arama Formu -->
<div class="search-page">
    <h1 class="page-title">Arama</h1>
    
    <form id="searchForm" method="get" class="search-form">
        <input type="hidden" name="do" value="search" />
        <input type="hidden" name="subaction" value="search" />
        
        <!-- Arama Kutusu -->
        <div class="form-group search-input">
            <input type="text" 
                   id="story" 
                   name="story" 
                   required 
                   minlength="3"
                   class="form-control"
                   placeholder="Aramak istediğiniz kelimeyi girin..." />
            <button type="submit" class="btn-search">
                <i class="icon-search"></i> Ara
            </button>
        </div>
        
        <!-- Gelişmiş Arama -->
        <div class="advanced-search">
            <button type="button" class="btn-toggle-advanced">
                <i class="icon-chevron-down"></i> Gelişmiş Arama
            </button>
            
            <div class="advanced-options">
                <!-- Kategori Seçimi -->
                <div class="form-group">
                    <label for="category">Kategori:</label>
                    <select id="category" name="category" class="form-control">
                        <option value="0">Tüm Kategoriler</option>
                        {category-options}
                    </select>
                </div>
                
                <!-- Tarih Aralığı -->
                <div class="form-group">
                    <label for="fromdate">Başlangıç Tarihi:</label>
                    <input type="date" id="fromdate" name="fromdate" class="form-control" />
                </div>
                
                <div class="form-group">
                    <label for="todate">Bitiş Tarihi:</label>
                    <input type="date" id="todate" name="todate" class="form-control" />
                </div>
                
                <!-- Arama Tipi -->
                <div class="form-group">
                    <label>Arama Tipi:</label>
                    <div class="radio-group">
                        <label>
                            <input type="radio" name="search_type" value="title" />
                            Sadece Başlıkta
                        </label>
                        <label>
                            <input type="radio" name="search_type" value="content" />
                            İçerikte
                        </label>
                        <label>
                            <input type="radio" name="search_type" value="tags" />
                            Etiketlerde
                        </label>
                        <label>
                            <input type="radio" name="search_type" value="all" checked />
                            Tümü
                        </label>
                    </div>
                </div>
                
                <!-- Sıralama -->
                <div class="form-group">
                    <label for="sortby">Sıralama:</label>
                    <select id="sortby" name="sortby" class="form-control">
                        <option value="date">Tarihe Göre</option>
                        <option value="title">Başlığa Göre</option>
                        <option value="views">Görüntülenmeye Göre</option>
                        <option value="rating">Oy Puanına Göre</option>
                    </select>
                </div>
                
                <!-- Sıralama Yönü -->
                <div class="form-group">
                    <label for="order">Sıralama Yönü:</label>
                    <select id="order" name="order" class="form-control">
                        <option value="desc">Azalan</option>
                        <option value="asc">Artan</option>
                    </select>
                </div>
            </div>
        </div>
        
        <!-- Gönder Butonu -->
        <div class="form-actions">
            <button type="submit" class="btn-submit">
                <i class="icon-search"></i> Ara
            </button>
        </div>
    </form>
    
    <!-- Arama Sonuçları -->
    [search-results]
    <div class="search-results">
        <h2>Arama Sonuçları</h2>
        <p class="results-info">{story} için {results-count} sonuç bulundu</p>
        
        <div class="results-list">
            {results}
        </div>
        
        <!-- Sayfalama -->
        {pagination}
    </div>
    [/search-results]
</div>
```

---

### 10. Özel Alan (X-Fields) HTML Yapısı

```html
<!-- Özel Alanlar Container -->
<div class="xfields-container">
    
    <!-- Text Alanı -->
    [xfvalue_yonetmen]
    <div class="xfield-item xfield-text">
        <label class="xfield-label">Yönetmen:</label>
        <span class="xfield-value">{xfvalue_yonetmen}</span>
    </div>
    [/xfvalue_yonetmen]
    
    <!-- Textarea Alanı -->
    [xfvalue_konu]
    <div class="xfield-item xfield-textarea">
        <label class="xfield-label">Konusu:</label>
        <div class="xfield-value">
            <p>{xfvalue_konu}</p>
        </div>
    </div>
    [/xfvalue_konu]
    
    <!-- Select Alanı -->
    [xfvalue_tur]
    <div class="xfield-item xfield-select">
        <label class="xfield-label">Tür:</label>
        <span class="xfield-value">{xfvalue_tur}</span>
    </div>
    [/xfvalue_tur]
    
    <!-- Checkbox Alanı -->
    [xfvalue_vizyonda]
    <div class="xfield-item xfield-checkbox">
        <label class="xfield-label">Vizyonda mı?</label>
        <span class="xfield-value">
            [xfvalue_vizyonda=1]
            <i class="icon-check"></i> Evet
            [/xfvalue_vizyonda=1]
            [xfvalue_vizyonda=0]
            <i class="icon-times"></i> Hayır
            [/xfvalue_vizyonda=0]
        </span>
    </div>
    [/xfvalue_vizyonda]
    
    <!-- Tarih Alanı -->
    [xfvalue_vizyon_tarihi]
    <div class="xfield-item xfield-date">
        <label class="xfield-label">Vizyon Tarihi:</label>
        <span class="xfield-value">{xfvalue_vizyon_tarihi=d.m.Y}</span>
    </div>
    [/xfvalue_vizyon_tarihi]
    
    <!-- Sayı Alanı -->
    [xfvalue_sure]
    <div class="xfield-item xfield-number">
        <label class="xfield-label">Süre:</label>
        <span class="xfield-value">{xfvalue_sure} dakika</span>
    </div>
    [/xfvalue_sure]
    
    <!-- Dosya Alanı -->
    [xfvalue_fragman]
    <div class="xfield-item xfield-file">
        <label class="xfield-label">Fragman:</label>
        <a href="{xfvalue_fragman}" class="xfield-value" download>
            <i class="icon-download"></i> İndir
        </a>
    </div>
    [/xfvalue_fragman]
    
    <!-- Resim Alanı -->
    [xfvalue_afis]
    <div class="xfield-item xfield-image">
        <label class="xfield-label">Afiş:</label>
        <div class="xfield-value">
            <img src="{xfvalue_afis}" alt="{title}" />
        </div>
    </div>
    [/xfvalue_afis]
    
    <!-- Video Alanı -->
    [xfvalue_video]
    <div class="xfield-item xfield-video">
        <label class="xfield-label">Video:</label>
        <div class="xfield-value">
            {xfvalue_video}
        </div>
    </div>
    [/xfvalue_video]
    
</div>
```

---

## Yönetim Paneli (Admin UI) Standartları

### Genel Bakış

DLE Yönetim Paneli, modern ve responsive bir tasarıma sahiptir. Bootstrap 3 ve Material Design prensiplerini kullanır.

**Giriş Noktası:** `admin.php`

**URL Yapısı:**
```
https://siteadi.com/admin.php
https://siteadi.com/admin.php?mod=module_adi&action=action_name
```

---

### Admin Panel Klasör Yapısı

```
engine/inc/
├── addnews.php           # Haber ekleme
├── editnews.php          # Haber düzenleme
├── categories.php        # Kategori yönetimi
├── xfields.php           # Özel alan yönetimi
├── templates.php         # Template yönetimi
├── options.php           # Sistem ayarları
├── editusers.php         # Kullanıcı düzenleme
├── usergroup.php         # Grup yönetimi
├── plugins.php           # Eklenti yönetimi
├── files.php             # Dosya yönetimi
├── comments.php          # Yorum yönetimi
├── complaint.php         # Şikayet yönetimi
├── search.php            # Arama ve indeks
├── rebuild.php           # Veritabanı yenileme
├── clean.php             # Temizlik işlemleri
├── newsletter.php        # Bülten gönderimi
├── social.php            # Sosyal medya ayarları
├── metatags.php          # Meta etiket yönetimi
├── redirects.php         # Yönlendirme yönetimi
├── links.php             # Link yönetimi
├── wordfilter.php        # Kelime filtresi
├── iptools.php           # IP araçları
├── blockip.php           # IP engelleme
├── question.php          # Güvenlik sorusu
├── videoconfig.php       # Video ayarları
├── dboption.php          # Veritabanı ayarları
├── email.php             # E-posta şablonları
├── editvote.php          # Anket yönetimi
├── static.php            # Statik sayfalar
├── tagscloud.php         # Etiket bulutu
├── storage.php           # Depolama yönetimi
├── userfields.php        # Profil alanları
├── massactions.php       # Toplu işlemler
├── logs.php              # Log kayıtları
└── include/
    ├── functions.inc.php   # Admin fonksiyonları
    ├── init.php            # Başlangıç dosyası
    └── ...
```

---

### Admin UI HTML Yapısı

#### 1. Admin Header (Üst Kısım)

```php
<?php
echoheader(
    "<i class=\"fa fa-file-text-o position-left\"></i><span class=\"text-semibold\">Başlık</span>",
    "Sayfa Açıklaması"
);
?>
```

**Çıktı HTML:**
```html
<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>DLE Admin Panel - Başlık</title>

    <!-- CSS Dosyaları -->
    <link rel="stylesheet" href="public/css/admin.css" />
    <link rel="stylesheet" href="public/css/font-awesome.min.css" />
    <link rel="stylesheet" href="public/css/bootstrap.css" />
    <link rel="stylesheet" href="public/css/components.css" />
    <link rel="stylesheet" href="public/css/colors.css" />
    <link rel="stylesheet" href="public/css/jquery-ui.css" />
</head>
<body class="admin-panel">
    <!-- Navigation -->
    <nav class="navbar navbar-inverse">
        <div class="navbar-header">
            <a class="navbar-brand" href="admin.php">DLE Admin</a>
        </div>
    </nav>

    <!-- Main Content -->
    <div class="admin-content">
```

---

#### 2. Admin Panel Yapısı

```html
<!-- Ana Panel -->
<div class="panel panel-default">
    <div class="panel-heading">
        <h6 class="panel-title">
            <i class="fa fa-cog position-left"></i>
            Panel Başlığı
        </h6>
        <div class="heading-elements">
            <ul class="icons-list">
                <li><a data-action="collapse"><i class="fa fa-chevron-up"></i></a></li>
                <li><a data-action="reload"><i class="fa fa-refresh"></i></a></li>
                <li><a data-action="close"><i class="fa fa-times"></i></a></li>
            </ul>
        </div>
    </div>

    <div class="panel-body">
        <!-- İçerik buraya -->
    </div>

    <div class="panel-footer">
        <!-- Footer butonları -->
    </div>
</div>
```

---

#### 3. Form Yapısı

```html
<!-- Yatay Form -->
<form method="post" action="admin.php?mod=module&action=save" class="form-horizontal" autocomplete="off">
    <input type="hidden" name="user_hash" value="<?php echo $dle_login_hash; ?>" />

    <!-- Text Input -->
    <div class="form-group">
        <label class="control-label col-sm-2">
            Alan Adı
            <span class="text-danger">*</span>
        </label>
        <div class="col-sm-10">
            <input type="text" name="field_name" class="form-control" value="" />
            <span class="help-block">Açıklama metni buraya</span>
        </div>
    </div>

    <!-- Textarea -->
    <div class="form-group">
        <label class="control-label col-sm-2">Açıklama</label>
        <div class="col-sm-10">
            <textarea name="description" class="form-control" rows="5"></textarea>
        </div>
    </div>

    <!-- Select -->
    <div class="form-group">
        <label class="control-label col-sm-2">Seçim</label>
        <div class="col-sm-10">
            <select name="select_field" class="uniform" data-width="100%">
                <option value="0">Seçiniz...</option>
                <option value="1">Seçenek 1</option>
                <option value="2">Seçenek 2</option>
            </select>
        </div>
    </div>

    <!-- Checkbox -->
    <div class="form-group">
        <label class="control-label col-sm-2">Aktif</label>
        <div class="col-sm-10">
            <div class="checkbox checkbox-switchery switchery-sm">
                <label>
                    <input type="checkbox" name="active" value="1" checked />
                    Aktif/Pasif
                </label>
            </div>
        </div>
    </div>

    <!-- Radio -->
    <div class="form-group">
        <label class="control-label col-sm-2">Tip</label>
        <div class="col-sm-10">
            <div class="radio">
                <label>
                    <input type="radio" name="type" value="1" checked />
                    Seçenek 1
                </label>
            </div>
            <div class="radio">
                <label>
                    <input type="radio" name="type" value="2" />
                    Seçenek 2
                </label>
            </div>
        </div>
    </div>

    <!-- File Upload -->
    <div class="form-group">
        <label class="control-label col-sm-2">Dosya</label>
        <div class="col-sm-10">
            <input type="file" name="upload" class="form-control" />
        </div>
    </div>

    <!-- Form Actions -->
    <div class="form-group form-actions">
        <div class="col-sm-offset-2 col-sm-10">
            <button type="submit" class="btn bg-teal btn-sm btn-raised">
                <i class="fa fa-save position-left"></i> Kaydet
            </button>
            <button type="reset" class="btn bg-grey btn-sm btn-raised">
                <i class="fa fa-undo position-left"></i> Temizle
            </button>
        </div>
    </div>
</form>
```

---

#### 4. Tablo Yapısı

```html
<!-- Veri Tablosu -->
<div class="table-responsive">
    <table class="table table-striped table-bordered table-hover">
        <thead>
            <tr>
                <th width="50">
                    <label class="fancy-checkbox">
                        <input type="checkbox" class="styled" id="select-all" />
                    </label>
                </th>
                <th>ID</th>
                <th>Başlık</th>
                <th>Kategori</th>
                <th>Yazar</th>
                <th>Tarih</th>
                <th>Yorumlar</th>
                <th>Görüntülenme</th>
                <th>Durum</th>
                <th width="150">İşlemler</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td>
                    <label class="fancy-checkbox">
                        <input type="checkbox" name="selected[]" value="1" class="styled" />
                    </label>
                </td>
                <td>1</td>
                <td>
                    <a href="?mod=editnews&action=edit&id=1">
                        Haber Başlığı
                    </a>
                </td>
                <td>Kategori Adı</td>
                <td>Yazar Adı</td>
                <td>15.01.2024 12:30</td>
                <td>
                    <span class="badge bg-blue">5</span>
                </td>
                <td>1,234</td>
                <td>
                    <span class="label label-success">Aktif</span>
                </td>
                <td>
                    <div class="btn-group">
                        <button type="button" class="btn btn-sm bg-teal dropdown-toggle" data-toggle="dropdown">
                            İşlemler <span class="caret"></span>
                        </button>
                        <ul class="dropdown-menu dropdown-menu-right">
                            <li>
                                <a href="?mod=editnews&action=edit&id=1">
                                    <i class="fa fa-edit"></i> Düzenle
                                </a>
                            </li>
                            <li>
                                <a href="?mod=editnews&action=approve&id=1">
                                    <i class="fa fa-check"></i> Onayla
                                </a>
                            </li>
                            <li>
                                <a href="?mod=editnews&action=fixed&id=1">
                                    <i class="fa fa-thumb-tack"></i> Sabitle
                                </a>
                            </li>
                            <li class="divider"></li>
                            <li>
                                <a href="#" onclick="return confirm('Silmek istediğinize emin misiniz?')">
                                    <i class="fa fa-trash text-danger"></i> Sil
                                </a>
                            </li>
                        </ul>
                    </div>
                </td>
            </tr>
        </tbody>
    </table>
</div>

<!-- Sayfalama -->
<div class="pagination">
    <ul>
        <li class="disabled"><a href="#">&laquo;</a></li>
        <li class="active"><a href="#">1</a></li>
        <li><a href="#">2</a></li>
        <li><a href="#">3</a></li>
        <li><a href="#">&raquo;</a></li>
    </ul>
</div>
```

---

#### 5. Mesaj Kutuları

```php
<?php
// Bilgi Mesajı
msg( "info", "Başlık", "Mesaj içeriği" );

// Başarı Mesajı
msg( "success", "Başarılı", "İşlem başarıyla tamamlandı" );

// Hata Mesajı
msg( "error", "Hata", "Bir hata oluştu" );

// Uyarı Mesajı
msg( "warning", "Uyarı", "Dikkat edilmesi gereken bir durum" );
?>
```

**HTML Çıktısı:**
```html
<!-- Bilgi -->
<div class="alert alert-info alert-styled-left alert-arrow-left">
    <button type="button" class="close" data-dismiss="alert">
        <span>&times;</span><span class="sr-only">Kapat</span>
    </button>
    <span class="text-semibold">Başlık:</span> Mesaj içeriği
</div>

<!-- Başarı -->
<div class="alert alert-success alert-styled-left alert-arrow-left">
    <button type="button" class="close" data-dismiss="alert">
        <span>&times;</span><span class="sr-only">Kapat</span>
    </button>
    <span class="text-semibold">Başarılı!</span> İşlem başarıyla tamamlandı
</div>

<!-- Hata -->
<div class="alert alert-danger alert-styled-left alert-arrow-left">
    <button type="button" class="close" data-dismiss="alert">
        <span>&times;</span><span class="sr-only">Kapat</span>
    </button>
    <span class="text-semibold">Hata!</span> Bir hata oluştu
</div>

<!-- Uyarı -->
<div class="alert alert-warning alert-styled-left alert-arrow-left">
    <button type="button" class="close" data-dismiss="alert">
        <span>&times;</span><span class="sr-only">Kapat</span>
    </button>
    <span class="text-semibold">Uyarı!</span> Dikkat edilmesi gereken bir durum
</div>
```

---

#### 6. Modal (Popup) Yapısı

```html
<!-- Modal Tetikleyici -->
<button type="button" class="btn bg-teal btn-sm btn-raised" data-toggle="modal" data-target="#myModal">
    <i class="fa fa-plus"></i> Modal Aç
</button>

<!-- Modal -->
<div class="modal fade" id="myModal" tabindex="-1" role="dialog">
    <div class="modal-dialog" role="document">
        <div class="modal-content">
            <div class="modal-header">
                <button type="button" class="close" data-dismiss="modal">
                    <span>&times;</span>
                </button>
                <h4 class="modal-title">Modal Başlığı</h4>
            </div>
            <div class="modal-body">
                <p>Modal içeriği buraya...</p>
            </div>
            <div class="modal-footer">
                <button type="button" class="btn btn-default" data-dismiss="modal">
                    Kapat
                </button>
                <button type="button" class="btn bg-teal">
                    Kaydet
                </button>
            </div>
        </div>
    </div>
</div>
```

---

#### 7. Bildirimler (Notifications)

```javascript
// Başarı Bildirimi
DLEPush.success('İşlem başarıyla tamamlandı!');

// Hata Bildirimi
DLEPush.error('Bir hata oluştu!');

// Uyarı Bildirimi
DLEPush.warning('Dikkat edilmesi gereken bir durum!');

// Bilgi Bildirimi
DLEPush.info('Bilgilendirme mesajı');

// Onay Kutusu
DLEconfirm('Bu işlemi yapmak istediğinize emin misiniz?', 'Onay', function() {
    // Onaylandı
}, function() {
    // İptal edildi
});

// Prompt
DLEprompt('Değer girin:', '', 'Başlık', function(value) {
    // Değer kaydedildi
}, true, 'Kaydet');
```

---

#### 8. Yükleme Göstergesi

```javascript
// Yükleniyor Göster
ShowLoading('İşlem yapılıyor...');

// Yükleniyor Gizle
HideLoading();
```

**HTML Çıktısı:**
```html
<div id="dle-loading" class="modal-loading">
    <div class="loading-content">
        <i class="fa fa-spinner fa-spin"></i>
        <span>İşlem yapılıyor...</span>
    </div>
</div>
```

---

#### 9. Sekme (Tab) Yapısı

```html
<!-- Tabs -->
<div class="tabbable">
    <ul class="nav nav-tabs" role="tablist">
        <li class="active">
            <a href="#tab1" data-toggle="tab" role="tab">
                <i class="fa fa-home"></i> Sekme 1
            </a>
        </li>
        <li>
            <a href="#tab2" data-toggle="tab" role="tab">
                <i class="fa fa-cog"></i> Sekme 2
            </a>
        </li>
        <li>
            <a href="#tab3" data-toggle="tab" role="tab">
                <i class="fa fa-users"></i> Sekme 3
            </a>
        </li>
    </ul>

    <div class="tab-content">
        <div class="tab-pane active" id="tab1" role="tabpanel">
            <div class="panel-body">
                Sekme 1 içeriği
            </div>
        </div>

        <div class="tab-pane" id="tab2" role="tabpanel">
            <div class="panel-body">
                Sekme 2 içeriği
            </div>
        </div>

        <div class="tab-pane" id="tab3" role="tabpanel">
            <div class="panel-body">
                Sekme 3 içeriği
            </div>
        </div>
    </div>
</div>
```

---

#### 10. Özel Alan (X-Fields) Yönetimi

```html
<!-- Özel Alan Ekleme/Düzenleme Formu -->
<div class="panel panel-default">
    <div class="panel-heading">
        Özel Alan Ayarları
    </div>
    <div class="panel-body">
        <div class="form-group">
            <label class="control-label col-sm-2">Alan Adı</label>
            <div class="col-sm-10">
                <input type="text" name="editedxfield[name]" class="form-control" />
            </div>
        </div>

        <div class="form-group">
            <label class="control-label col-sm-2">Açıklama</label>
            <div class="col-sm-10">
                <input type="text" name="editedxfield[description]" class="form-control" />
            </div>
        </div>

        <div class="form-group">
            <label class="control-label col-sm-2">Alan Tipi</label>
            <div class="col-sm-10">
                <select name="editedxfield[type]" class="uniform" data-width="100%">
                    <option value="text">Text (Tek Satır)</option>
                    <option value="textarea">Textarea (Çok Satır)</option>
                    <option value="select">Select (Seçim)</option>
                    <option value="checkbox">Checkbox</option>
                    <option value="radio">Radio</option>
                    <option value="image">Image (Resim)</option>
                    <option value="imagegalery">Image Gallery (Resim Galerisi)</option>
                    <option value="file">File (Dosya)</option>
                    <option value="video">Video</option>
                    <option value="audio">Audio</option>
                    <option value="datetime">Date/Time</option>
                    <option value="htmljs">HTML/JS</option>
                </select>
            </div>
        </div>

        <div class="form-group">
            <label class="control-label col-sm-2">Varsayılan Değer</label>
            <div class="col-sm-10">
                <input type="text" name="editedxfield[default]" class="form-control" />
            </div>
        </div>

        <div class="form-group">
            <label class="control-label col-sm-2">Kategoriler</label>
            <div class="col-sm-10">
                <select name="editedxfield[category][]" class="uniform" multiple data-width="100%">
                    <option value="1">Kategori 1</option>
                    <option value="2">Kategori 2</option>
                </select>
            </div>
        </div>

        <div class="form-group">
            <label class="control-label col-sm-2">Zorunlu Alan</label>
            <div class="col-sm-10">
                <div class="checkbox">
                    <label>
                        <input type="checkbox" name="editedxfield[not_required]" value="1" />
                        Bu alan zorunlu olmasın
                    </label>
                </div>
            </div>
        </div>

        <div class="form-group">
            <label class="control-label col-sm-2">Min/Max Değer</label>
            <div class="col-sm-10">
                <div class="row">
                    <div class="col-sm-6">
                        <input type="number" name="editedxfield[min]" class="form-control" placeholder="Min" />
                    </div>
                    <div class="col-sm-6">
                        <input type="number" name="editedxfield[max]" class="form-control" placeholder="Max" />
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>
```

---

#### 11. Kategori Yönetimi

```html
<!-- Kategori Ağacı -->
<div class="category-tree">
    <ul class="tree">
        <li>
            <label>
                <input type="checkbox" name="cat[]" value="1" />
                <i class="fa fa-folder"></i> Kategori 1
            </label>
            <ul>
                <li>
                    <label>
                        <input type="checkbox" name="cat[]" value="2" />
                        <i class="fa fa-folder-o"></i> Alt Kategori 1.1
                    </label>
                </li>
                <li>
                    <label>
                        <input type="checkbox" name="cat[]" value="3" />
                        <i class="fa fa-folder-o"></i> Alt Kategori 1.2
                    </label>
                </li>
            </ul>
        </li>
        <li>
            <label>
                <input type="checkbox" name="cat[]" value="4" />
                <i class="fa fa-folder"></i> Kategori 2
            </label>
        </li>
    </ul>
</div>
```

---

#### 12. Dosya Yöneticisi

```html
<!-- Dosya Yükleme -->
<div class="file-uploader">
    <div id="uploader">
        <div class="qq-upload-drop-area">
            <span>Dosyaları buraya sürükleyin</span>
        </div>
        <div class="qq-upload-button btn bg-teal btn-sm btn-raised">
            <i class="fa fa-upload"></i> Dosya Seç
        </div>
        <ul class="qq-upload-list"></ul>
    </div>
</div>

<!-- Yüklenen Dosyalar -->
<div class="uploaded-files">
    <div class="file-item">
        <i class="fa fa-file-image-o"></i>
        <span>dosya.jpg</span>
        <span class="file-size">125 KB</span>
        <button type="button" class="btn btn-xs bg-danger">
            <i class="fa fa-trash"></i>
        </button>
    </div>
</div>
```

---

#### 13. WYSIWYG Editör (TinyMCE)

```php
<?php
// Editör JS Dosyaları
$js_array[] = "public/editor/tiny_mce/tinymce.min.js";
?>

<script>
// Editör Başlatma
tinymce.init({
    selector: "textarea.editor",
    height: 400,
    language: "<?php echo $lang['language_code']; ?>",
    plugins: [
        "advlist autolink lists link image charmap print preview anchor",
        "searchreplace visualblocks code fullscreen",
        "insertdatetime media table contextmenu paste"
    ],
    toolbar: "insertfile undo redo | styleselect | bold italic | alignleft aligncenter alignright alignjustify | bullist numlist outdent indent | link image"
});
</script>
```

---

### Admin Fonksiyonları

#### Temel Admin Fonksiyonları

```php
// Header gösterimi
echoheader( $icon, $title );

// Footer gösterimi
echofooter();

// Mesaj kutusu
msg( $type, $title, $text, $url = "" );

// Güvenlik hash kontrolü
if( !isset($_REQUEST['user_hash']) OR $_REQUEST['user_hash'] != $dle_login_hash ) {
    die( "Hacking attempt! User not found" );
}

// Log kaydı
$db->query( "INSERT INTO " . USERPREFIX . "_admin_logs
             (name, date, ip, action, extras)
             values ('".$db->safesql($member_id['name'])."', '{$_TIME}', '{$_IP}', '73', '{$name}')" );

// Yetki kontrolü
if( !$user_group[$member_id['user_group']]['admin_addnews'] ) {
    msg( "error", $lang['index_denied'], $lang['index_denied'] );
}

// Cache temizleme
clear_cache( array('news_', 'full_', 'comm_', 'rss' ) );

// Dosya izinleri
@chmod( $file, 0666 );
@chmod( $folder, 0777 );
```

---

### Admin CSS Sınıfları

#### Butonlar

```html
<!-- Temel Butonlar -->
<button class="btn bg-teal btn-sm btn-raised">Kaydet</button>
<button class="btn bg-danger btn-sm btn-raised">Sil</button>
<button class="btn bg-blue btn-sm btn-raised">Düzenle</button>
<button class="btn bg-grey btn-sm btn-raised">İptal</button>

<!-- İkonlu Butonlar -->
<button class="btn bg-teal btn-sm btn-raised">
    <i class="fa fa-save position-left"></i> Kaydet
</button>

<!-- Dropdown Buton -->
<div class="btn-group">
    <button type="button" class="btn btn-sm bg-teal dropdown-toggle" data-toggle="dropdown">
        İşlemler <span class="caret"></span>
    </button>
    <ul class="dropdown-menu">
        <li><a href="#"><i class="fa fa-edit"></i> Düzenle</a></li>
        <li><a href="#"><i class="fa fa-trash"></i> Sil</a></li>
    </ul>
</div>
```

#### Badge ve Label

```html
<!-- Badge -->
<span class="badge bg-blue">5</span>
<span class="badge bg-success">10</span>
<span class="badge bg-danger">3</span>

<!-- Label -->
<span class="label label-success">Aktif</span>
<span class="label label-warning">Beklemede</span>
<span class="label label-danger">Pasif</span>
```

#### Form Elemanları

```html
<!-- Uniform Select -->
<select class="uniform" data-width="100%">
    <option>Seçenek</option>
</select>

<!-- Checkbox -->
<div class="checkbox">
    <label>
        <input type="checkbox" class="styled" />
        Checkbox
    </label>
</div>

<!-- Switchery -->
<div class="checkbox checkbox-switchery switchery-sm">
    <label>
        <input type="checkbox" checked />
        Switch
    </label>
</div>
```

---

### Admin JavaScript Fonksiyonları

```javascript
// Yükleme Göster
ShowLoading('Mesaj');

// Yükleme Gizle
HideLoading();

// Bildirim
DLEPush.success('Başarılı');
DLEPush.error('Hata');
DLEPush.warning('Uyarı');
DLEPush.info('Bilgi');

// Onay
DLEconfirm('Emin misiniz?', 'Başlık', function() {
    // Onaylandı
});

// Prompt
DLEprompt('Değer girin:', '', 'Başlık', function(value) {
    // Değer
});

// AJAX
$.post('index.php?controller=ajax&mod=module', {
    action: 'save',
    user_hash: dle_login_hash
}, function(data) {
    // Başarılı
}, 'json');
```

---

### Admin Güvenlik Standartları

#### 1. Yetki Kontrolü

```php
// Admin yetkisi kontrolü
if( !$user_group[$member_id['user_group']]['admin_addnews'] ) {
    msg( "error", $lang['index_denied'], $lang['index_denied'] );
}

// Super Admin kontrolü
if( $member_id['user_group'] != 1 ) {
    msg( "error", $lang['opt_denied'], $lang['opt_denied'] );
}
```

#### 2. CSRF Koruması

```php
// Hash kontrolü
if( !isset($_REQUEST['user_hash']) OR $_REQUEST['user_hash'] != $dle_login_hash ) {
    die( "Hacking attempt! User not found" );
}

// Formda hash
<input type="hidden" name="user_hash" value="<?php echo $dle_login_hash; ?>" />
```

#### 3. Input Temizleme

```php
// String temizleme
$value = trim( htmlspecialchars( stripslashes( $_POST['value'] ), ENT_QUOTES, 'UTF-8' ) );

// Translit
$value = totranslit( $_POST['value'], true, false );

// Integer
$id = intval( $_REQUEST['id'] );

// SQL Safe
$value = $db->safesql( $_POST['value'] );
```

#### 4. Log Kaydı

```php
// Admin log
$db->query( "INSERT INTO " . USERPREFIX . "_admin_logs
             (name, date, ip, action, extras)
             values ('".$db->safesql($member_id['name'])."', '{$_TIME}', '{$_IP}', '73', '{$name}')" );
```

---

### Admin Menü Yapısı

```php
// Menü opsiyonları
$options = array(
    'config' => array(
        array(
            'name' => 'Genel Ayarlar',
            'url' => '?mod=options&action=syscon',
            'descr' => 'Sistem ayarlarını yapılandırın',
            'image' => 'tools.png',
            'access' => 'admin'
        ),
        array(
            'name' => 'Kategoriler',
            'url' => '?mod=categories',
            'descr' => 'Kategori yönetimi',
            'image' => 'cats.png',
            'access' => $user_group[$member_id['user_group']]['admin_categories']
        )
    ),
    'user' => array(
        // Kullanıcı menüleri
    ),
    'templates' => array(
        // Template menüleri
    ),
    'filter' => array(
        // Filtre menüleri
    ),
    'others' => array(
        // Diğer menüler
    )
);
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

```
https://siteadi.com/kategori/2024/01/15/haber-adi.html
https://siteadi.com/?newsid=123
```

**Kullanılan Template Değişkenleri:**

```
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

```
https://siteadi.com/
https://siteadi.com/kategori/
https://siteadi.com/page/2/
```

**Kullanılan Template Değişkenleri:**

```
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

```
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
$.post(dle_root + "index.php?controller=ajax&mod=favorites", {
    id: news_id,
    action: "add",
    user_hash: dle_login_hash
}, function(data) {
    $(".add-favorites").hide();
    $(".del-favorites").show();
}, "json");

// Favorilerden sil
$.post(dle_root + "index.php?controller=ajax&mod=favorites", {
    id: news_id,
    action: "delete",
    user_hash: dle_login_hash
}, function(data) {
    $(".del-favorites").hide();
    $(".add-favorites").show();
}, "json");
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

```
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
// ✅ DOĞRU FORMAT
$.get(dle_root + "index.php?controller=ajax&mod=comments", {
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
// ✅ DOĞRU FORMAT
$.post(dle_root + "index.php?controller=ajax&mod=addcomments", {
    id: news_id,
    text: comment_text,
    name: user_name,
    email: user_email,
    user_hash: dle_login_hash
}, function(data) {
    if (data.success) {
        // Yorum başarıyla eklendi
        DLEPush.success("Yorum eklendi");
    } else {
        // Hata mesajı
        DLEPush.error(data.error);
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
$.post(dle_root + "index.php?controller=ajax&mod=favorites", {
    id: news_id,
    action: "add",
    user_hash: dle_login_hash
}, function(data) {
    $(".add-favorites").hide();
    $(".del-favorites").show();
}, "json");

// Favorilerden sil
$.post(dle_root + "index.php?controller=ajax&mod=favorites", {
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
    $.post(dle_root + "index.php?controller=ajax&mod=rating", {
        id: news_id,
        type: type,
        user_hash: dle_login_hash
    }, function(data) {
        if (data.success) {
            updateRatingDisplay(news_id, data.rating);
        } else {
            DLEPush.error(data.error);
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
    url: dle_root + "index.php?controller=ajax&mod=upload",
    type: "POST",
    data: formData,
    processData: false,
    contentType: false,
    success: function(data) {
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
        $.get(dle_root + "index.php?controller=ajax&mod=search", {
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
// ✅ DOĞRU FORMAT
$.post(dle_root + "index.php?controller=ajax&mod=profile", {
    action: "update",
    fullname: fullname,
    land: land,
    signature: signature,
    user_hash: dle_login_hash
}, function(data) {
    if (data.success) {
        DLEPush.success("Profil güncellendi");
    } else {
        DLEPush.error(data.error);
    }
}, "json");
```

---

### 8. registration.php - AJAX Kayıt

**Dosya Yolu:** `engine/ajax/registration.php`

**İşlevi:** AJAX ile kullanıcı kaydı.

**Kullanım:**

```javascript
// ✅ DOĞRU FORMAT
$.post(dle_root + "index.php?controller=ajax&mod=registration", {
    login: username,
    password: password,
    email: email,
    sec_code: captcha,
    user_hash: dle_login_hash
}, function(data) {
    if (data.success) {
        DLEPush.success("Kayıt başarılı");
    } else {
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

```
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

**ÖNEMLI:** DLE'de tüm AJAX istekleri `controller=ajax&mod=modul_adi` formatında yapılır!

```javascript
// ✅ DOĞRU KULLANIM
function saveData(id, data) {
    $.post(dle_root + "index.php?controller=ajax&mod=my_ajax", {
        action: "save",
        id: id,
        data: data,
        user_hash: dle_login_hash
    }, function(response) {
        if (response.success) {
            DLEPush.success("Başarıyla kaydedildi!");
        } else {
            DLEPush.error(response.error);
        }
    }, "json");
}

### DLEPush Kullanımı

**ÖNEMLİ:** DLEPush fonksiyonları **tek parametre** alır (mesaj string):

```javascript
// ✅ DOĞRU - Tek parametre (mesaj string)
DLEPush.success('Başarı mesajı');
DLEPush.error('Hata mesajı');
DLEPush.warning('Uyarı mesajı');
DLEPush.info('Bilgi mesajı');

// ❌ YANLIŞ - Objekt ile kullanılmaz
DLEPush.success({ message: 'Mesaj' });
DLEPush.error({ error: 'Hata' });
```

### AJAX Örnekleri

```javascript
// ✅ GET İsteği Örneği
$.get(dle_root + "index.php?controller=ajax&mod=comments", {
    news_id: news_id,
    cstart: cstart
}, function(data) {
    $("#dle-ajax-comments").html(data.comments);
}, "json");

// ✅ Favori İşlemleri
$.post(dle_root + "index.php?controller=ajax&mod=favorites", {
    id: news_id,
    action: "add",
    user_hash: dle_login_hash
}, function(data) {
    $(".add-favorites").hide();
    $(".del-favorites").show();
}, "json");

// ✅ Oy Verme
$.post(dle_root + "index.php?controller=ajax&mod=rating", {
    id: news_id,
    type: "plus",
    user_hash: dle_login_hash
}, function(data) {
    if (data.success) {
        updateRatingDisplay(news_id, data.rating);
    } else {
        DLEPush.error(data.error);
    }
}, "json");

// ✅ Dosya Yükleme
var formData = new FormData();
formData.append('controller', 'ajax');
formData.append('mod', 'upload');
formData.append('user_hash', dle_login_hash);
formData.append('files', fileInput.files[0]);

$.ajax({
    url: dle_root + "index.php?controller=ajax&mod=upload",
    type: "POST",
    data: formData,
    processData: false,
    contentType: false,
    success: function(data) {
        console.log(data.url);
    }
});

// ✅ Arama
$("#search-input").on("input", function() {
    var query = $(this).val();

    if (query.length > 2) {
        $.get(dle_root + "index.php?controller=ajax&mod=search", {
            story: query
        }, function(data) {
            $("#search-results").html(data);
        });
    }
});

// ✅ Profil Güncelleme
$.post(dle_root + "index.php?controller=ajax&mod=profile", {
    action: "update",
    fullname: fullname,
    land: land,
    signature: signature,
    user_hash: dle_login_hash
}, function(data) {
    if (data.success) {
        DLEPush.success("Profil güncellendi");
    } else {
        DLEPush.error(data.error);
    }
}, "json");

// ✅ Kayıt
$.post(dle_root + "index.php?controller=ajax&mod=registration", {
    login: username,
    password: password,
    email: email,
    sec_code: captcha,
    user_hash: dle_login_hash
}, function(data) {
    if (data.success) {
        DLEPush.success("Kayıt başarılı");
    } else {
        $.each(data.errors, function(key, value) {
            $("#" + key + "-error").text(value);
        });
    }
}, "json");

// ✅ Yorum Ekleme
$.post(dle_root + "index.php?controller=ajax&mod=addcomments", {
    id: news_id,
    text: comment_text,
    name: user_name,
    email: user_email,
    user_hash: dle_login_hash
}, function(data) {
    if (data.success) {
        DLEPush.success("Yorum eklendi");
        $("#dle-ajax-comments").prepend(data.comment);
    } else {
        DLEPush.error(data.error);
    }
}, "json");
```

### AJAX Modül Parametreleri

| Parametre | Açıklama | Örnek |
|-----------|----------|-------|
| `controller` | AJAX yönlendirici | `ajax` |
| `mod` | Modül adı | `favorites`, `rating`, `comments` |
| `user_hash` | Güvenlik token | `dle_login_hash` |
| `action` | İşlem tipi | `add`, `delete`, `update` |
| `id` | Kayıt ID | `123` |

### Mevcut AJAX Modülleri

```
engine/ajax/
├── addcomments.php      # Yorum ekleme
├── adminfunction.php    # Admin fonksiyonları
├── allvotes.php         # Tüm oylar
├── antivirus.php        # Antivirüs tarama
├── clean.php            # Temizlik
├── comments.php         # Yorum listeleme
├── commentssubscribe.php # Yorum aboneliği
├── complaint.php        # Şikayet
├── controller.php       # AJAX yönlendirici
├── deletecomments.php   # Yorum silme
├── editcomments.php     # Yorum düzenleme
├── editnews.php         # Haber düzenleme
├── emotions.php         # Emoticonlar
├── favorites.php        # Favoriler
├── feedback.php         # Geri bildirim
├── find_relates.php     # İlgili haberler
├── find_tags.php        # Etiket arama
├── keywords.php         # Anahtar kelimeler
├── message.php          # Mesaj
├── newsletter.php       # Bülten
├── plugins.php          # Eklentiler
├── pm.php               # Özel mesaj
├── poll.php             # Anket
├── profile.php          # Profil
├── quote.php            # Alıntı
├── rating.php           # Oy verme
├── rebuild.php          # Yeniden oluştur
├── registration.php     # Kayıt
├── replycomments.php    # Yorum cevabı
├── rss.php              # RSS
├── search.php           # Arama
├── templates.php        # Template
├── twofactor.php        # 2FA
├── updates.php          # Güncellemeler
├── upload.php           # Dosya yükleme
└── vote.php             # Anket oy
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
- **DLE Türkçe Destek:** https://dlehub.com.tr/

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
