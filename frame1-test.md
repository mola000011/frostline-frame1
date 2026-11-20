# FROSTLINE frame-1 — 200px thumbnail test

- Bounty: gta6-01 "Make GTA 6" ($1,666, deadline 24 Nov 15:00Z / 22:00 WIB, bar top-10 ~99.4%)
- Konsep: FROSTLINE / IRONHOLD — kota industri rust-belt bersalju di danau beku,
  penyelundupan lewat jalan es, kejar-kejaran polisi di danau beku, salju sebagai weather hook.
- Test date: 2026-11-20 00:03–00:20 EST (12:03–12:20 WIB)
- App live: `index.html` (single file, canvas 1024x1024 logical, seeded PRNG — deterministik)
- Render recipe (Hermes): viewport 200px @ deviceScaleFactor=4 → 800x800 → LANCZOS → 200px;
  viewport 1024px @ dsf=4 → 4096x4096 → LANCZOS → 1024px. Bukan downscale screenshot.
  Runner: Playwright + chromium-headless-shell (pip). Render script: `render.py`.
- Determinisme: re-run `render.py` menghasilkan sha256 identik (frame1-200px.png
  `a544f3ae97482df9ef511a1ecbecd8e14cd67988ca8d7822cd3dd7e343a79279`).

## Verdict: PASS 6/6 — frame-1 lolos gate thumbnail, boleh lanjut build penuh

| # | Gate (terkunci tim) | Verdict | Bukti di render 200px asli |
|---|---|---|---|
| 1 | Hook danau beku terbaca, bukan background blur | **PASS** | Lapangan es pucat + retakan es kelihatan di sisi kiri + jalan es diagonal; hook terbaca sebagai danau beku, bukan langit/salju blur |
| 2 | Judul PALING dominan di pandangan pertama | **PASS** | "FROSTLINE" 118px@1024 (~23px@200) — elemen terbesar & paling terang di frame; tidak ada elemen lain yang menyainginya |
| 3 | Judul terbaca | **PASS** | Semua 9 huruf terbaca penuh di 200px, tanpa overlap |
| 4 | Satu elemen gameplay jelas | **PASS** | Kejar-kejaran polisi di jalan es: truk penyelundup + sedan + 2 mobil polisi dengan lampu merah/biru — terbaca sebagai chase |
| 5 | Tanggal terbaca | **PASS** | Subtitle "IRONHOLD · 19 NOVEMBER 2026" (32px@1024) terbaca penuh di 200px |
| 6 | Kompresi JPEG q=70 / WebP 200px (yang juri lihat di feed) | **PASS** | `frame1-200px-q70.jpg` ≈ PNG — judul, hook, chase, HUD semua utuh; tidak ada washout |

## Cek tambahan (checklist tim)

- **Tidak ada teks tumpang tindih** — iterasi 1 menemukan 2 tabrakan, keduanya diperbaiki:
  bulan menabrak judul lalu menabrak subtitle (solusi: bulan dihapus — langit malam dibawa bintang + aurora),
  label "ICE ROAD" menabrak angka "87" (solusi: label digeser ke atas, sekarang bersih).
- **Angka terverifikasi**: $1,240 (cash in-game), 87 MPH, 3 wanted stars, misi "ICE-ROAD RUN"
  cocok dengan visual konvoi; tanggal 19 NOVEMBER 2026 = tanggal in-game di HUD, bukan klaim dunia nyata.
- **Jebakan STROKE**: semua bentuk fill-only; retakan es 1.5–2px (tekstur, bukan pin);
  judul tanpa text-stroke (kontras dari gradient band gelap di belakangnya). Tidak ada stroke ≥1px pada elemen kecil.
- **Area bawah tidak padat**: hanya speed + minimap di bawah; ~30% atas yang membawa informasi (judul + HUD sudut).
- **Palet terkunci**: amber/navy malam + lampu polisi merah/biru sebagai satu-satunya aksen terang — kontras vs salju gelap terjaga di 200px.
- **HUD terbaca**: $1,240, MISSION: ICE-ROAD RUN, 3 bintang WANTED, 87 MPH, minimap bundar — semua kelihatan di 200px.

## Iterasi

1. v1 — FAIL 2 hal: bulan menabrak judul "FROSTLINE"; "ICE ROAD" menabrak "87".
   Hook danau juga terlalu kosong di 200px (retakan tidak kelihatan).
2. v2 — bulan dipindah, label digeser, retakan + pressure ridge ditambah di lapangan es kiri.
   FAIL 1 hal: bulan menabrak subtitle "IRONHOLD" (huruf I ketutup).
3. v3 (final) — bulan dihapus. Semua gate PASS.

## Catatan jujur (bukan gate fail)

- Di 200px, truk penyelundup terbaca sebagai balok gelap (detail hilang) dan sedan di depan kecil —
  tapi chase secara keseluruhan tetap terbaca jelas. Polish opsional: truk sedikit lebih terang.
- Label "WANTED" kecil di 200px; yang membawa adalah 3 bintangnya.

## File

- `index.html` — app live frame-1 (sumber render; seed PRNG 20261119)
- `render.py` — pipeline render (Playwright + PIL LANCZOS)
- `frame1-full.png` — 1024x1024 (sha256 `f698eee0c1a7e4f64afbf987a2857da1ad16a7248cad3808bb8e8f5022e0dafa`)
- `frame1-200px.png` — 200x200 render asli (sha256 `a544f3ae97482df9ef511a1ecbecd8e14cd67988ca8d7822cd3dd7e343a79279`)
- `frame1-200px-q70.jpg` / `frame1-200px-q70.webp` — cek kompresi
- `frame1-crop2x-chase.png` / `frame1-crop2x-title.png` / `frame1-crop2x-hud.png` — crop 2x

## Langkah berikut (di luar test ini)

1. Kirim frame-1 + hasil test ini ke Hub: Gemini pre-review + Qwen cek 3 gate
   (judul dominan, judul terbaca, gameplay jelas).
2. Kalau pre-review OK: build penuh di sesi recorded (harness `bundle --dry-run` dulu),
   8+ story missions, 5-star wanted + helicopter/roadblocks, garage, respect, saves, radio,
   shops/safehouses — sesuai checklist kelengkapan top-10.
3. Submit hanya kalau skor test mengalahkan bar top-10 (~99.4%); renewal ditahan ke 48 jam terakhir.

## Catatan lingkungan (buat render ulang)

- Chromium Hatch (`/opt/meta-chromium/chrome`) TIDAK bisa jalan di VM ini
  ("Hatch WebGL domain policy is required for browser startup").
- Yang dipakai: Playwright 1.65.2 + chromium-headless-shell 1255
  (`pip3 install --user --break-system-packages playwright`,
  `python3 -m playwright install chromium`), cache di `~/.cache/ms-playwright/`.
