# Asiye — Knowledge Processing System

## Project Overview

Asiye is a multi-agent knowledge processing pipeline that transforms raw documents
(PDF, TXT, MD, etc.) placed in `ham-veriler/` into a structured, interconnected
Obsidian wiki vault at `asiye-obsidian/`.

Every detail from every source is captured, classified, cross-referenced, and made
meaningful. Nothing is forgotten.

## Pipeline Flow

The pipeline runs as 7 sequential agents. Each agent completes its stage, presents
results to the user, and waits for approval before the next agent begins.

```
ham-veriler/[document]
  → 1. Reader       → [onay]
  → 2. Scope        → [kullanıcı kapsam seçimi]
  → 3. Analyzer     → [onay]
  → 4. Classifier   → [onay]
  → 5. Relationship → [onay]
  → 6. Writer       → [onay]
  → 7. Discuss      → [onay/revize]
  → asiye-obsidian/
```

### Agent Descriptions

| # | Agent | Skill | Responsibility |
|---|-------|-------|---------------|
| 1 | Reader | `.opencode/skills/asiye-reader/SKILL.md` | Ham belgeyi okur, metni çıkarır, yapısal unsurları tespit eder, yapılandırılmış çıkarım üretir |
| 2 | Scope | `.opencode/skills/asiye-scope/SKILL.md` | Metni başlıklara/bölümlere/konulara ayırır, kullanıcıya kapsam seçenekleri sunar (tamamı, belirli bölümler, özel odak), Analyzer'a kapsam filtresi iletir |
| 3 | Analyzer | `.opencode/skills/asiye-analyzer/SKILL.md` | Kapsam filtresine göre metni ayrıştırır, tüm varlıkları tanımlar (kişi, kurum, yer, olay, tarih, kavram) |
| 4 | Classifier | `.opencode/skills/asiye-classifier/SKILL.md` | Varlıkları kategorize eder, vault klasör yapısına atar |
| 5 | Relationship | `.opencode/skills/asiye-relationship/SKILL.md` | Varlıklar arası ilişkileri haritalandırır, ilişki grafiği oluşturur |
| 6 | Writer | `.opencode/skills/asiye-writer/SKILL.md` | Obsidian wiki notları ve detaylı raporları tam bağlantılılık ile yazar |
| 7 | Discuss | `.opencode/skills/asiye-discuss/SKILL.md` | Akademik tartışma, yeni perspektifler, revizyon önerileri |

## Core Principles

### Detay Önceliklidir
- **Varsayılan: detaylı analiz.** Özet raporlar yalnızca açıkça istendiğinde oluşturulur.
- Kaynaktaki her bilgi parçası yakalanmalı ve sınıflandırılmalıdır.
- Hiçbir detay çok küçük değildir. Kaynakta varsa, vault'a girer.

### Türkçe Çıktı
- Tüm çıktı Türkçedir, kaynak dilinden bağımsız olarak.
- Yabancı dildeki isimler/terimler orijinal formda korunur ancak açıklamalar Türkçedir.

### Obsidian Wiki Formatı
- Tüm dahili bağlantılar için `[[wikilink]]` sözdizimi kullanılır.
- Her not; `tags`, `aliases`, `created`, `source` içeren YAML frontmatter'a sahiptir.
- Notlar tamamen birbirine bağlıdır — her varlık, ilgili her varlığa bağlanır.

### Her Adımda Kullanıcı Onayı
- Her agent çıktısını sunar ve kullanıcı onayı ister.
- Kullanıcı değişiklik talep ederse, agent ilerlemeden önce revize eder.
- Pipeline, açık kullanıcı onayı olmadan ilerlemez.

### Akademik Tartışma (Discuss Agent)
- Profesyonel bir tarihçi gibi davranır.
- Yeni perspektifler sunar, fikir dayatmaz.
- Kullanıcıyı pohpohlamaz.
- Kullanıcının materyali özümsemesine yardımcı olur.
- Raporlarda revizyon gerektiğinde, kullanıcı onayıyla önerir.

## Obsidian Vault Structure

