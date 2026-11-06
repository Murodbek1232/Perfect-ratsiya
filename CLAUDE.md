# CLAUDE.md

## Muloqot
- Foydalanuvchi bilan doim **o'zbek tilida** (lotin yozuvida) gaplash. Javoblar qisqa va oddiy bo'lsin.
- Foydalanuvchi asosan telefondan ishlaydi: ko'rsatmalarni bosqichma-bosqich, oddiy tilda ber.
- Ilova ichidagi barcha matnlar (tugmalar, xabarlar) ham o'zbek tilida bo'lishi kerak.

## Loyiha haqida
**RATSIYA — Ochiq kanal**: brauzerda ishlaydigan push-to-talk (ratsiya) ilovasi. Telefonga PWA sifatida o'rnatiladi.
Foydalanuvchi kanal nomi va ismini kiritadi, katta tugmani bosib ushlab gapiradi, qo'yib yuborganda ovozli xabar shu kanaldagi hammaga yuboriladi.

Build tizimi, npm yoki framework yo'q — oddiy statik fayllar:

| Fayl | Vazifasi |
|---|---|
| `index.html` | Butun ilova: HTML, CSS (`<style>`) va JavaScript (`<script>`) bitta faylda |
| `manifest.json` | PWA manifesti (nom, ikonkalar, ranglar) |
| `service-worker.js` | Offline kesh. **Fayllarni o'zgartirganda `CACHE_NAME` versiyasini oshir** (`ratsiya-v1` → `ratsiya-v2`), aks holda telefonlarda eski versiya qoladi |
| `icon-192.png`, `icon-512.png` | Ilova ikonkalari |
| `.claude/skills/` | Claude Code uchun skill'lar (ilova kodi emas) |

## Ilova qanday ishlaydi (`index.html` ichidagi JS)
- Ovoz `MediaRecorder` bilan yoziladi (`audio/webm;codecs=opus`), base64 ga aylantiriladi.
- Xabarlar `window.storage` orqali saqlanadi: kalit `msg:<kanal>:<vaqt>:<userId>`, qiymat JSON `{user, uid, ts, audio, mime}`.
- Har 1.5 soniyada `pollMessages()` yangi xabarlarni tekshiradi va avtomatik ijro etadi (agar "Ovozni yoqish" bosilgan bo'lsa).
- 2 soatdan eski xabarlar `cleanupOld()` bilan o'chiriladi.
- Ism va oxirgi kanal ham `window.storage` da saqlanadi.

### Muhim cheklov
`window.storage` — bu **Claude artifact** muhitining API'si. Ilova Claude artifact ichida ochilganda ishlaydi,
lekin oddiy veb-sayt (masalan, GitHub Pages) sifatida joylansa, `window.storage` mavjud bo'lmaydi va xabar almashish ishlamaydi.
Mustaqil hosting uchun haqiqiy backend kerak (masalan, Firebase, Supabase yoki WebSocket server).
Bu haqda foydalanuvchiga eslatib tur va backend tanlashni undan so'ra.

## Kod uslubi
- Hamma narsa `index.html` ichida qoladi, alohida build qadamisiz. Tashqi kutubxona qo'shishdan oldin foydalanuvchidan so'ra.
- Ranglar `:root` dagi CSS o'zgaruvchilaridan olinadi (`--amber`, `--green`, `--red` va h.k.). Yangi rang qo'shsang, o'sha yerga qo'sh.
- Dizayn: qorong'i "harbiy ratsiya" uslubi, shriftlar JetBrains Mono va Oswald. Mobil ekran birinchi o'rinda (`max-width: 420px`).
- Foydalanuvchi kiritgan matnni HTML'ga qo'yishda `escapeHtml()` dan foydalan.

## Tekshirish
Avtomatik testlar yo'q. O'zgartirishdan keyin:
1. `python3 -m http.server 8000` bilan ishga tushirib, Playwright bilan (`webapp-testing` skill) sahifa xatosiz ochilishini va konsolda xato yo'qligini tekshir.
2. Mobil o'lchamda (masalan, iPhone emulyatsiyasi) skrinshot olib ko'r.
3. Mikrofon va `window.storage` brauzer testida to'liq ishlamasligi mumkin — buni foydalanuvchiga ayt.
