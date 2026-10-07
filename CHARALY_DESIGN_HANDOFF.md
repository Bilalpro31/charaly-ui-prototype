# Charaly UI Handoff (v5)

Referans: `docs/design/charaly-app-design-v5.html` (tarayıcıda aç, her ekran üstteki sekmelerden gezilir). Bu dosya ile HTML çelişirse HTML kazanır, ama aşağıdaki "Kurallar" bölümü her zaman geçerlidir.

## 0. Kurallar (pazarlık yok)
1. **Engine, JNI, llama.cpp, ModelRegistry, ModelManager, EventEngine, WorldState, CharacterRuntime koduna dokunma.** Sadece `ui/` altı: screens, sheets, components, design, nav, shell.
2. **UI hikâye gösterir, veritabanı değil.** Ekranda id, skor, tier adı, event tipi, model adı (Models/Me dışında), token, backend adı görünmez. `ProductPresentationTest` bunu string taramasıyla doğruluyor, geçmeye devam etmeli.
3. **Pack detayı** yalnızca: kapak, başlık, evren/dönem, premise (tek paragraf), 2-3 atmosferik cümle, ton/atmosfer etiketleri, Enter Story. `PackShowcase` tipine karakter/lokasyon/event alanı ekleme.
4. **Dürüstlük:** ölçülmemiş hız "Not measured" yazar. İndirme düğmesi olmayan satırda (Charaly knows about) düğme olmaz. Boş onClick yok.
5. Her yeni ekran için mevcut test stili: presenter/state testleri, boş/hata/yükleniyor durumları.

## 1. Tasarım dili
**Hissi:** modern, düz, koyu. c.ai kalitesinde ama kişi değil dünya odaklı. Kart çorbası yok, çok border yok.

**Renk token'ları** (`DesignTokens.kt`)
| Token | Değer | Kullanım |
|---|---|---|
| bg | `#0F0F11` | ekran zemini |
| surface1 | `#1C1C1F` | kart, chip, input |
| surface2 | `#2B2B2F` | pill, arama çubuğu, pasif buton |
| text | `#F4F4F6` | ana metin |
| textMuted | `#9B9BA4` | meta, ikincil |
| line | beyaz %10 | ayırıcı |
| **amber** | `#F2B66D` | **dünya yaşıyor**: saat, Live, hafıza/ripple, konuşan karakter adı, ilerleme |
| **blue** | `#3B63FF` | **senin için yeni**: New rozeti, okunmamış nokta |
| onPrimary | `#F4F4F6` zemin, `#111` metin | ana buton (beyaz pill) |

Başka vurgu rengi ekleme. Amber = engine'in canlı sinyali, mavi = yeni. Bu ayrım tutarlı kalmalı.

**Tipografi:** Figtree (400/500/600/700, bundle et). Başlıklar 700, harf aralığı -2.5%. Ekran başlığı 30sp, kart başlığı 20sp, gövde 16sp, meta 13sp muted.

**Şekiller:** kart/kapak 24dp, sheet üst köşe 28dp, pill ve butonlar tam yuvarlak, avatar daire veya 11-18dp köşe. Gölge yok (sadece yüzen glass öğelerde hafif).

**Glass:** sohbet üstü öğeler yarı saydam koyu zemin + blur (`RenderEffect` API 31+, altında opak koyu yedek). Blur yoksa okunabilirlik bozulmamalı.

**Sahne sanatı:** pack başına vektör/Canvas sahne (gradient gökyüzü + silüet). HTML'deki `scene()` fonksiyonu referans. Mevcut `ui/art` paketini kullan, `Visuals can never fail to render` ilkesi (README) aynen sürsün.

## 2. Gezinme
Alt bar, **sadece ikon** (etiket yok), 5 öğe: Home, Sessions (mavi nokta = yeni), Create (+), Library, Me. Aktif olan beyaz (Home ve Me dolu). Sadece ana ekranlarda görünür; pack detayı, Enter Story, Stage, Authoring, CharacterImport, ModelDetail, Onboarding'de **gizli** (kendi geri düğmesi ve alt CTA var). Models, Me'nin alt ekranıdır (Me aktif kalır).

## 3. Ekran eşleştirmesi
| HTML sekmesi | Mevcut dosya | Not |
|---|---|---|
| Intro | `OnboardingScreen` | README'deki 3 cümle, tam ekran sahne, nokta ilerleme, Skip |
| Home | `HomeScreen` | logo "charaly", bildirim, arama pill, filtre pill'leri (aktif beyaz), **Continue** geniş kart (Live pill + saat), 2 sütun pack kartları |
| Library | `LibraryScreen` | Continue playing, Previously on… (recap posterleri), Story packs, People you've met. Bölüm başlığı + "More >" |
| Story pack | `ShowcaseScreen` | kural 3'e uy |
| Enter story | `EnterWorldScreen` | açılış (3 seçenek), rol, yanında kim var; alt CTA "Begin" |
| Story chat | `StageScreen` | aşağıda ayrıntı |
| World / Memory / People / Story | `StageSheets` | alttan sheet |
| Sessions | mevcut sessions listesi | oturum satırı: kapak, başlık, gün·saat·zaman, son replik italik, Live nokta |
| Create | yeni giriş ekranı | Write your own world, Import a character, Import a story pack |
| New world | `AuthoringScreen` | 3 adım: ad/tagline/premise, ton + ilk sahne, Feature this world |
| Import character | `CharacterImportScreen` | kartın tamamını önizle, sonra "Add to world" |
| Models / Model | `ModelHubScreen`, `ModelDetailScreen` | README'deki sıra: aktif model, canlı indirme, Hub arama+filtre, sonuçlar, repo dosyaları, On this device, Charaly knows about, Import GGUF |
| Me | `SettingsScreen` | Developer mode açılınca `DeveloperScreen` içeriği (saat adımlama) satır içinde |