```
asiye-obsidian/
├── 00-Giris/
│   └── _index.md              # Ana indeks, grafik genel bakış
├── 01-Kaynaklar/
│   └── [kaynak-adi].md        # Kaynak analiz raporları
├── 02-Kisiler/
│   └── [kisi-adi].md          # Kişi notları
├── 03-Kurumlar/
│   └── [kurum-adi].md         # Kurum/kuruluş notları
├── 04-Olaylar/
│   └── [olay-adi].md          # Olay notları
├── 05-Yerler/
│   └── [yer-adi].md           # Yer/konum notları
├── 06-Kronoloji/
│   └── [tarih-araligi].md     # Kronoloji notları
├── 07-Raporlar/
│   └── [rapor-adi].md         # Çapraz analiz raporları
└── 08-Templates/
    └── [template-adi].md      # Not şablonları
```

## Note Format Standard

Her not ŞU formatta olMALIDIR:

```yaml
---
title: "Not Başlığı"
aliases: ["Alternatif İsim", "Diğer Yazım"]
tags: [etiket1, etiket2, etiket3]
created: YYYY-MM-DD
source: "[[kaynak-adi]]"
---
```

### Wikilink Kuralları
- Tüm varlık referansları `[[wikilink]]` formatında olur.
- Bir notta bir varlığın ilk geçtiği yer her zaman wikilink'tir.
- Her notun altında ilgili varlıklar bölümü bulunur.
- Gerektiğinde görüntü metni kullanılır: `[[hedef|görüntü metni]]`

### Rapor Formatı
- Raporlar Türkçedir ve olabildiğince detaylıdır.
- Yapılandırılmış veriler için markdown tabloları içerir.
- Uygun yerlerde ilişki matrisleri bulunur.
- Wikilink'ler aracılığıyla diğer notlara çapraz referans verir.
- Her iddia için kaynak gösterimi yapılır.

## Quality Standards

1. **Tamlık:** Kaynaktaki her gerçek, isim, tarih, yer ve ilişki yakalanır.
2. **Doğruluk:** Uydurma bilgi yoktur. Yalnızca kaynakta olanlar yazılır.
3. **Bağlantılılık:** Her not, ilgili tüm notlara bağlanır.
4. **Okunabilirlik:** Raporlar iyi yapılandırılmış, tablolar, başlıklar ve net dil kullanır.
5. **Tutarlılık:** İsimlendirme kuralları vault genelinde tekdüzedir.

## Naming Conventions

- Dosya adları: küçük harf, tire ile ayrılmış, Türkçe karakterler korunur.
  - Örnekler: `mustafa-kemal-ataturk.md`, `buyuk-millet-meclisi.md`
- Kişi isimleri: `ad-soyad.md` formatı.
- Kurum isimleri: tam resmi ad, küçük harf.
- Olay isimleri: tanımlayıcı, küçük harf.
- Yer isimleri: güncel resmi ad, küçük harf.

## How to Start a New Processing Run

1. Kullanıcı belgeyi `ham-veriler/` klasörüne yerleştirir.
2. Kullanıcı OpenCode'a yeni belgeyi haber verir.
3. OpenCode, Reader agent'tan başlayarak pipeline'ı çalıştırır.
4. Reader metni çıkarır ve yapısal unsurları tespit eder.
5. Scope agent içeriği bölümlere ayırır ve kullanıcıya kapsam seçenekleri sunar:
   - **Tamamını işle:** Belgenin tüm içeriği analiz edilir.
   - **Belirli bölümleri seç:** Kullanıcı listeden bir veya birkaç bölüm seçer.
   - **Özel odak belirle:** Kullanıcı belirli bir konu, tema veya kişi belirtir.
6. Kullanıcının kapsam seçimine göre Analyzer agent çalışır.
7. Her agent `skill` tool'u ile kendi skill'ini yükler ve checklist'ini takip eder.

## External Skills and Tools

