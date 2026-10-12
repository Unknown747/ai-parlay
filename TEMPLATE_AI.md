# ai-parlay

TEMPLATE PROJECT — AI PREDICTION

Khusus proyek prediksi berbasis AI (Gemini, Groq, GPT, Claude).
Tidak bercampur dengan statistik/Poisson.

---

KONSEP DASAR

Proyek ini beda dari proyek statistik:

Aspek Statistik AI
Cara prediksi Hitung dari data Tanya AI
Backtest Bisa 5 musim Tidak bisa
Validasi Walk-forward Live test
Look-ahead Tidak ada Jebakan utama
Konsistensi Deterministik Tidak
Biaya Gratis Free tier / $
Ensemble Tidak ada Bisa 2+ AI

Kesimpulan: AI tidak bisa backtest historis. Validasi harus live.

---

FASE 0: PERTANYAAN (SEBELUM CODING)

```
HIPOTESIS:
  Model:  Multiple AI
  Pasar:  ...................
  Liga:   ...................
  Edge:   Ensemble AI > pasar

METRIK SUKSES:
  ROI > +2% dalam 100 laga LIVE
  Konsisten 2+ minggu

API KEY:
  Gratis:  Gemini, Groq, Cerebras
  Bayar:   GPT-4, Claude (kalau perlu)

BATAS:
  Test look-ahead: 1 hari
  Live test: 30 hari
  Berhenti kalau: 100 laga, ROI negatif
```

---

FASE 1: UJI LOOK-AHEAD (WAJIB)

Kenapa wajib

AI mungkin tahu hasil historis dari training. Kalau iya, backtest historis tidak valid.

Prompt test

```
"Prediksi Over/Under 2.5 untuk Valencia vs Barcelona di La Liga.
Jawab HANYA: Over / Under / Tidak tahu."
```

Jangan kasih tanggal. Jangan kasih skor historis.

Cara cek

```
1. Ambil 20 laga historis dari backtest_odds.csv
2. Tanya AI (tanpa tanggal)
3. Catat: Over / Under / Tidak tahu
4. Bandingkan dengan hasil nyata

HASIL:
  AI benar > 70%     → TAHU HASIL → backtest tidak valid
  AI benar ~50%      → tidak tahu → bisa lanjut
  AI jawab "Tidak tahu" > 50% → tidak tahu → bisa lanjut
```

Kalau AI tahu hasil

Backtest historis tidak valid. Harus live prediction:

```
1. Tunggu laga belum main
2. Tanya AI sebelum kickoff
3. Catat prediksi
4. Tunggu hasil
5. Bandingkan
6. Kumpulkan 100+ laga
```

Butuh 2-4 minggu.

---

FASE 2: PILIH AI

AI gratis (rekomendasi mulai)

Provider Model Free tier
Gemini gemini-2.0-flash 15 RPM, 1500/hari
Groq llama-3.3-70b 30 RPM
Cerebras llama-3.1-70b 30 RPM
OpenRouter (beberapa) Terbatas

Cara ambil API key

Gemini:

```
1. Buka https://aistudio.google.com/app/apikey
2. Login Google
3. Create API Key
4. Copy
```

Groq:

```
1. Buka https://console.groq.com/keys
2. Login
3. Create API Key
4. Copy
```

Install library

```
pip install google-generativeai
pip install groq
```

Set env

```
export GEMINI_KEY="..."
export GROQ_KEY="..."
echo 'export GEMINI_KEY="..."' >> ~/.bashrc
echo 'export GROQ_KEY="..."' >> ~/.bashrc
source ~/.bashrc
```

Test

```
python -c "
import google.generativeai as genai
import os
genai.configure(api_key=os.environ['GEMINI_KEY'])
m = genai.GenerativeModel('gemini-2.0-flash')
r = m.generate_content('Tes. Jawab: OK')
print(r.text)
"
```

---

FASE 3: DESAIN PROMPT

Prompt yang baik

