# Speech therapist website – setup guide
Built by **SmartMenuKJ** · https://smartmenukj.com

One self-contained file (`index.html`) hosted on **GitHub Pages**, with **Firebase** (Firestore + Authentication) as the database and admin login.

Without a Firebase config, the site runs in **demo mode**: everything works, but data stays in the visitor's browser. Demo admin login: `admin@demo.bg` / `demo123`.

---

## 1. Personalise the site (5 min)

Open `index.html` and search for `CLIENT SETTINGS`. Edit the `SITE` object:

| Field | What it is |
|---|---|
| `therapist`, `role`, `city` | Name, title, city |
| `address`, `phone`, `email` | Contact details (also used in the legal pages and notification texts) |
| `company`, `eik` | Legal entity name and ЕИК/БУЛСТАТ (required for the legal pages) |
| `policyDate` | Date the legal pages take effect |
| `photo` | Optional file name, e.g. `photo.jpg`, uploaded next to `index.html` |
| `stats` | The four figures in the blush band. Placeholders — replace with her real numbers or delete entries. |
| `bio`, `credentials`, `services` | About text, qualifications, services (add `price: "40 лв."` to show a price) |

Also update the `<title>` and `<meta name="description">` at the top of the file for Google.

## 2. Firebase (15 min, free Spark plan)

1. Go to https://console.firebase.google.com, then **Add project**. You can switch **Google Analytics off** (not needed, simpler for GDPR).
2. **Project settings → Your apps → Web (`</>`)** → register the app → copy the `firebaseConfig` object.
   Paste it into `index.html` in place of `const FIREBASE_CONFIG = null;`.
3. **Firestore Database → Create database** → **Production mode** → choose an **EU location** (e.g. `europe-west3` Frankfurt or `eur3`). The privacy policy assumes EU storage.
4. **Firestore → Rules** → replace everything with the contents of `firestore.rules` → **Publish**.
5. **Authentication → Get started → Email/Password → Enable**.
   - **Users → Add user**: the therapist's email and a strong password.
   - Copy the new user's **User UID**.
   - If your console offers it: **Settings → User actions**, untick "Enable create (sign-up)" so nobody else can register. Even without this, other accounts get no admin rights.
6. **Firestore → Start collection** `admins` → **Document ID = the UID** from step 5 → add a field `role` (string) = `admin` → Save.
7. **Authentication → Settings → Authorized domains** → add `YOUR-USERNAME.github.io` (and a custom domain if you use one).
8. Recommended: in **Google Cloud Console → APIs & Services → Credentials**, restrict the browser API key to HTTP referrers `https://YOUR-USERNAME.github.io/*` (and your domain).
9. Optional anti-spam: **App Check**. Note that reCAPTCHA is a Google service, so add it to the privacy policy if you enable it.

> The Firebase web `apiKey` is not a secret. Security comes from `firestore.rules`.

**What the rules enforce**
- Booking details (name, phone, email, note) can **only be read by the admin**.
- The public calendar reads only anonymous `slots` markers (date + time), so a taken hour disappears for everyone, live.
- A time slot can be created only once, so two people can never book the same hour.
- New reviews are always `pending` and invisible until approved.
- Articles, schedule and moderation can only be changed by the admin.

## 3. Publish on GitHub Pages (5 min)

1. https://github.com/new → repository name (e.g. `logoped`) → **Public** → Create.
2. **Add file → Upload files** → upload `index.html` and `.nojekyll` (and `photo.jpg` if used) → Commit.
3. **Settings → Pages** → Source: **Deploy from a branch** → Branch **main** / **(root)** → Save.
4. After about a minute the site is live at `https://YOUR-USERNAME.github.io/logoped/`.

**Custom domain (optional):** Settings → Pages → Custom domain → enter the domain. At the DNS provider, add a `CNAME` record pointing to `YOUR-USERNAME.github.io`. Then tick **Enforce HTTPS**. Add the domain to Firebase authorized domains (step 2.7).

Command line alternative:
```bash
git init && git add index.html .nojekyll && git commit -m "Initial site"
git branch -M main
git remote add origin https://github.com/YOUR-USERNAME/logoped.git
git push -u origin main
```

**Admin panel:** `https://…/#/admin` (also linked in the footer as "Вход за администратор").

## 4. EU / Bulgarian legal checklist

- [ ] Fill in `company`, `eik`, `address` and `policyDate`.
- [ ] Firestore location is in the EU (step 2.3).
- [ ] The booking form tells visitors **not** to enter health data. Keep it that way: speech and diagnosis notes belong in her offline records, not in this system.
- [ ] Never add made-up reviews. Fake or unverifiable reviews are banned under the EU Omnibus Directive and the Bulgarian Consumer Protection Act. The demo reviews exist only in demo mode.
- [ ] Use **"Изчисти стари записи"** in the admin panel every few months. It applies the retention periods stated in the privacy policy.
- [ ] If you add analytics, Maps embeds, reCAPTCHA or other third-party tools, update the privacy and cookie policies and add a consent banner with a "reject" option.
- [ ] Have the legal pages reviewed by a lawyer before launch. They are a solid template, not legal advice.

Fonts load from **Bunny Fonts** (EU, no cookies, no tracking) instead of Google Fonts, because of the German court ruling on Google Fonts and IP addresses.

## 5. Limits and possible upgrades

- There are no automatic emails or SMS. After confirming a booking, the admin panel shows **SMS** and **Имейл** buttons with a ready-written message, plus **В календара** for Google Calendar. Automatic sending needs the Blaze plan with Cloud Functions, or a service like EmailJS or Brevo.
- The free Spark plan (50k reads and 20k writes per day) is far more than a single practice needs.
- Article images are added by URL. Firebase Storage uploads need the Blaze plan.

---

# Ръководство за администратора (за логопеда)

**Вход:** отворете сайта → долу „Вход за администратор“ → имейл и парола.

**Записвания**
- **Предстоящи** показва всички бъдещи часове. Жълтите етикети чакат вашето потвърждение.
- **Потвърди** – часът остава запазен. След това натиснете **SMS** или **Имейл**, за да изпратите готово съобщение на клиента, и **В календара**, за да го добавите в Google Calendar.
- **Откажи** – часът отново става свободен в сайта.
- **Изтрий** – премахва записа напълно.
- **Добави запис** – за часове, уговорени по телефона. Те се потвърждават веднага и часът изчезва от сайта.
- **Изчисти стари записи** – натискайте на няколко месеца (изискване по GDPR).

**Отзиви** – новите отзиви не се виждат, докато не натиснете **Публикувай**. **Скрий** ги прибира, без да ги изтрива.

**Статии** – **Нова статия** → заглавие, кратко описание, текст. Отметнете „Публикувана“, за да се вижда. Форматиране: `## Заглавие`, `**удебелен**`, `*курсив*`, редове с `- ` за списък.

**Работно време** – работни дни и часове, продължителност на часа, почивка между часовете и неработни дни (отпуска, празници). Промените важат веднага.