## 4. Story chat (en önemli ekran)
- **Arka plan:** karakter portresi değil **mekân sahnesi**, tam ekran. Üstte ve altta koyu gradient (okunabilirlik).
- **Üst bar:** geri, başlık (tek satır, ellipsis), **saat pill'i** (amber, `22:40 ⌄`, World sheet'i açar), **beyin** ikonu (Memory sheet, yeni varsa mavi "New" rozeti), **!** ikonu (Story sheet, mavi nokta), **menü+✦** ikonu (Story controls: Pace, Scene length, Tone).
- **Sahne notu:** üstte küçük, düşük kontrastlı: pack adı + "Everything here is fiction".
- **Sağ ray (HUD):** yığılmış kişi avatarları (People), mekân pin'i (World), kum saati (Skip time), göz (What you know). **Sol:** dikey gerilim çubuğu (amber gradient), etiketi "tension".
- **HUD açık/kapalı** Settings'te seçenek; kapalıyken ray ve gerilim çubuğu gizli, mesaj alanı tam genişlik.
- **Mesajlar:** koyu yarı saydam baloncuk, 22dp köşe.
  - *Previously* baloncuğu en üstte (önceki bölüm özeti).
  - Anlatım: italik, avatarsız.
  - Karakter: 36dp yuvarlak köşe avatar solda, ad amber 13sp bold, metin normal. Avatara dokununca People→Person sheet.
  - Kullanıcı: sağda, açık yarı saydam baloncuk, başında mod etiketi (Do/Say/Think amber).
  - **Ripple etiketleri:** bir turdan sonra engine'in etkileri, hikâye diliyle, amber pill: "Marinette trusts you more", "Marinette will remember this", "New thread · The roof", "22:40 → 22:52". Teknik ifade yok.
- **Üretim durumu:** altta "The world is moving…" + üç nokta. Model/token/hız gösterme. Input sağındaki ✦ yerini Stop (kare) alır. Durdurulursa "↻ Retry".
- **Alt bar:** sol kontrol ikonu (mavi nokta), input pill (içinde mod seçici `Do ⌄`, ✦ öneri, yazınca beyaz ↑ gönder), sağ araç ızgarası (Do/Say/Think, Skip time, Rewind, People).
- ✦ öneri: 3 yatay chip (örn. "Ask Marinette what she saw"). Dokununca Do olarak gönderir.
- **Klavye:** `imePadding()`, input klavyenin hemen üstünde, hiçbir zaman ekran ortasında değil. Mesaj listesi altta hizalı (`reverseLayout`/`Arrangement.Bottom`).

### Sheet'ler (alttan, scrim ile, 28dp köşe)
- **World:** büyük saat (amber 52sp), "Thursday · Day 3", yer ve hava, gün yayı (sabah→gece, işaretçi), "Around you" (kişi + ruh hali + ne istiyor), "Elsewhere in Paris" (sen yokken olanlar), Rewind one turn, Recap.
- **Memory:** "What Paris remembers", kim hatırlıyor, yeni olanlar mavi nokta.
- **People:** sahnedeki kişiler; Person: ruh hali, "ne düşünüyor" çubuğu (Wary → Trusts you), ne istiyor, bu gece ne biliyor.
- **Story:** "What you're here to do": açık konular + verdiğin sözler (süresiyle).
- **Skip time:** +10 dk, +1 saat, Until morning. Saat engine'in `advance` yolundan ilerler (UI saati kendisi set etmez).
- **Story controls:** Pace, Scene length, Tone segment seçiciler.

## 5. Ortak bileşenler (`CharalyKit.kt`)
`SceneArt(packId)`, `LivePill`, `NewBadge`, `GlassPill`, `FilterPill`, `PackCard` (kare kapak + başlık + 2 satır açıklama + etiketler + sayaç/yaratıcı), `PosterCard` (3:4.1, sol altta ▶ sayaç), `SectionHeader(icon, title, more)`, `PrimaryButton` (beyaz pill), `AvatarStack`, `RippleChip`, `StoryBubble`, `ModeInput`, `SheetScaffold`, `StepDots`, `ToggleSwitch` (amber), `CompatLabel` (Runs well / Heavy, may be slow / Engine update required).

## 6. Çalışma sırası (her adım ayrı commit, her biri testle)
1. `DesignTokens` + Figtree + temel bileşenler (kit), önizleme (`@Preview`) ekranı.
2. Shell + alt bar + Home + Library.
3. Showcase + EnterWorld + Onboarding.
4. **StageScreen** (arka plan, üst bar, mesaj baloncukları, input, klavye davranışı) engine'e dokunmadan mevcut state'e bağlı.
5. StageSheets (World, Memory, People, Story, Skip, Controls).
6. Sessions, Create, Authoring, CharacterImport.
7. ModelHub + ModelDetail + Settings.
8. Cilalama: boş/hata durumları, erişilebilirlik (contentDescription, 48dp dokunma alanı, kontrast), `reduced motion`.

## 7. Kabul ölçütleri
- `./gradlew verify` ve `:app:testDebugUnitTest` yeşil, `ProductPresentationTest` dahil.
- Hiçbir ekranda model adı (Models/Me dışı), id, skor, tier, event tipi yok.
- Chat'te klavye açıkken input klavyenin hemen üstünde.
- Alt bar etiketsiz ama her ikonun `contentDescription`'ı var.
- Amber ve mavi dışında vurgu rengi yok.
- Cihazda test edilmedi ise raporda açıkça "cihazda test edilmedi" yaz.
