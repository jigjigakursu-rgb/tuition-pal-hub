# Veriyi Lovable Cloud'a taşıma

Panelin görünümü, renkleri, yerleşimi ve tüm özellikleri aynı kalır. Değişen tek şey: kayıtlar artık uygulamanın kendi bulut veritabanında tutulur. Giriş ekranı eklenmez, sistem boş başlar.

## Ne yapılacak

1. **Bulut veritabanı açılır** ve iki kayıt alanı oluşturulur:
   - **Talebeler**: isim, grup, sınıf, telefon, doğum, notlar, fotoğraf, sayfa, haftalık hedef, kıraat/fıkıh/hadis takip günleri, geçmiş, aidat ödemeleri.
   - **Ayarlar**: aidat tutarı, grup ve mesul hoca listesi, hoca e-postaları, gönderen bilgisi, gönderilmiş hatırlatma kayıtları.
2. **Veri katmanı değiştirilir.** Uygulamadaki tüm ekranlar zaten tek bir ortak veri kapısından besleniyor; sadece o kapının arkası yeniden yazılır. Ekran dosyalarına dokunulmaz.
3. **Eski bağlantı kaldırılır** (Firebase paketi ve ayar dosyası silinir).
4. Ekleme, düzenleme, silme, toplu hedef, aidat işaretleme, grup kaydetme, PDF/Excel çıktıları ve hatırlatma mailleri aynı şekilde çalışmaya devam eder.

## Hız ve güvenlik kuralları

- Açılışta son bilinen liste anında gösterilir, güncel veri arka planda gelir; ekran donmaz, iskelet/yükleniyor durumu korunur.
- Aynı veri için tek bir ortak sorgu kullanılır; bileşenler bunu paylaşır, gereksiz tekrar sorgu yapılmaz.
- Sorgularda yalnızca gereken sütunlar istenir; "hepsini getir" kullanılmaz.
- Sık filtrelenen alanlara (grup, isim, sıra, aidat durumu) veritabanı indeksleri eklenir.
- Büyük listeler 1000'erlik parçalar hâlinde çekilir.
- Canlı güncelleme tek bir kanaldan yürütülür ve ekran kapanınca kapatılır.
- Ayarlar tek kayıttan okunur; liste içinde tek tek sorgu yapılmaz.
- Satır güvenliği açık tutulur, izinler açıkça tanımlanır; hata olursa uygulama çökmez, elindeki son veriyle devam eder ve kullanıcıya kibar uyarı gösterilir.

## Teknik detaylar

- `supabase--enable` ile Cloud açılır. Migration: `public.talebeler` (uuid pk, ölçülebilir alanlar sütun, gün haritaları/geçmiş/aidat `jsonb`) ve `public.ayarlar` (tek satır, `id text pk 'genel'`).
- Her tabloya `GRANT`lar + RLS açık; giriş olmadığı için `anon` ve `authenticated` rollerine SELECT/INSERT/UPDATE/DELETE politikaları tanımlanır.
- İndeksler: `talebeler(grup)`, `talebeler(isim)`, `talebeler(sira)`, `talebeler(aidat_sadece)`, `talebeler(aidat_haric)`.
- Yeni `src/lib/talebelerSupabase.ts`, `talebelerFirestore.ts` ile **birebir aynı dışa aktarım imzasını** sağlar; `src/lib/talebeler.ts` façade'ı lazy import hedefini bu dosyaya çevirir. `talebelerTipler.ts`, `yerelCache.ts`, `use-gruplar.ts` ve tüm bileşenler değişmez.
- Ortak abonelik + localStorage önbellek + iyimser güncelleme mantığı korunur; Firestore `onSnapshot` yerine tek `supabase.channel` postgres_changes aboneliği, abone kalmayınca kapatılır.
- Okumalarda açık sütun projeksiyonu, `.range()` ile sayfalı çekim, Türkçe sıralama istemcide.
- `src/lib/firebase.ts` ve `src/lib/talebelerFirestore.ts` silinir, `firebase` paketi kaldırılır.
- `bunx tsgo --noEmit` ve tarayıcıda ekleme/düzenleme/silme akışı doğrulanır.
