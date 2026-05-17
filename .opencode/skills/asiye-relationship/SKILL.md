---
name: asiye-relationship
description: "Sınıflandırılmış varlıklar arasındaki ilişkileri haritalandırır. Kişiler, kurumlar, yerler ve olayların nasıl bağlandığını gösteren bir ilişki grafiği oluşturur. Scope kapsam durumunu dikkate alır. Classifier agent varlıkları sınıflandırdıktan sonra kullanın."
---

# Relationship Agent

Classifier agent'tan gelen sınıflandırılmış varlıklar arasındaki ilişkileri haritalandırır. Her varlığın diğer her varlıkla nasıl bağlandığını gösteren kapsamlı bir ilişki grafiği oluşturur.

## Checklist

Sırasıyla aşağıdaki adımları tamamlayın ve her adımda kullanıcı onayı bekleyin:

1. **Sınıflandırılmış varlıkları ve kapsam bilgisini yükle** — Classifier agent'tan gelen sınıflandırma raporunu ve scope_info'yu al
2. **Kaynak metne başvur** — bağlam için orijinal çıkarılmış metne referans et
3. **KİŞİ-KİŞİ ilişkilerini tanımla:**
    - Aile ilişkileri (ebeveyn, çocuk, kardeş, eş)
    - Profesyonel ilişkiler (üst, ast, meslektaş)
    - Siyasi ilişkiler (müttefik, rakip, muhalif)
    - Kişisel ilişkiler (arkadaş, mentor, öğrenci)
4. **KİŞİ-KURUM ilişkilerini tanımla:**
    - Üyelik
    - Liderlik pozisyonları
    - Kurucu roller
    - İstihdam
5. **KİŞİ-YER ilişkilerini tanımla:**
    - Doğum yeri
    - Ölüm yeri
    - İkamet
    - Atandığı/görevlendirildiği yer
    - Ziyaret ettiği yer
6. **KİŞİ-OLAY ilişkilerini tanımla:**
    - Katılım
    - Liderlik
    - Nedenlik (neden oldu, engelledi)
    - Tanıklık
7. **KURUM-YER ilişkilerini tanımla:**
    - Merkez konumu
    - Faaliyet alanı
    - Yetki alanı
8. **KURUM-OLAY ilişkilerini tanımla:**
    - Düzenledi, ev sahipliği yaptı, katıldı
    - Tarafından kuruldu, tarafından feshedildi
9. **OLAY-YER ilişkilerini tanımla:**
    - Olayın gerçekleştiği yer
    - Birden fazla yer varsa hepsi
10. **OLAY-OLAY ilişkilerini tanımla:**
    - Neden oldu, önce geldi, sonra geldi
    - Parçası olarak (daha büyük olay)
    - Tematik olarak ilişkili
11. **YER-YER ilişkilerini tanımla:**
    - İdari hiyerarşi (şehir, il içinde, ülke içinde)
    - Coğrafi yakınlık
    - Tarihi bağlantı
12. **Kapsam durumunu işaretle** — her ilişkinin kapsam durumunu belirt:
    - Her iki varlık da `in_scope` ise → ilişki `in_scope`
    - En az bir varlık `context_only` ise → ilişki `context`
    - Bu bilgi Writer'ın hangi ilişkileri detaylı yazacağını belirler
13. **İlişki grafiği oluştur** — tüm ilişkilerin yapılandırılmış listesini oluştur:
    - Kaynak varlık → Hedef varlık → İlişki türü → Bağlam/Kanıt → Kapsam durumu
14. **Kullanıcıya sun** — ilişki özetini göster ve onay iste
15. **Kullanıcı onayını bekle** — onay almadan ilerleme

## Çıktı Formatı

```yaml
relationships:
  - source: "[[varlik-adi]]"
    target: "[[varlik-adi]]"
    type: "iliski turu"
    description: "İlişkinin açıklaması"
    evidence: "Kaynak metinden alıntı veya referans"
    confidence: "high/medium/low"
    scope: "in_scope | context"

relationship_matrix:
  | Varlık A | Varlık B | Tür | Açıklama | Kapsam |
  |----------|----------|-----|----------|--------|
  | ...      | ...      | ... | ...      | ...    |
```

## Önemli Notlar

- Her ilişki kaynak metindeki kanıtlarla desteklenmelidir.
- Düşük güvenilirlikli ilişkileri kullanıcı incelemesi için işaretle.
- İlişkilere zamansal bağlam ekle (bu ilişki ne zaman vardı?).
- Kapsam durumunu her ilişki için belirt — Writer bu bilgiyi kullanacak.
- Tüm açıklamalar Türkçedir.
- Kullanıcıya sunarken: "Toplam N ilişki haritalandırıldı: X in_scope, Y context. Devam edelim mi?" diye sor.
