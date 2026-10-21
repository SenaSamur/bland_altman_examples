# Bland-Altman Agreement Analysis: A Practical Guide
# Bland-Altman Uyum Analizi: Uygulamalı Rehber

Companion to `bland_altman_study.ipynb` · `bland_altman_study.ipynb` için yardımcı rehber

> ⚠️ **EN:** The two studies used here (AI bladder volume, AI liver length) are **synthetic** and for training only.
> **TR:** Burada kullanılan iki çalışma (AI mesane hacmi, AI karaciğer uzunluğu) **sentetiktir** ve yalnızca eğitim amaçlıdır.

---

## Contents / İçindekiler

1. [What question does Bland-Altman answer? / Bland-Altman hangi soruyu cevaplar?](#1-what-question-does-bland-altman-answer--bland-altman-hangi-soruyu-cevaplar)
2. [Study design / Çalışma tasarımı](#2-study-design--çalışma-tasarımı)
3. [The primary endpoint, step by step / Birincil sonlanım, adım adım](#3-the-primary-endpoint-step-by-step--birincil-sonlanım-adım-adım)
4. [Why correlation is not enough / Neden korelasyon yetmez](#4-why-correlation-is-not-enough--neden-korelasyon-yetmez)
5. [Choosing the scale: absolute or percent / Ölçek seçimi: mutlak mı yüzde mi](#5-choosing-the-scale-absolute-or-percent--ölçek-seçimi-mutlak-mı-yüzde-mi)
6. [Acceptance limits and their sources / Kabul sınırları ve kaynakları](#6-acceptance-limits-and-their-sources--kabul-sınırları-ve-kaynakları)
7. [Supportive analyses / Destekleyici analizler](#7-supportive-analyses--destekleyici-analizler)
8. [Sample size / Örneklem büyüklüğü](#8-sample-size--örneklem-büyüklüğü)
9. [Results of the synthetic studies / Sentetik çalışmaların sonuçları](#9-results-of-the-synthetic-studies--sentetik-çalışmaların-sonuçları)
10. [Files, data dictionary and how to run / Dosyalar, veri sözlüğü ve çalıştırma](#10-files-data-dictionary-and-how-to-run--dosyalar-veri-sözlüğü-ve-çalıştırma)
11. [Common mistakes / Sık yapılan hatalar](#11-common-mistakes--sık-yapılan-hatalar)
12. [References / Kaynaklar](#12-references--kaynaklar)

---

## 1. What question does Bland-Altman answer? / Bland-Altman hangi soruyu cevaplar?

**EN:** Bland-Altman analysis compares two methods that measure the same continuous quantity (mL, mm, cm, %). It answers: *for an individual patient, how far apart can the two methods be?* It is the standard primary analysis in FDA submissions for AI measurement tools (product code QIH), where a new method (the AI) is compared with an established one (expert readers).

It does **not** decide on its own whether the agreement is good enough. It gives an interval; the decision comes from comparing that interval with a clinically justified **acceptance limit** written into the protocol in advance.

It also measures **agreement with the experts, not accuracy**. The reference is the experts' average, not the true value. If the experts are systematically wrong, an AI that copies them will still look "in agreement".

**TR:** Bland-Altman analizi aynı sürekli büyüklüğü (mL, mm, cm, %) ölçen iki yöntemi karşılaştırır. Şu soruyu cevaplar: *tek bir hastada iki yöntem birbirinden ne kadar farklı olabilir?* Yeni bir yöntemin (AI) yerleşik bir yöntemle (uzman okuyucular) karşılaştırıldığı AI ölçüm araçlarının (ürün kodu QIH) FDA başvurularında standart birincil analizdir.

Uyumun yeterli olup olmadığına tek başına **karar vermez**. Bir aralık verir; karar, bu aralığın protokolde önceden yazılmış, klinik olarak gerekçelendirilmiş bir **kabul sınırıyla** karşılaştırılmasıyla verilir.

Ayrıca **doğruluğu değil, uzmanlarla uyumu** ölçer. Referans gerçek değer değil, uzmanların ortalamasıdır. Uzmanlar sistematik olarak yanılıyorsa, onları taklit eden bir AI yine "uyumlu" görünür.

---

## 2. Study design / Çalışma tasarımı

**EN:** Both synthetic studies follow the pattern seen in FDA 510(k) summaries such as K230497 (Bladder AI) and K253595/K260667 (Philips Auto Measure Abdomen):

| Element | Bladder study | Liver study |
|---|---|---|
| Design | Retrospective, multi-site, standalone | Retrospective, multi-site, standalone |
| Sites / devices | 3 sites; cart-based and handheld | 3 sites; cart-based and handheld |
| Enrolled / analyzed | 150 / 137 | 160 / 147 |
| Measurement | Bladder volume (mL) | Liver craniocaudal length, midclavicular line (cm) |
| Reference standard | Mean of 3 independent, blinded experts | Mean of 3 independent, blinded experts |
| Primary endpoint | Bland-Altman 95 % LoA, absolute (mL) | Bland-Altman 95 % LoA, percent (%) |
| Acceptance limit | ±55 mL | ±15 % |
| Clinical cut-off | PVR > 200 mL | Hepatomegaly > 16 cm |

Averaging 3 readers reduces noise in the reference. With a single reader, that reader's own error would widen the LoA and penalise the AI unfairly.

**TR:** İki sentetik çalışma da K230497 (Bladder AI) ve K253595/K260667 (Philips Auto Measure Abdomen) gibi FDA 510(k) özetlerindeki kalıbı izler: retrospektif, çok merkezli, standalone tasarım; 3 merkez, cart-based ve handheld cihazlar; referans olarak 3 bağımsız ve kör uzmanın ortalaması. Mesane çalışmasında 150 vakanın 137'si, karaciğer çalışmasında 160 vakanın 147'si analiz edildi. Birincil sonlanım mesanede mutlak fark (±55 mL), karaciğerde yüzde fark (±%15).

3 okuyucunun ortalamasını almak referanstaki gürültüyü azaltır. Tek okuyucu kullanılsaydı, o okuyucunun kendi hatası LoA'yı genişletir ve AI'ı haksız yere cezalandırırdı.

---

## 3. The primary endpoint, step by step / Birincil sonlanım, adım adım

*Notebook section 2 · Notebook 2. bölüm*

### Step 1 · The difference / Adım 1 · Fark

**EN:** For each patient *i*:

- absolute scale: $d_i = \text{AI}_i - \text{Ref}_i$
- percent scale: $d_i = \dfrac{\text{AI}_i - \text{Ref}_i}{(\text{AI}_i + \text{Ref}_i)/2} \times 100$

A positive difference means the AI measures larger than the experts.

**TR:** Her hasta *i* için fark yukarıdaki gibi hesaplanır. Pozitif fark AI'ın uzmanlardan büyük ölçtüğünü gösterir.

### Step 2 · Bias / Adım 2 · Bias

**EN:** $\bar d = \frac{1}{n}\sum d_i$, the average systematic shift. A bias of 0 does not mean good agreement: +50 and −50 also average to 0.

**TR:** $\bar d = \frac{1}{n}\sum d_i$, ortalama sistematik sapma. Bias'ın 0 olması iyi uyum demek değildir: +50 ile −50'nin ortalaması da 0'dır.

### Step 3 · SD of the differences / Adım 3 · Farkların SD'si

**EN:** $s_d$ measures how scattered the individual differences are around the bias: the random part of the disagreement.

**TR:** $s_d$, bireysel farkların bias etrafında ne kadar saçıldığını ölçer: uyumsuzluğun rastgele kısmı.

### Step 4 · Limits of agreement / Adım 4 · Uyum sınırları

$$\text{LoA} = \bar d \pm 1.96\, s_d$$

**EN:** If the differences are roughly normal, about 95 % of patients fall between the two limits. This is the clinically useful statement: *"for 95 % of patients the AI differs from the experts by between X and Y."*

**TR:** Farklar yaklaşık normal dağılıyorsa hastaların yaklaşık %95'i iki sınır arasında kalır. Klinik olarak kullanışlı ifade budur: *"Hastaların %95'inde AI uzmanlardan X ile Y arasında farklı ölçer."*

**Worked example / Çözümlü örnek (bladder / mesane, n = 137):**

| Quantity / Büyüklük | Calculation / Hesap | Result / Sonuç |
|---|---|---|
| Bias | Σd / n | +1.77 mL |
| SD | s_d | 21.59 mL |
| Lower LoA / Alt LoA | 1.77 − 1.96 × 21.59 | −40.54 mL |
| Upper LoA / Üst LoA | 1.77 + 1.96 × 21.59 | +44.09 mL |

### Step 5 · Confidence intervals / Adım 5 · Güven aralıkları

**EN:** The bias and LoA are sample estimates, so they have confidence intervals (Bland & Altman 1999):

- $SE(\bar d) = s_d/\sqrt{n}$
- $SE(\text{LoA}) \approx s_d\sqrt{\tfrac{1}{n} + \tfrac{1.96^2}{2(n-1)}}$
- each CI = estimate ± *t*(n−1) × SE

If the bias CI excludes 0, the systematic shift is statistically real (liver study: bias −2.13 %, CI −3.04 to −1.22). The LoA CIs are used by the strict acceptance criterion.

**TR:** Bias ve LoA örneklem tahminleridir, bu yüzden güven aralıkları vardır (formüller yukarıda). Bias'ın GA'sı 0'ı içermiyorsa sistematik sapma istatistiksel olarak gerçektir (karaciğer çalışması: bias −%2.13, GA −3.04 ile −1.22). LoA'nın GA'ları sıkı kabul kriterinde kullanılır.

### Step 6 · Decision / Adım 6 · Karar

| Criterion / Kriter | Rule / Kural |
|---|---|
| Point / Nokta | −limit ≤ lower LoA and upper LoA ≤ +limit |
| Strict / Sıkı | −limit ≤ lower end of lower-LoA CI and upper end of upper-LoA CI ≤ +limit |

**EN:** The strict criterion protects against passing by luck with a small sample; Philips used it in K260667.
**TR:** Sıkı kriter, küçük örneklemde şans eseri geçmeye karşı korur; Philips K260667'de bunu kullandı.

### Step 7 · Assumption checks / Adım 7 · Varsayım kontrolleri

| Check / Kontrol | How / Nasıl | If it fails / Sağlanmazsa |
|---|---|---|
| Proportional bias: does the *average* difference change with size? / Orantısal hata: *ortalama* fark boyutla değişiyor mu? | Regress d on the mean; test the slope / d'yi ortalamaya göre regrese et, eğimi test et | Use percent scale or log transform / Yüzde ölçek veya log dönüşümü |
| Heteroscedasticity: does the *spread* change with size? / Heteroskedastisite: *yayılım* boyutla değişiyor mu? | Spearman ρ between \|d\| and the mean (section 5) / \|d\| ile ortalama arasında Spearman ρ (5. bölüm) | Change scale or report LoA as a function of size / Ölçeği değiştir veya LoA'yı boyuta göre raporla |
| Normality / Normallik | Histogram, Q-Q plot, Shapiro-Wilk | Also report nonparametric LoA (2.5th–97.5th percentile) / Nonparametrik LoA'yı da raporla |
| Outliers / Aykırı değerler | ~5 % outside the LoA is expected / ~%5 beklenir | Review each case; never remove after unblinding / Her vakayı incele; körlük kalktıktan sonra asla çıkarma |

**EN:** Shapiro-Wilk becomes significant easily with large n. In the bladder study it is significant (p < 0.001) because of a few large AI errors, but the nonparametric LoA (−34.8 / +44.6 mL) is close to the parametric one, so the conclusion does not change.

**TR:** n büyükken Shapiro-Wilk kolayca anlamlı çıkar. Mesane çalışmasında birkaç büyük AI hatası nedeniyle anlamlı (p < 0.001), ama nonparametrik LoA (−34.8 / +44.6 mL) parametrik LoA'ya yakın; dolayısıyla sonuç değişmiyor.

### Reading the Bland-Altman plot / Bland-Altman grafiğini okumak

| Element / Öğe | Meaning / Anlamı |
|---|---|
| x-axis / x ekseni | Mean of AI and reference: best estimate of the true size / AI ve referansın ortalaması: gerçek boyutun en iyi tahmini |
| y-axis / y ekseni | Difference / Fark |
| Blue line + band / Mavi çizgi + bant | Bias and its 95 % CI / Bias ve %95 GA |
| Orange dashed lines + bands / Turuncu kesikli çizgiler + bantlar | LoA and their 95 % CIs / LoA ve %95 GA'ları |
| Grey band / Gri bant | Acceptance limit / Kabul sınırı |
| Dash-dot line / Kesik-noktalı çizgi | Regression trend (proportional bias) / Regresyon trendi (orantısal hata) |
| Red circles / Kırmızı halkalar | Cases outside the LoA / LoA dışındaki vakalar |

**EN — four questions:** Is the blue line far from 0? Is the band narrow enough clinically? Do the dots slope or fan out? Do outliers cluster in one region?
**TR — dört soru:** Mavi çizgi 0'dan uzak mı? Bant klinik olarak yeterince dar mı? Noktalar eğim gösteriyor ya da yelpaze gibi açılıyor mu? Aykırı değerler bir bölgede toplanıyor mu?

---

## 4. Why correlation is not enough / Neden korelasyon yetmez

*Notebook section 3 · Notebook 3. bölüm*

**EN:**

1. **Correlation measures association, not agreement.** r asks whether the points lie on *any* straight line; agreement requires the line y = x.
2. **r is blind to systematic error.** Pearson r does not change when a constant is added or when the values are multiplied by a constant. In the notebook, the original AI, "AI + 30 mL" and "AI × 1.20" all have r = 0.9906, while the bias and LoA of the faulty versions fall far outside the limit.
3. **r depends on the range.** Splitting the cases into thirds by size gives a much lower r in each third, even though the disagreement per patient is similar. A study with very different patient sizes can report an impressive r without the agreement being any better.

ICC(A,1), which measures absolute agreement, does fall for the faulty versions, so it is a better supportive measure than r. It still gives no answer in clinical units.

**TR:**

1. **Korelasyon ilişkiyi ölçer, uyumu değil.** r noktaların *herhangi bir* doğru üzerinde olup olmadığını sorar; uyum ise y = x doğrusunu gerektirir.
2. **r sistematik hatayı görmez.** Pearson r, bir sabit eklendiğinde ya da değerler bir sabitle çarpıldığında değişmez. Notebook'ta orijinal AI, "AI + 30 mL" ve "AI × 1.20" için r = 0.9906; hatalı versiyonların bias ve LoA'sı ise sınırın çok dışına çıkıyor.
3. **r aralığa bağlıdır.** Vakaları boyuta göre üçe bölmek, her dilimde çok daha düşük bir r verir; oysa hasta başına uyumsuzluk benzerdir. Çok farklı boyutlarda hastalar alan bir çalışma, uyum hiç iyileşmeden etkileyici bir r raporlayabilir.

Mutlak uyumu ölçen ICC(A,1) hatalı versiyonlarda düşer; bu yüzden destekleyici ölçü olarak r'den iyidir. Yine de klinik birimde bir cevap vermez.

---

## 5. Choosing the scale: absolute or percent / Ölçek seçimi: mutlak mı yüzde mi

*Notebook section 4 · Notebook 4. bölüm*

| Scale / Ölçek | Question / Soru | Natural when / Ne zaman doğal |
|---|---|---|
| Absolute (mL, cm) / Mutlak | "How many units can the AI be off?" / "AI kaç birim yanılabilir?" | Error is constant across sizes; decision uses a fixed threshold / Hata boyuttan bağımsız; karar sabit bir eşiğe dayanıyor |
| Percent (%) / Yüzde | "By what fraction can the AI be off?" / "AI hangi oranda yanılabilir?" | Error grows with size; wide range of sizes / Hata boyutla büyüyor; boyut aralığı geniş |

**EN:** The check is the Spearman correlation between |difference| and the mean. The scale with ρ closer to 0 has the more stable spread.

| Study | ρ absolute | ρ percent | Primary scale | Why |
|---|---|---|---|---|
| Bladder | +0.50 | −0.24 | Absolute (mL) | The clinical threshold (PVR 200 mL) and FDA K230497 use mL, even though the spread grows with volume. The secondary percent analysis (reference ≥ 100 mL) covers the gap. |
| Liver | +0.19 | +0.02 | Percent (%) | Percent removes the size dependence; cm is reported as secondary for clinicians. |

**General rule:** choose the scale with stable spread, unless a clinical threshold or regulatory precedent requires a specific unit. In that case, report the other scale as a secondary endpoint and justify the trade-off in the protocol.

**TR:** Kontrol, |fark| ile ortalama arasındaki Spearman korelasyonudur. ρ'su 0'a daha yakın olan ölçeğin yayılımı daha sabittir.

- **Mesane:** mutlak ölçekte ρ = +0.50 (yayılım hacimle belirgin şekilde büyüyor), yüzde ölçekte ρ = −0.24 (küçük hacimlerde yüzde şişiyor). Birincil ölçek yine mL, çünkü klinik eşik (PVR 200 mL) ve FDA K230497 mL kullanıyor. Bedeli: tek bir ±mL aralığı küçük mesaneler için fazla cömert, büyük mesaneler için fazla katı. Referans ≥ 100 mL ile sınırlı ikincil yüzde analizi bu boşluğu kapatıyor.
- **Karaciğer:** mutlak ölçekte ρ = +0.19, yüzde ölçekte ρ = +0.02. Yüzde ölçek boyut bağımlılığını ortadan kaldırdığı için birincil; cm ikincil olarak raporlanıyor.

**Genel kural:** yayılımın sabit olduğu ölçeği seç. Klinik eşik ya da düzenleyici emsal belirli bir birim gerektiriyorsa onu kullan, diğer ölçeği ikincil sonlanım olarak raporla ve ödünleşimi protokolde gerekçelendir.

---

## 6. Acceptance limits and their sources / Kabul sınırları ve kaynakları

**EN:** The usual logic is that *the AI should not disagree with the experts more than the experts disagree with each other*. The limit is derived from published inter-reader variability or from accepted precedents.

**TR:** Genel mantık şudur: *AI uzmanlardan, uzmanların birbirinden farklılaştığından daha fazla farklılaşmamalı*. Sınır, yayımlanmış okuyucular arası değişkenlikten veya kabul görmüş emsallerden türetilir.

### Bladder volume: ±55 mL / Mesane hacmi: ±55 mL

- **EN:** Inter-sonographer agreement for manual 2D US: 3.2 ± 54.4 mL (AI bladder volume study, 2024, PMC11353831). In that study the AI itself reached bias 17.1 mL, LoA −52.7 / +86.9 mL against a single expert.
- **EN:** FDA K230497 Bladder AI: 122 subjects, volumes 11–645 mL, reference = mean of 3 experts. Dual-view bias 2 mL, LoA −42 / +46 mL; ICC 0.98. No pre-specified acceptance limit is stated in the summary.
- **TR:** Manuel 2D US'de sonograflar arası uyum 3.2 ± 54.4 mL (PMC11353831, 2024). FDA K230497: 122 hasta, hacim 11–645 mL, çift görüntüde bias 2 mL, LoA −42 / +46 mL. Özette önceden belirlenmiş bir kabul sınırı yazmıyor.

### Liver craniocaudal length: ±15 % / Karaciğer kraniokaudal uzunluğu: ±%15

- **EN:** No published numeric LoA for liver length was found. The limit follows FDA organ-length precedents in K260667 (Philips Auto Measure Abdomen): kidney sagittal length ±14.3 %, spleen length ±15.9 %. Liver measurements are less reproducible than kidney (inter-rater ICC 0.71–0.94, Childs et al. 2014), so a tighter limit would not be realistic.
- **TR:** Karaciğer uzunluğu için yayımlanmış sayısal LoA bulunamadı. Sınır, K260667'deki FDA organ uzunluğu emsallerine dayanıyor: böbrek ±%14.3, dalak ±%15.9. Karaciğer ölçümleri böbrekten daha az tekrarlanabilir (okuyucular arası ICC 0.71–0.94, Childs ve ark. 2014); bu yüzden daha dar bir sınır gerçekçi olmaz.

### Clinical cut-offs / Klinik eşikler

- PVR > 200 mL: elevated post-void residual / artmış post-void rezidü (PMC11353831).
- Hepatomegaly > 16 cm (midclavicular line); the upper limit of normal is usually given as 15–16 cm and varies by institution / Hepatomegali > 16 cm; normalin üst sınırı genellikle 15–16 cm, kurumdan kuruma değişir.

> **EN:** In a real study, write the limit into the protocol before unblinding and confirm it with FDA in a Pre-Submission meeting.
> **TR:** Gerçek bir çalışmada sınırı körlük kaldırılmadan önce protokole yaz ve FDA ile Pre-Submission görüşmesinde teyit et.

---

## 7. Supportive analyses / Destekleyici analizler

*Notebook sections 5–7 · Notebook 5–7. bölümler*

| Analysis / Analiz | EN | TR |
|---|---|---|
| ICC(A,1) | Absolute-agreement index, 0–1 | 0–1 arası mutlak uyum indeksi |
| MAE | Mean absolute error in clinical units | Klinik birimde ortalama mutlak hata |
| Clinical cut-off | Sensitivity, specificity (Wilson CI), kappa: does the AI put the patient on the same side of the decision threshold? | Duyarlılık, özgüllük (Wilson GA), kappa: AI hastayı karar eşiğinin aynı tarafına koyuyor mu? |
| AI vs. readers | Compares the AI's LoA with reader-vs-reader LoA; also tests whether the limit is realistic | AI'ın LoA'sını okuyucular arası LoA ile kıyaslar; sınırın gerçekçiliğini de sınar |
| Subgroups | Site, device, sex, BMI, age, image quality; exploratory, they do not change the decision | Merkez, cihaz, cinsiyet, BMI, yaş, görüntü kalitesi; keşifsel, kararı değiştirmez |

---

## 8. Sample size / Örneklem büyüklüğü

*Notebook section 8 · Notebook 8. bölüm*

**EN:** The notebook simulates, for each n, the probability that both LoA confidence intervals lie inside the limit (strict criterion), given an assumed bias and SD. This mirrors the analytic method of Lu et al. (2016). The SD should come from pilot data or the literature, and 10–20 % should be added for non-analyzable cases.

| Study | Assumed bias | Assumed SD | Limit | n for 80 % power | n for 90 % power |
|---|---|---|---|---|---|
| Bladder | +2 mL | 22 mL | ±55 mL | 140 | 180 |
| Liver | −1.5 % | 5 % | ±15 % | 60 | 80 |

The bladder study (150 enrolled, ~9 % excluded, 137 analyzed) matches the 80 % target.

**TR:** Notebook, varsayılan bias ve SD ile her n için her iki LoA güven aralığının sınır içinde kalma olasılığını (sıkı kriter) simüle eder. Bu, Lu ve ark. (2016) analitik yönteminin karşılığıdır. SD pilot veriden veya literatürden alınmalı, analiz edilemeyecek vakalar için %10–20 eklenmelidir. Mesane çalışması (150 kayıtlı, ~%9 dışlanma, 137 analiz) %80 güç hedefiyle uyumlu.

---

## 9. Results of the synthetic studies / Sentetik çalışmaların sonuçları

| | Bladder / Mesane | Liver / Karaciğer |
|---|---|---|
| Analyzed / Analiz edilen | 137 / 150 | 147 / 160 |
| Bias (95 % CI) | +1.77 mL (−1.87 / +5.42) | −2.13 % (−3.04 / −1.22) |
| LoA | −40.54 / +44.09 mL | −13.10 / +8.84 % |
| Limit / Sınır | ±55 mL | ±15 % |
| Point criterion / Nokta | PASS | PASS |
| Strict criterion / Sıkı | PASS | PASS |
| Reader-vs-reader LoA / Okuyucular arası LoA | about / yaklaşık −52 / +50 to −76 / +55 mL | about / yaklaşık −18 / +14 to −21 / +12 % |
| ICC(A,1) | 0.991 | 0.926 |
| Clinical cut-off kappa / Klinik eşik kappa | 0.851 | 0.843 |
| Subgroups exceeding the limit / Sınırı aşan alt gruplar | Age 65+ / 65 yaş üstü | Site_A, BMI ≥ 30 |

**EN — interpretation:** Both AIs pass on both criteria and agree with the reference more closely than the readers agree with each other. The liver AI has a small but statistically real negative bias (it measures slightly short). The flagged subgroups are exploratory, but in a real submission they would need discussion.

**TR — yorum:** İki AI da iki kriteri geçiyor ve referansla okuyucuların birbiriyle olduğundan daha yakın uyum gösteriyor. Karaciğer AI'ında küçük ama istatistiksel olarak gerçek bir negatif bias var (biraz kısa ölçüyor). İşaretlenen alt gruplar keşifsel, ama gerçek bir başvuruda tartışılmaları gerekir.

---

## 10. Files, data dictionary and how to run / Dosyalar, veri sözlüğü ve çalıştırma

### Files / Dosyalar

| File / Dosya | Content / İçerik |
|---|---|
| `bland_altman_study.ipynb` | Full analysis with explanations / Açıklamalı tam analiz |
| `bladder_study.csv` | 150 bladder cases / 150 mesane vakası |
| `liver_study.csv` | 160 liver cases / 160 karaciğer vakası |
| `bland_altman_guide.md` | This guide / Bu rehber |

### Data dictionary / Veri sözlüğü

| Column / Sütun | Description / Açıklama |
|---|---|
| `case_id` | Case number / Vaka numarası |
| `site` | Site_A / B / C |
| `device_type` | Cart-based / Handheld |
| `age`, `sex`, `bmi`, `bmi_category` | Demographics / Demografi |
| `image_quality` | Good / Adequate / Poor |
| `analyzable` | Yes / No |
| `exclusion_reason` | Reason for exclusion; measurements are empty for excluded cases / Dışlanma nedeni; dışlananlarda ölçümler boş |
| `reader1_*`, `reader2_*`, `reader3_*` | Expert measurements (`_ml` or `_cm`) / Uzman ölçümleri |
| `ai_ml` / `ai_cm` | AI measurement / AI ölçümü |

The reference (mean of 3 readers) is computed by the notebook. / Referans (3 okuyucunun ortalaması) notebook tarafından hesaplanır.

### How to run / Nasıl çalıştırılır

```bash
# 1) Install packages / Paketleri kur
pip install numpy pandas scipy matplotlib jupyter

# 2) Open interactively, then Kernel -> Restart & Run All
#    Etkileşimli aç, sonra Kernel -> Restart & Run All
jupyter notebook bland_altman_study.ipynb

# 3) Run headless and keep the executed copy
#    Arayüzsüz çalıştır, çalıştırılmış kopyayı kaydet
jupyter nbconvert --to notebook --execute bland_altman_study.ipynb --output bladder_executed.ipynb

# 4) HTML report / HTML rapor
jupyter nbconvert --to html --execute bland_altman_study.ipynb
```

**EN:** In the first code cell set `STUDY = "bladder"` or `"liver"`. `RUN_POWER = False` skips the ~1 min simulation. To use your own data, keep the same column names and set `CSV_PATH`. Limits, scales and cut-offs are in the `CONFIG` dictionary. Figures and tables are saved to `results_bladder/` and `results_liver/`.

**TR:** İlk kod hücresinde `STUDY = "bladder"` veya `"liver"` yap. `RUN_POWER = False` ~1 dakikalık simülasyonu atlar. Kendi verin için aynı sütun adlarını koru ve `CSV_PATH`'i ayarla. Sınırlar, ölçekler ve eşikler `CONFIG` sözlüğündedir. Grafikler ve tablolar `results_bladder/` ve `results_liver/` klasörlerine kaydedilir.

---

## 11. Common mistakes / Sık yapılan hatalar

| Mistake / Hata | Why it is wrong / Neden yanlış |
|---|---|
| Reporting only r or R² / Yalnızca r veya R² raporlamak | Blind to systematic error; inflated by range / Sistematik hatayı görmez; aralıkla şişer |
| Choosing the limit after seeing the data / Sınırı veriyi gördükten sonra seçmek | The result becomes meaningless; FDA expects a pre-specified limit / Sonuç anlamsızlaşır; FDA önceden belirlenmiş sınır bekler |
| Treating the LoA as a hypothesis test / LoA'yı hipotez testi gibi yorumlamak | There is no p-value; the LoA is an interval judged against a clinical limit / p değeri yoktur; LoA klinik sınırla değerlendirilen bir aralıktır |
| Ignoring the LoA confidence intervals / LoA güven aralıklarını yok saymak | A small study can pass by chance / Küçük bir çalışma şans eseri geçebilir |
| Using absolute differences when the spread grows with size (or the reverse) / Yayılım boyutla büyürken mutlak fark kullanmak (veya tersi) | The LoA is wrong for part of the range / LoA aralığın bir kısmında yanlış olur |
| Removing outliers after unblinding / Körlük kalktıktan sonra aykırı değer çıkarmak | Biases the result; review and report them instead / Sonucu yanlı kılar; incele ve raporla |
| Using one reader as the reference / Tek okuyucuyu referans almak | That reader's error widens the LoA / O okuyucunun hatası LoA'yı genişletir |
| Treating repeated images of one patient as independent / Aynı hastanın tekrarlı görüntülerini bağımsız saymak | Underestimates the SD; use Bland & Altman 2007 for repeated measures / SD'yi küçük tahmin eder; tekrarlı ölçümler için Bland & Altman 2007 |
| Claiming the AI can replace experts / AI'ın uzmanların yerini alabileceğini iddia etmek | The analysis shows agreement with experts in a standalone setting, not reader performance with AI; that needs an MRMC design / Analiz standalone ortamda uzmanlarla uyumu gösterir, AI ile okuyucu performansını değil; o MRMC tasarımı gerektirir |

---

## 12. References / Kaynaklar

**Method / Yöntem**
- Bland JM, Altman DG. Statistical methods for assessing agreement between two methods of clinical measurement. *Lancet* 1986;327:307–310.
- Bland JM, Altman DG. Measuring agreement in method comparison studies. *Stat Methods Med Res* 1999;8:135–160.
- Bland JM, Altman DG. Agreement between methods of measurement with multiple observations per individual. *J Biopharm Stat* 2007;17:571–582.
- Lu MJ et al. Sample size for assessing agreement between two methods of measurement by Bland-Altman method. *Int J Biostat* 2016.
- McGraw KO, Wong SP. Forming inferences about some intraclass correlation coefficients. *Psychol Methods* 1996;1:30–46.
- Wen S, Gallas BD. Three-way mixed effect ANOVA to estimate MRMC limits of agreement. arXiv:2107.08891.

**FDA precedents / FDA emsalleri**
- K230497 Bladder AI: https://fda.innolitics.com/device/K230497
- K260667 Philips Auto Measure Abdomen: https://fda.innolitics.com/device/K260667
- K262548 Philips 2D Auto LA: https://www.accessdata.fda.gov/cdrh_docs/pdf26/K262548.pdf

**Clinical literature / Klinik literatür**
- AI-powered automatic bladder volume measurement with 2D ultrasound (2024): https://pmc.ncbi.nlm.nih.gov/articles/PMC11353831/
- Alfuraih et al. Handheld 3D vs 2D ultrasound for bladder volume. *Diagnostics* 2025;15:2229: https://pmc.ncbi.nlm.nih.gov/articles/PMC12428280/
- Childs JT et al. Ultrasound measurements of the liver: an intra and inter-rater reliability study. *Australas J Ultrasound Med* 2014;17:113–119.
- Liver size reference values and hepatomegaly thresholds: https://radshare.net/articles/liver-size-reference-values-and-hepatomegaly-thresholds-on-ultrasound
