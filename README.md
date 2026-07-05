# Fel7o Downloader (Electron edition)

## تشغيل المشروع أول مرة (على جهازك، ويندوز)

```
git clone https://github.com/Ahmed77khaled/Fel7o-Music.git
cd Fel7o-Music
npm install
npm start
```

هيفتح نافذة Electron فيها الواجهة الجديدة.

## محتاج قبل ما تشتغل فعليًا

المشروع بيجيلك **جاهز** — كل حاجة موجودة في مجلد `bin/` أول ما تعمل `npm install`:

1. **yt-dlp.exe** — موجود بالفعل جوه `bin/yt-dlp.exe`، مفيش داعي تحمّله يدوي.

2. **ffmpeg.exe** — مضغوط في `bin/ffmpeg.zip` (عشان حجم الملف كان أكبر من حد GitHub
   لملفات الريبو العادية). أول ما تعمل `npm install`، سكريبت `postinstall`
   (`scripts/extract-ffmpeg.js`) بيفكه تلقائيًا ويحطه `bin/ffmpeg.exe` — مفيش
   أي خطوة يدوية مطلوبة منك.

   لو لأي سبب مفكش تلقائي، تقدر تشغّله يدوي:
   ```
   node scripts/extract-ffmpeg.js
   ```

## بناء نسخة exe نهائية للتوزيع

```
npm run dist
```

الناتج هيبقى في مجلد `release/win-unpacked/` — ده المجلد اللي تدّيه للمستخدمين
(فيه `Fel7o Downloader.exe` + كل ملفات Electron/Chromium المطلوبة، بالظبط
زي شكل Vidora اللي شفته).

## ملاحظات

- التصميم بيستخدم ألوان/نظام تصميم premium (ألوان، كروت، أنيميشن) حسب
  البرومبت اللي حددته.
- المنطق (queue, history, settings, progress parsing) شغال فعليًا مش mockup،
  بس محتاج اختبار حقيقي على جهازك مع رابط يوتيوب فعلي.
- لو yt-dlp أو ffmpeg مش موجودين لأي سبب، الواجهة هتبين رسالة خطأ واضحة في
  شريط الحالة تحت.
