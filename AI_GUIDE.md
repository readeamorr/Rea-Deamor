# Aethermoor: Kızıl Ejder — AI Çalışma Rehberi

Bu dosya, oyunu geliştirecek yapay zekâ (ve sen) için hazırlandı. **Önce bunu oku, sonra yalnızca ilgili modülü aç.**

## Oyun nedir
- Tek sayfalık, **bağımlılıksız** (kütüphane/CDN yok) piksel-sanat 2D RPG + çiftlik simülasyonu. Tek `<canvas id="g">` + DOM arayüz katmanları. Mobil Chrome öncelikli (joystick) ve klavye (WASD).
- Saf JavaScript, `'use strict'`, **tek bir IIFE** içinde (`(()=>{ ... })();`). Tüm modüller aynı kapsamı paylaşır; modüller arası `import` yok.
- Kayıt: `localStorage` anahtarı `aethermoor_px_v2` (+ yedek anahtarları). Sürümlü ve doğrulamalı kayıt sistemi `36_kayit_...` dosyasında.
- Ana nesneler: `P` (oyuncu, `window.P`), `S` (kayıt/durum), `WORLD_DATA`, `CROPS`, `CLASSES`, `MATERIALS`, `FARM`, `TOWNSFOLK`, `BUILD_CONFIG` vb.
- Başlatma: `startGame()` → `36_kayit_sistemi_surumlu_dogrulanmis_geri.js`. Ana döngü: `loop(t)` → `36_kayit_sistemi_surumlu_dogrulanmis_geri.js`; her karede `update(dt)` → `17_guncelleme_dongusu.js` ve `draw()` → `20_nesne_detayi_golgelendirilmis_yaprak_k.js`.

## Neden önizlenemiyordu (büyük olasılıkla)
Dosya tek parça **~1 MB / ~14.800 satır** (kabaca 300 bin+ token). Arena benzeri araçlar bunu ne tam okuyabilir ne de geri yazabilir; kesilince önizleme bozuluyor. Ayrıca dosyanın sonunda hosting'in eklediği 2 adet Cloudflare `<script>` vardı; korumalı (sandbox) önizlemede `SecurityError` veriyordu. Bu paket ikisini de çözer: oyun **modüllere bölündü**, Cloudflare kodları **temizlendi**.

## Dosya yapısı
```
src/00_head.html      <head> başı + <style>
src/style.css         tüm CSS
src/01_body.html      </style> … <body> arayüz HTML'i … <script>
src/js/NN_*.js        oyun kodu, SIRAYLA birleşen parçalar (aşağıdaki tablo)
src/js/zz_footer.js   IIFE kapanışı
build.py              parçaları birleştirir → dist/index.html
dist/index.html       oyunun hazır, tek dosyalık sürümü (önizleme için bunu aç)
```
`python3 build.py` çalıştır → `dist/index.html` üretilir ve JS sözdizimi kontrol edilir.
**Önemli:** `src/js/*.js` dosyaları tek başına çalışmaz; hepsi aynı IIFE'nin parçalarıdır. Parçaların başına/sonuna `(()=>{` ya da `})()` ekleme.

## Modül haritası (sırası değiştirilmez)
Başlıklar orijinal yorum bannerlarından alındı; içerik başlıkla her zaman birebir örtüşmeyebilir (ör. `19_vfx_sonu` aslında çizim kodu içerir). Sütunlar: dosya, boyut, satır, başlık, içindeki başlıca fonksiyon/sabitler.