```
Prediksi Over/Under 2.5 untuk [Home] vs [Away] di [Liga].

Konteks:
- [Home]: posisi [X], 5 laga terakhir [hasil]
- [Away]: posisi [Y], 5 laga terakhir [hasil]

Jawab HANYA:
PILIHAN: Over / Under / Tidak tahu
ALASAN: (max 20 kata)
```

Yang harus ada

```
✅ Nama tim + liga
✅ Konteks form terkini
✅ Format output tegas
✅ Temperature = 0
```

Yang harus dihindari

```
❌ Kasih tanggal — AI tahu hasil
❌ Kasih skor historis — AI tahu hasil
❌ Prompt terlalu panjang — token mahal
❌ Minta AI hitung statistik — AI tidak bisa
```

---

FASE 4: ENSEMBLE LOGIC

Opsi A: Majority vote

```
3 AI:
  2 Over, 1 Under → Over
  1 Over, 2 Under → Under
  1 Over, 1 Under, 1 Tidak tahu → skip
```

Opsi B: Confidence-weighted

```
AI kasih confidence 0-100:
  Score = rata-rata
  Score > 70 → taruhan
```

Opsi C: Hanya kalau setuju

```
Semua AI Over  → taruhan Over
Semua AI Under → taruhan Under
Beda           → skip
```

Rekomendasi: C dulu. Paling konservatif.

---

FASE 5: LIVE TEST (30 HARI)

Kenapa live

AI tidak bisa backtest historis (look-ahead). Harus live.

Cara

```
1. Setiap hari, fetch jadwal laga < 24 jam
2. Tanya AI untuk tiap laga
3. Catat: laga, AI, prediksi, odds, timestamp
4. Tunggu hasil
5. Update: menang/kalah, P/L
```

File output

```
predictions_ai.csv:
tanggal, liga, home, away, ai_name, prediksi, confidence,
odds_over, odds_under, hasil, menang, pl
```

Setelah 100 laga

```
[ ] Hitung ROI per AI
[ ] Hitung ROI ensemble
[ ] Cek konsistensi per minggu
[ ] AI mana paling akurat
```

Yang harus dihindari

```
❌ Taruhan uang sebelum 100 laga
❌ Ganti prompt di tengah test
❌ Ganti AI di tengah test
❌ Skip hari
```

---

FASE 6: KEPUTUSAN

Kalau ROI positif

```
[ ] Lanjut produksi
[ ] Skala lebih banyak laga
[ ] Monitoring mingguan
```

Kalau ROI negatif

```
[ ] BERHENTI. Jangan tuning.
[ ] Jangan ganti prompt.
[ ] Jangan ganti AI.
[ ] Pilihan:
    a. Ganti pasar (O/U → 1x2, BTTS)
    b. Ganti AI (Gemini → GPT-4)
    c. Ganti liga (top → kecil)
    d. Berhenti total
```

---

STRUKTUR PROYEK AI

```
ai-prediction/
├── TEMPLATE_AI.md       ← file ini
├── ROADMAP.md           ← rencana
├── keys.env             ← API keys (JANGAN commit)
├── fetch_jadwal.py      ← jadwal hari ini
├── ai_client.py         ← wrapper Gemini + Groq
├── prompt.txt           ← template prompt
├── predict.py           ← orkestrator
├── evaluate.py          ← hitung ROI
├── predictions_ai.csv   ← data
└── harian.sh            ← cron
```

---

CONTOH SKRIP MINIMAL

ai_client.py

```python
import os
import google.generativeai as genai
from groq import Groq

def tanya_gemini(prompt):
    genai.configure(api_key=os.environ["GEMINI_KEY"])
    m = genai.GenerativeModel("gemini-2.0-flash")
    r = m.generate_content(prompt, generation_config={"temperature": 0})
    return r.text

def tanya_groq(prompt):
    c = Groq(api_key=os.environ["GROQ_KEY"])
    r = c.chat.completions.create(
        model="llama-3.3-70b-versatile",
        messages=[{"role": "user", "content": prompt}],
        temperature=0,
    )
    return r.choices[0].message.content
```

