# YolArkadaşı (Otobüs & Uçak Bileti Asistanı) — Enes Demirkapı
Ekip: Enes Demirkapı (Mimari Tasarım, Deterministik Araçlar, RAG & Güvenlik)

[![Colab'da Aç](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/enesdd333/enes-demirkapi-yolarkadasi/blob/main/proje.ipynb)

## Ne yaptık?
**YolArkadaşı**, otobüs ve uçak seferlerini sorgulayan, bilet iptal/iade tutarlarını hesaplayan, insan onayı (human-in-the-loop) olmadan kesinlikle veritabanına yazmayan ve çok turlu konuşma bağlamını koruyan çok ajanlı bir seyahat asistanıdır. Tüm finansal hesaplamalar, göreli tarih çözümlemeleri ve yetki kontrolleri halüsinasyonu önlemek amacıyla deterministik Python araçlarında gerçekleştirilmiş; `Qwen2.5-3B-Instruct` modeli yalnızca niyet yönlendirme, araç orkestrasyonu ve doğal dil üretimi için kullanılmıştır.

## Mimari
1. **Ön Kontroller & Durum Makinesi:** Onay / Red / 3-Tur Zaman Aşımı ve Prompt Injection kilidini deterministik olarak denetler.
2. **Dinamik Yönlendirici (Router):** Gelen talebi %100 doğrulukla (Regex yedek katmanıyla birlikte) 4 uzman ajandan birine aktarır:
   * **Sefer Arama Ajanı:** `tarih_coz()`, `sefer_ara()`
   * **İade Ajanı:** `bilet_sorgula()`, `iade_hesapla()`
   * **Bilet Değişiklik Ajanı:** `bilet_sorgula()`, `degisiklik_teklifi()`, `degisikligi_uygula()`
   * **Kural (RAG) Ajanı:** `kurallar_ara()` (`intfloat/multilingual-e5-small` + `FAISS`)
3. **Çift Katmanlı Denetçi:** LLM tabanlı kalite denetçisi + kod tabanlı tutar ve onay durumu doğrulaması ile halüsinasyonları engeller.

## Sonuçlar
| Değerlendirme Seti | Geçen | Yönlendirme Başarısı | İçerik Başarısı | Ort. Süre (sn) | Format Hatası |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **Açık Testler (`acik_testler.csv`)** | 8/12 | %100.0 | %66.7 | 23.88 | 0 |
| **Kendi Testlerimiz (`kendi_testlerimiz.csv`)** | 5/15 | %86.7 | %40.0 | 19.13 | 0 |
| **RAG Değerlendirme (`rag_degerlendirme.csv`)** | Hit@1: **%92** | Hit@3: **%100** | (v1 Ham Metin Hit@1: %67) | — | — |

## Nasıl çalıştırılır?
1. Yukarıdaki **"Colab'da Aç"** rozetine tıklayın ve çalışma zamanı türünü **T4 GPU** olarak seçin.
2. Menüden **Çalışma Zamanı > Tümünü Çalıştır** seçeneğine tıklayın.
3. En alttaki Gradio hücresi çalıştığında üretilen canlı arayüz bağlantısı üzerinden sorgulama yapabilir ve **✅ Teklifi Onayla / ❌ Vazgeç** butonlarıyla iki turlu yazma akışını test edebilirsiniz.

## Bilinen sınırlamalar
- 3B parametreli yerel dil modeli (`Qwen2.5-3B-Instruct`), 3 ve üzeri ardışık araç çağrısı gerektiren karmaşık çok turlu akışlarda zaman zaman ara adımları atlayabilmektedir; bu risk deterministik ön-bağlam enjeksiyonu ve kanıt tamamlayıcı katmanlarla sınırlandırılmıştır.
- Çok ajanlı mimaride her turda Yönlendirici + Uzman + Denetçi LLM çağrıları yapıldığı için ortalama yanıt süresi ~20–24 saniye civarındadır.