Bu proje, `C:\Users\kocak\.agents\skills\` dizininde bulunan harici skill'lerden faydalanır. Aşağıdaki skill'ler pipeline sırasında aktif olarak kullanılmalıdır:

### PDF İşleme (ZORUNLU)
- **Skill:** `pdf` (`C:\Users\kocak\.agents\skills\pdf\SKILL.md`)
- **Ne zaman:** Ham belge PDF formatında ise Reader agent bu skill'i yüklemelidir.
- **Nasıl:** `skill` tool'u ile `pdf` skill'ini yükle. PDF'den metin çıkarmak için `pdfplumber` (metin ve tablo çıkarımı), `pypdf` (meta veri ve temel işlemler) veya `pdftotext` (komut satırı) kullan. Taranmış PDF'ler için OCR (`pytesseract`) uygula.
- **Kural:** PDF skill'i YÜKLENMEDEN Reader agent PDF belgelerini işlemeye çalışmamalıdır.

### Web Arama ve İçerik Çıkarma (OPSİYONEL, kullanıcı isterse)
- **Skill'ler:** `.agents\skills\tavily-*\SKILL.md` dosyaları
- **Ne zaman:** Kullanıcı belgeyle ilgili harici araştırma yapılmasını isterse Discuss agent veya Analyzer agent bu skill'leri kullanabilir.
- **Kullanılabilir skill'ler:**
  - `tavily-search` — Web'de arama yap
  - `tavily-research` — Derinlemesine araştırma (alıntılı)
  - `tavily-extract` — URL'den temiz içerik çıkar
  - `tavily-crawl` — Siteyi tara
  - `tavily-dynamic-search` — Filtreli arama
- **Kural:** Harici araştırma SADECE kullanıcı açıkça talep ettiğinde yapılır. Varsayılan olarak yalnızca ham kaynak işlenir.

### Skill Kullanma Kuralı
Her agent, kendi göreviyle ilgili harici bir skill'in faydalı olabileceğini düşünürse, önce `skill` tool'u ile o skill'i yüklemeli ve talimatlarını takip etmelidir. Özellikle:
- **Reader agent:** PDF, DOCX, veya özel formatlarda ilgili processing skill'ini kullan
- **Scope agent:** Belge yapısını analiz ederken metindeki başlık/bölüm işaretlerini dikkatle çıkar
- **Analyzer agent:** Kapsam filtresine sadık kal, seçilen odak范围内 varlıkları tanımla. Tablo çıkarımı gerektiğinde `pdf` skill'indeki `pdfplumber` yaklaşımını kullan
- **Writer agent:** Seçilen kapsam dışındaki varlıklar için not yazma (focus modunda context_only varlıkları bağlam notu olarak ekle)
- **Discuss agent:** Kullanıcı harici doğrulama veya ek araştırma isterse `tavily-*` skill'lerini kullan

## Superpowers Skills

Bu proje, `superpowers` paketindeki process skill'lerinden faydalanır. Aşağıdaki skill'ler pipeline geliştirme ve yürütme sırasında kullanılmalıdır:

| Skill | Ne Zaman Kullanılır |
|-------|---------------------|
| **executing-plans** | Bir uygulama planını task-by-task execute ederken. Pipeline task'larını sırayla uygularken. |
| **subagent-driven-development** | Her agent görevini bağımsız bir subagent olarak çalıştırmak istediğinde. Hızlı paralel execution için. |
| **dispatching-parallel-agents** | Bağımsız task'ları (örn: birden fazla not yazma) paralel subagent'lara dağıtırken. |
| **verification-before-completion** | Her agent işini tamamladığını iddia ettiğinde — çıktıyı doğrulama, dosyaların varlığını kontrol etme, formatı doğrulama. |
| **brainstorming** | Yeni agent, özellik veya pipeline aşaması tasarlarken. Kullanıcı "yeni bir şey ekleyelim" dediğinde. |
| **writing-plans** | Yeni bir özellik veya agent için uygulama planı yazarken. |
| **writing-skills** | Yeni bir agent SKILL.md dosyası oluştururken veya mevcut bir skill'i düzenlerken. |
| **systematic-debugging** | Pipeline'da bir hata, beklenmeyen davranış veya test başarısızlığı olduğunda. |
| **finishing-a-development-branch** | Pipeline geliştirme çalışması tamamlandığında — merge, PR veya cleanup kararı verirken. |

### Superpowers Kullanım İlkeleri

1. **Plan varsa executing-plans kullan:** Bir uygulama planı yazıldıysa, onu körü körüne uygulamak yerine `executing-plans` skill'ini yükle ve talimatlarını takip et.
2. **Bağımsız işleri paralelle:** Writer agent birden fazla not yazarken, her not için ayrı subagent dispatch etmek `subagent-driven-development` veya `dispatching-parallel-agents` ile yapılır.
3. **İddia etmeden önce doğrula:** "Tüm notlar yazıldı" deme. `verification-before-completion` ile dosyaları kontrol et, formatı doğrula, bağlantıları test et.
4. **Tasarlamadan önce brainstorm et:** Yeni bir agent veya pipeline aşaması eklemeden önce `brainstorming` skill'ini kullan.