predict.py

```python
from ai_client import tanya_gemini, tanya_groq

def prediksi(home, away, liga, form_h, form_a):
    prompt = f"""Prediksi Over/Under 2.5 untuk {home} vs {away} di {liga}.
Konteks:
- {home}: {form_h}
- {away}: {form_a}
Jawab HANYA: PILIHAN: Over / Under / Tidak tahu"""
    return {
        "gemini": tanya_gemini(prompt),
        "groq": tanya_groq(prompt),
    }
```

evaluate.py

```python
import csv

rows = list(csv.DictReader(open("predictions_ai.csv")))
settled = [r for r in rows if r["hasil"]]

for ai in ["gemini", "groq", "ensemble"]:
    picks = [r for r in settled if r.get(f"{ai}_pick")]
    n = len(picks)
    if n == 0:
        continue
    w = sum(1 for r in picks if r[f"{ai}_menang"] == "1")
    pl = sum(float(r[f"{ai}_pl"]) for r in picks)
    print(f"{ai}: n={n} menang={w} ({w/n*100:.1f}%) P/L={pl:+.2f} ROI={pl/n*100:+.2f}%")
```

---

ATURAN EMAS AI

```
1. UJI LOOK-AHEAD DULU.
   Kalau AI tahu hasil, backtest tidak valid.
2. Validasi LIVE, bukan historis.
3. Minimal 2 AI untuk ensemble.
4. Temperature = 0.
5. Prompt sama untuk semua AI.
6. Jangan kasih tanggal ke AI.
7. Jangan kasih skor historis ke AI.
8. Jangan taruhan sebelum 100 laga live.
9. Kalau ROI negatif, JANGAN tuning prompt.
10. Pasar top Eropa efisien. AI juga tidak bisa.
```

---

CHECKLIST

```
[ ] Hipotesis + metrik sukses tertulis
[ ] API key gratis (Gemini + Groq)
[ ] Uji look-ahead — AI tidak tahu hasil
[ ] 2+ AI siap
[ ] Prompt sama untuk semua AI
[ ] Temperature = 0
[ ] Live test 30 hari, 100 laga
[ ] Catat semua prediksi + hasil
[ ] Setelah 100 laga: hitung ROI
[ ] Kalau positif: lanjut. Negatif: berhenti.
[ ] Jangan tuning prompt setelah gagal
[ ] Jangan taruhan uang sebelum 100 laga
```

---

YANG HARUS DIHINDARI (dari proyek statistik)

```
❌ Tuning parameter tanpa henti
❌ Keputusan dari < 100 laga
❌ Cari kombinasi ajaib
❌ Bolak-balik tempel-menempel
❌ Regenerate tanpa backup
❌ Backup menumpuk
❌ Lanjut walau sudah jelas gagal
```

---

LANGKAH PERTAMA

```
1. Ambil API key:
   - Gemini: https://aistudio.google.com/app/apikey
   - Groq:   https://console.groq.com/keys

2. Set env:
   export GEMINI_KEY="..."
   export GROQ_KEY="..."

3. Test 1 request:
   python -c "import google.generativeai as genai; ..."

4. Tulis test_ai_lookahead.py:
   - 20 laga historis
   - Tanya AI
   - Bandingkan dengan hasil

5. Kalau AI tidak tahu → lanjut live test.
   Kalau AI tahu → live test saja.
```

---

SATU PARAGRAF PENUTUP

AI ensemble mungkin punya edge. AI punya konteks yang model statistik tidak punya — berita, cedera, formasi. Tapi jebakan look-ahead nyata. Uji dulu. Kalau tidak tahu, live test 100 laga. Kalau ROI positif, lanjut. Kalau negatif, berhenti. Jangan ulangi kesalahan proyek statistik. Jangan tuning prompt. Jangan cari kombinasi ajaib. Jangan taruhan sebelum 100 laga.

---