| Dosya | Boyut | Satır | Başlık | Başlıca isimler |
|---|---|---|---|---|
| `00_cekirdek_yardimcilar.js` | 1 KB | 31 | Çekirdek: yardımcılar | playChord |
| `01_diyarlar_ve_coklu_harita_sistemi.js` | 44 KB | 796 | DİYARLAR VE ÇOKLU HARİTA SİSTEMİ (WORLD, HOME) | WORLD_DATA, GUIDE_ROUTE, guideIs, guideNow, navigationTarget, navigationCue, MAP_COLORS, LAKE_DOCK, SEA_PORT … |
| `02_adim_3_kasaba_parki_mozaik_bahce.js` | 29 KB | 605 | ADIM 3: KASABA PARKI + MOZAİK BAHÇE (harita üretimi) | buildTownPark, initHomeMap, buildHomeAtlas, initMineFloor, revealMineLadder, worldEntrySide, switchMap, bl, mv |
| `03_esyalar_ve_ekipman_sistemi.js` | 3 KB | 65 | EŞYALAR VE EKİPMAN SİSTEMİ | RAR, PASSIVES, gen, upgradeMultiplier, FISH, FISH_RARITY |
| `04_miktarli_malzemeler_tek_sayma_tuketme_.js` | 11 KB | 221 | MİKTARLI MALZEMELER · TEK SAYMA / TÜKETME / TAŞIMA KURALI | BAG_CAPACITY, MATERIALS, ITEMS, countInv, inventoryRecipe, inventoryWarning, tryInventoryRecipe, normalizeInventories, awardInventoryItem … |
| `05_oyuncu_ve_sinif_sistemi.js` | 4 KB | 92 | OYUNCU VE SINIF SİSTEMİ | CLASSES |
| `06_gorevler_ve_basarimlar.js` | 25 KB | 221 | GÖREVLER VE BAŞARIMLAR | CHAPTERS, chapterAt, CHAPSCENES, QUEST_META, QUEST_FLOW, closeQuestPrep, closeChapterComplete, closeChapterBrief, qp … |
| `07_dusmanlar_ve_mini_bosslar.js` | 29 KB | 632 | DÜŞMANLAR VE MİNİ-BOSSLAR | ENCOUNTER_VARIANTS, go, doParry, hurtP, hitMob, drop, bossAI, BOSS_SYSTEM, hitCritter … |
| `08_adim_4_ciftlik_otomasyonu.js` | 12 KB | 220 | ADIM 4: ÇİFTLİK OTOMASYONU (Golem, Akıllı Sevkiyat, Tedarik, Posta Güvercini) | AUTO |
| `09_adim_3_park_bahce_piknik_oturma.js` | 6 KB | 113 | ADIM 3: PARK / BAHÇE / PİKNİK / OTURMA | TOWN_PARK |
| `10_adim_2_yasayan_kasaba_isler_gelir_gunl.js` | 19 KB | 290 | ADIM 2: YAŞAYAN KASABA — İŞLER, GELİR, GÜNLÜK RUTİNLER | NPC_NEEDS_CONFIG, TOWN_LIFE, updNpcs, TOWN_CHAT, chat, NOTIFICATIONS, MODAL_QUEUE, ban, advanceGuide |
| `11_savas_mekanikleri.js` | 5 KB | 104 | SAVAŞ MEKANİKLERİ | DAY_CLOCK, addXp, near, selectTool, updHotbarUI |
| `12_yeni_ciftlik_sistemi_takvim_4_mevsim_2.js` | 71 KB | 1187 | YENİ ÇİFTLİK SİSTEMİ: Takvim, 4 Mevsim, 22 Ekin, Kalite, Gübre, Tohum Kesesi | SEASONS, CROPS, FARM_BOUNDS, GARDEN_BEDS, ORCHARD_TREES, ORCHARD_SLOTS, SOIL_UPGRADES, SPRINKLER_UPGRADES, FARM |
| `13_yeni_hayvancilik_sistemi_kumes_ahir_6_.js` | 21 KB | 418 | YENİ HAYVANCILIK SİSTEMİ: Kümes, Ahır, 6 Hayvan Türü, Sevgi, Yem, Ürün Kalitesi | HTYPES, HPENS, HERD, T2, harvestPlot, plantSeedInPlot, quickEatFood, cycleTool, useActiveTool |
| `14_savas_modulu_okunabilir_uyarilar_ilk_s.js` | 18 KB | 431 | SAVAŞ MODÜLÜ: okunabilir uyarılar, ilk savaş dersi ve hedef durumu | closeCombatGuide, openCombatGuide, replayCombatGuide, COMBAT, attack, showCombo, doHit, skill1, skill2 … |
| `15_demirci_ve_gelistirme_sistemi.js` | 12 KB | 248 | DEMİRCİ VE GELİŞTİRME SİSTEMİ | FORGE, forgeUI, updateForgeDetail, priestUI, shopUI, achUI, talent, death, waterDir … |
| `16_girdi_sistemi.js` | 7 KB | 157 | GİRDİ SİSTEMİ | — |
| `17_guncelleme_dongusu.js` | 25 KB | 542 | GÜNCELLEME DÖNGÜSÜ | restAtWorldOutpost, claimLakeBait, tradeSeaFish, nearFrontierCamp, buildFrontierCamp, restFrontierCamp, CAMP_UPGRADES, campStorageCapacity, campItemCount … |
| `18_aethermoor_vfx_havuzlu_gorsel_motor.js` | 29 KB | 396 | AETHERMOOR VFX · HAVUZLU GÖRSEL MOTOR | — |
| `19_vfx_sonu.js` | 30 KB | 641 | VFX SONU | DRAW, PAL3, LBL, WLAB, T, labelNear, outline, spr, hero … |
| `20_nesne_detayi_golgelendirilmis_yaprak_k.js` | 27 KB | 581 | NESNE DETAYI: gölgelendirilmiş yaprak kümeleri ve çam katmanları | lobe, pineTier, objRaw, critterD, resD, houseD, houseRaw, fountain, paintCottage … |
| `21_arayuz_ve_menuler.js` | 52 KB | 733 | ARAYÜZ VE MENÜLER | updateCombatTarget, ui, inventoryMatches, equipmentScore, equipmentComparison, equipBestItems, invUI, sellBagItem, openInv … |
| `22_ciftlik_ev_ve_gundelik_yasam_fonksiyon.js` | 3 KB | 72 | ÇİFTLİK, EV VE GÜNDELİK YAŞAM FONKSİYONLARI | sleepInBed, updHomeAnimals |
| `23_hayvancilik_cizim_fonksiyonlari.js` | 32 KB | 597 | HAYVANCILIK ÇİZİM FONKSİYONLARI (KÜMES, TAVUKLAR, İNEK) | chickenD, cowD, sheepD, eggD, ladderD, elevatorD, coopD, drawWaypointGuide, gardenCampusGroundD … |
| `24_kasaba_siparis_panosu_ve_itibar_sistem.js` | 3 KB | 67 | KASABA SİPARİŞ PANOSU VE İTİBAR SİSTEMİ | ORDER_POOL, getRepTier, initOrders, boardUI |
| `25_ev_kileri_ve_yayla_deposu_ayri_envante.js` | 3 KB | 54 | EV KİLERİ VE YAYLA DEPOSU: AYRI ENVANTER, AYNI GÜVENLİ ARAYÜZ | activeStorageBox, storageUI, quickDepositStorage |
| `26_sadik_yoldas_ve_otomasyon_sistemi.js` | 7 KB | 173 | SADIK YOLDAŞ (PET) VE OTOMASYON SİSTEMİ | petD, sprinklerD, updPet, petUI |
| `27_yasam_becerileri_ustaligi.js` | 6 KB | 141 | YAŞAM BECERİLERİ USTALIĞI (LIFE SKILLS MASTERY) | MASTERY_INFO, addMasteryXp, masteryUI, updWeather |
| `28_evin_sicak_ic_mekani_ve_retro_eglence_.js` | 9 KB | 199 | EVİN SICAK İÇ MEKANI VE RETRO EĞLENCE SİSTEMİ | MUSIC_TRACKS, initInsideMap, insideObjD, tvUI, gramophoneUI |
| `29_kasabalilarla_dostluk_ve_iliski_sistem.js` | 21 KB | 286 | KASABALILARLA DOSTLUK VE İLİŞKİ SİSTEMİ | TOWNSFOLK, NPC_BOND_STORIES, NPC_PERSONAL_REQUESTS, friendshipStory, npcLifeState, FRIEND_WORLD_NAMES, friendPosition, nearFriend, trackFriend … |
| `30_ortacag_kasabasi_ve_carsi_cizim_fonksi.js` | 8 KB | 188 | ORTAÇAĞ KASABASI VE ÇARŞI ÇİZİM FONKSİYONLARI (REFERANS GÖRSELLER BİREBİR) | marketStallD, chapelD, campfireD, cartD, wellD, watermillD, fruitTreeD, lampD, duckD … |
| `31_canli_konusma_balonlari_ve_kayit_payla.js` | 14 KB | 317 | CANLI KONUŞMA BALONLARI VE KAYIT PAYLAŞIM SİSTEMİ | BOT_CHATTER, say, updBubbles, drawPixelTextDirect, bubbleD, sortInventory, openSaveModal, showSavedCode, millUI … |
| `32_canavar_ansiklopedisi_ve_ayarlar.js` | 20 KB | 149 | CANAVAR ANSİKLOPEDİSİ (BESTIARY) VE AYARLAR | BESTIARY, bestiaryUI, WORLD_EVENTS, DAY_RECAP, closeDayRecap, SESSION_COACH, GAME_BALANCE, CONTROLS, haptic … |
| `33_ekosistem_ve_islevsel_bina_sistemleri.js` | 17 KB | 326 | EKOSİSTEM VE İŞLEVSEL BİNA SİSTEMLERİ (ŞAPEL, TAVERNA, BELEDİYE) | chapelUI, MERCS, tavernUI, updMerc, mercD, townhallUI, openAch, rewardsUI, MENU_LABELS … |
| `34_yasayan_dunya_ortak_ilerleme_ekonomi.js` | 97 KB | 963 | YAŞAYAN DÜNYA · ORTAK İLERLEME / EKONOMİ | ADVENTURE, DAILY_FOCUS, FEATURES, closeFeatureUnlock, ECONOMY, GOALS, EXPEDITIONS, JOURNAL, BUILD_CONFIG … |
| `35_genisleme_modulleri.js` | 102 KB | 996 | GENİŞLEME MODÜLLERİ | GAME_CORE, ONBOARD, guideChapterCard, guideUI, openGuide, STORY, DECISIONS, migrateSeedPouches, addPetXp … |
| `36_kayit_sistemi_surumlu_dogrulanmis_geri.js` | 25 KB | 397 | KAYIT SİSTEMİ · SÜRÜMLÜ / DOĞRULANMIŞ / GERİ ALINABİLİR | LK_CORRUPT, LK_FOUNDATION_BACKUP, SAVE_VERSION, SAVE_KEYS, UNSAFE_KEYS, SAVE_MIGRATIONS, readStore, saveNotice, saveNoticeCurrent … |

Büyük dosyalar (>70 KB): `12_yeni_ciftlik…`, `34_yasayan_dunya…`, `35_genisleme_modulleri…`. Bunları okurken `grep -n "function ADI"` ile tek fonksiyona atla; tamamını bağlama yükleme.

## AI için çalışma kuralları
1. **Dosyanın tamamını yeniden yazma.** Sadece değişecek modülü aç; çıktıda yalnızca değişen fonksiyonları / yeni dosyaları ver.
2. Yeni özellik = yeni modül dosyası (`NN_ad.js`, zz_footer'dan önce gelecek bir numara) veya mevcut modülde küçük ekleme. Yeni dosya IIFE kapsamında çalışır, `P`, `S`, `$`, `rnd`, `ri`, `pick`, `cl` yardımcılarını doğrudan kullanır.
3. Dış kütüphane, CDN, `fetch`, ağ isteği **ekleme**; her şey tek HTML'de çalışmalı.
4. Mobil öncelikli kal: dokunmatik joystick, küçük ekran, 60 FPS. Ağır döngü / her karede nesne üretmekten kaçın (VFX havuz sistemi kullanır).
5. Kayıt uyumluluğunu bozma: `S` içine alan eklerken eski kayıtta yoksa varsayılan ver; yeni `localStorage` anahtarı ekleme, mevcut kayıt sistemini kullan.
6. Sandbox önizlemede `localStorage` engellenir; **"Kayıt yazılamıyor" uyarısı beklenen davranıştır**, hata değildir.
7. Değişiklikten sonra: `python3 build.py` → `dist/index.html` aç → konsolda hata yok mu bak.
8. Dil: arayüz metinleri Türkçe.

## Arena'ya yapıştırılacak başlangıç istemi
`ARENA_PROMPT.txt` dosyasına bak.
