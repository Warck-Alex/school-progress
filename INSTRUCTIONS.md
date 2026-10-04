# Дневник успеваемости — инструкции

Приложение — один файл `index.html`. Данные хранятся в `data.json` (репозиторий `school-progress`), заявки учеников — в `pending.json` (репозиторий `school-pending`). Серверов, Firebase и VPN не нужно.

---

## 1. Установка

### 1.1. Два репозитория
1. Войдите на github.com → **New repository**.
2. Первый: имя `school-progress`, **Public**, поставьте «Add a README». Create.
3. Второй: имя `school-pending`, **Private** (или Public — токен ученика всё равно ограничен этим репозиторием), «Add a README». Create.

### 1.2. Два токена (Fine-grained Personal Access Token)
GitHub → аватар → **Settings** → **Developer settings** → **Personal access tokens** → **Fine-grained tokens** → **Generate new token**.

**Токен админа** (вводится в приложении, в HTML не вшивается):
- Token name: `school-admin`. Expiration: на максимум (напомните себе обновить).
- Repository access: **Only select repositories** → `school-progress`.
- Permissions → Repository permissions → **Contents: Read and write**.
- Generate → скопируйте `github_pat_…` (показывается один раз).

**Токен ученика** (вшивается в HTML):
- Token name: `school-student`.
- Repository access: **Only select repositories** → **только** `school-pending`.
- Permissions → **Contents: Read and write**.
- Generate → скопируйте.

> Токен ученика виден всем, кто откроет код страницы. Поэтому он должен иметь доступ **только** к `school-pending`. Худшее, что с ним можно сделать, — испортить файл заявок.

### 1.3. Что заменить в коде (блок констант в начале `<script>`)
| Константа | Что сделать |
|---|---|
| `SUPER_ADMIN_EMAIL` | Ваш реальный email (строчными буквами) |
| `SUPER_ADMIN_SALT` | Любая своя длинная случайная строка |
| `SUPER_ADMIN_PASSWORD_HASH` | SHA-256 от строки `СОЛЬ:ПАРОЛЬ` (см. ниже) |
| `GH_OWNER` | Ваш логин GitHub (в файле сейчас `Warck-Alex`) |
| `GH_REPO` | `school-progress` |
| `GH_PENDING.owner` | Ваш логин GitHub |
| `GH_PENDING.token` | Токен ученика (`github_pat_…`) |

**Как посчитать SHA-256.** Хеш считается от **соль + двоеточие + пароль**. Например, соль `my_salt_77`, пароль `MyPass#2026`:
- Linux/macOS: `printf '%s' 'my_salt_77:MyPass#2026' | sha256sum`
- Браузер: F12 → Console, вставьте
  ```js
  crypto.subtle.digest('SHA-256', new TextEncoder().encode('my_salt_77:MyPass#2026')).then(b => console.log([...new Uint8Array(b)].map(x => x.toString(16).padStart(2,'0')).join('')))
  ```
- Windows PowerShell: `$s='my_salt_77:MyPass#2026'; -join ([Security.Cryptography.SHA256]::Create().ComputeHash([Text.Encoding]::UTF8.GetBytes($s)) | % { $_.ToString('x2') })`

> Сейчас в файле стоит демо-пароль **`Admin#2026`** (соль `school_salt_superadmin_v1`). **Обязательно замените** его до публикации.

### 1.4. Загрузка и GitHub Pages
1. В `school-progress` → **Add file → Upload files** → загрузите `index.html` → Commit.
2. **Settings → Pages** → Source: **Deploy from a branch** → Branch: `main`, папка `/ (root)` → Save.
3. Через 1–2 минуты сайт будет по адресу `https://ВАШ-ЛОГИН.github.io/school-progress/`.

### 1.5. Создать `pending.json`
В репозитории `school-pending` → **Add file → Create new file** → имя `pending.json`, содержимое ровно:
```json
[]
```
Commit.

`data.json` создавать вручную не нужно: он появится при первой публикации админом.

### 1.6. Telegram-уведомления (необязательно)
1. В Telegram откройте **@BotFather** → `/newbot` → задайте имя и username → получите токен вида `123456:ABC…`.
2. Напишите вашему боту любое сообщение (например, `/start`).
3. Откройте в браузере `https://api.telegram.org/bot<ТОКЕН>/getUpdates` → найдите `"chat":{"id":123456789,…` — это `chat_id`.
4. Впишите в коде: `NOTIFY.telegramBot = '<ТОКЕН>'`, `NOTIFY.telegramChat = '<chat_id>'`.

Telegram шлёт сам браузер админа при обнаружении новой заявки, поэтому вкладка/приложение админа должны быть открыты. Если Telegram недоступен в вашей сети — оставьте поля пустыми (звук и браузерные уведомления работают независимо).

### 1.7. EmailJS (необязательно)
Без EmailJS код подтверждения при регистрации показывается на экране.
1. Зарегистрируйтесь на emailjs.com → **Email Services** → подключите почту → запомните `Service ID`.
2. **Email Templates** → создайте шаблон. В поле «To email» поставьте `{{to_email}}`, в тексте используйте `{{code}}` (например: «Ваш код: {{code}}»). Запомните `Template ID`.
3. **Account → General** → `Public Key`.
4. Впишите в `EMAILJS`: `serviceId`, `templateId`, `publicKey`.

---

## 2. Использование

### 2.1. Вход супер-админа
Супер-админ **не регистрируется**: он вшит в код. Откройте сайт → вкладка «Вход» → email и пароль из констант. При первом запуске запись создаётся автоматически. Его нельзя удалить, изменить роль или пароль через интерфейс (пароль меняется только в коде).

### 2.2. Первый запуск админа
1. Войдите как супер-админ.
2. Нажмите **＋** в шапке → введите ФИО ученика.
3. **Личный кабинет → Публикация → 📤 Опубликовать** → вставьте токен админа (сохранится на устройстве). Дальше публикация идёт автоматически после каждой правки (зелёная точка в шапке — всё опубликовано, оранжевая — есть неопубликованное).
4. Вкладка «Оценки» → **+ Предмет** (из каталога) → **+** в строке предмета → оценка, тип, дата, описание. Оценки сами сортируются по дате; кнопка «+» всегда последняя в строке. Порядок предметов — кнопками ▲/▼.
5. Каталог предметов правится в личном кабинете.

### 2.3. Приглашение пользователей
Личный кабинет → «✉️ Приглашение участников» → email, роль, ученик → «Создать приглашение». Приглашение публикуется вместе с данными. Приглашённому достаточно зарегистрироваться **с тем же email** (или загрузить файл-приглашение кнопкой «📥 Скачать» → на экране регистрации поле «Файл-приглашение»). Роль и привязка к ученику берутся из приглашения.
- Роль «Админ» может выдать только супер-админ.
- Без приглашения новый пользователь получает роль «Наблюдатель».
- Повторное приглашение снимает email с чёрного списка.

### 2.4. Регистрация и заявки
1. Пользователь: «Регистрация» → email, пароль, **ФИО ученика** → получает 3-значный код (на почту или на экране; 5 попыток, 10 минут).
2. ФИО нормализуется (регистр, лишние пробелы, ё/е), поэтому «Иванов  Иван» и «иванов иван» — один ученик.
3. Уходит заявка `new_user` / `new_student` в `pending.json`. Пользователь сразу может работать на своём устройстве; вход с других устройств заработает после одобрения.

### 2.5. Ученик отправляет заявки
Ученик (роль «Ученик») добавляет/меняет/удаляет оценки, уроки и пункты каталога — изменения показываются у него сразу, а админу уходит заявка: «📤 Заявка отправлена администратору». Статусы видны в панели «📨 Мои заявки». После следующего обновления данных у ученика остаются только одобренные изменения.

### 2.6. Админ одобряет
Админ опрашивает `pending.json` каждые 20 секунд: звук (800 Гц), браузерное уведомление, Telegram, красный бейдж на вкладке «Личный кабинет». Панель «🔔 Заявки» → **✅ Принять** (применяется и публикуется) или **✖ Отклонить** (можно указать причину). Супер-админ видит заявки всех, обычный админ — только своих учеников.

### 2.7. Публикация
- Автоматически — после любой правки (только у админов).
- Вручную: «📤 Опубликовать» или «📤» над таблицей оценок.
- «🔄 Обновить с GitHub» — принудительно загрузить свежие данные.
- Публикация делает слияние с тем, что лежит на GitHub (побеждает более поздняя правка ученика), поэтому два админа не затирают друг друга.
- ⏱ GitHub Pages обновляется с задержкой 1–2 минуты после публикации: ученики увидят новое не мгновенно. Админ с токеном читает данные через API без задержки.

### 2.8. PDF
- **Табель**: вкладка «Оценки» → «📄 PDF» (файл `tabel-ФИО-дата.pdf`).
- **Расписание**: вкладка «Расписание» → «📄 PDF» (файл `raspisanie-ФИО-дата.pdf`, альбомная страница).

Нужен интернет для загрузки библиотеки html2pdf.js (CDN).

### 2.9. Тема
Кнопка 🌙/☀️ в шапке; выбор сохраняется.

### 2.10. Управление пользователями (вкладка «Пользователи»)
Супер-админ видит всех; админ — только пользователей своих учеников. Смена роли — выпадающий список, удаление — 🗑 (email попадает в чёрный список `deletedEmails`). Нельзя: удалить себя, супер-админа, последнего админа; админ не может удалять других админов; роль супер-админа через интерфейс не назначается.

### 2.11. Сброс данных
«💾 Сбросить всё» (админ/супер-админ) — два предупреждения подряд. Очищаются оценки, расписание, каталоги, приглашения; аккаунты пользователей остаются. Супер-админ сбрасывает всё, обычный админ — только своих учеников.

---

## 3. Сборка APK (Android Studio)

1. **New Project → Empty Views Activity** (Kotlin), package, например, `by.school.diary`.
2. `app/src/main/AndroidManifest.xml` — до `<application>`:
   ```xml
   <uses-permission android:name="android.permission.INTERNET" />
   ```
   и в `<application>` добавьте `android:usesCleartextTraffic="false"`.
3. `res/layout/activity_main.xml`:
   ```xml
   <?xml version="1.0" encoding="utf-8"?>
   <WebView xmlns:android="http://schemas.android.com/apk/res/android"
       android:id="@+id/web" android:layout_width="match_parent" android:layout_height="match_parent" />
   ```
4. `MainActivity.kt`:
   ```kotlin
   package by.school.diary

   import android.content.ContentValues
   import android.os.Bundle
   import android.os.Environment
   import android.provider.MediaStore
   import android.util.Base64
   import android.webkit.*
   import android.widget.Toast
   import androidx.activity.OnBackPressedCallback
   import androidx.appcompat.app.AppCompatActivity

   class MainActivity : AppCompatActivity() {
       private lateinit var web: WebView
       private val url = "https://ВАШ-ЛОГИН.github.io/school-progress/"

       override fun onCreate(savedInstanceState: Bundle?) {
           super.onCreate(savedInstanceState)
           setContentView(R.layout.activity_main)
           web = findViewById(R.id.web)
           web.settings.javaScriptEnabled = true
           web.settings.domStorageEnabled = true        // localStorage — обязательно
           web.webViewClient = WebViewClient()
           web.webChromeClient = WebChromeClient()
           web.addJavascriptInterface(Bridge(), "AndroidBridge")
           web.loadUrl(url)
           onBackPressedDispatcher.addCallback(this, object : OnBackPressedCallback(true) {
               override fun handleOnBackPressed() { if (web.canGoBack()) web.goBack() else finish() }
           })
       }

       // Мост для скачивания файлов из страницы (PDF и JSON) в папку «Загрузки»
       inner class Bridge {
           @JavascriptInterface fun saveFile(name: String, mime: String, text: String) = save(name, mime, text.toByteArray())
           @JavascriptInterface fun savePdf(dataUri: String, name: String) =
               save(name, "application/pdf", Base64.decode(dataUri.substringAfter("base64,"), Base64.DEFAULT))
           private fun save(name: String, mime: String, bytes: ByteArray) {
               val v = ContentValues().apply {
                   put(MediaStore.Downloads.DISPLAY_NAME, name)
                   put(MediaStore.Downloads.MIME_TYPE, mime)
                   put(MediaStore.Downloads.RELATIVE_PATH, Environment.DIRECTORY_DOWNLOADS)
               }
               val uri = contentResolver.insert(MediaStore.Downloads.EXTERNAL_CONTENT_URI, v)
               uri?.let { contentResolver.openOutputStream(it)?.use { o -> o.write(bytes) } }
               runOnUiThread { Toast.makeText(this@MainActivity, "Сохранено в «Загрузки»: $name", Toast.LENGTH_LONG).show() }
           }
       }
   }
   ```
   Приложение само находит `AndroidBridge` и сохраняет PDF/JSON через него (иначе в WebView скачивание файлов из страницы не работает). Требуется Android 10+ (API 29); для более старых версий нужно разрешение на запись.
5. **Build → Build Bundle(s) / APK(s) → Build APK(s)**. Готовый файл: `app/build/outputs/apk/debug/app-debug.apk`. Для установки на телефон разрешите «Установку из неизвестных источников».

---

## 4. Важно знать (ограничения и безопасность)

- **`data.json` публичный.** В нём лежат email, ФИО, оценки и хеши паролей (с солью). Используйте надёжные пароли и не храните там ничего секретного. Приватный репозиторий с GitHub Pages — только на платном плане.
- **Без EmailJS код регистрации виден на экране**, то есть почту он не подтверждает. Для проверки личности настройте EmailJS. В заявке на сброс пароля админу видно, как был получен код.
- **Админский токен** лежит в `localStorage` устройства — не вводите его на чужих устройствах; «Забыть токен» удаляет его.
- **Токен ученика истекает** вместе со своим сроком — тогда заявки перестанут уходить; создайте новый и обновите `GH_PENDING.token`.
- **Если у ученика (`createdBy = 'self'`) нет админа** (самостоятельная регистрация с новым ФИО без приглашения), его заявки видит только супер-админ.
- **Перетаскивание уроков** работает мышью и на большинстве Android WebView долгим нажатием; запасной вариант — поменять день в окне «Изменить урок».
- При открытии `index.html` прямо с диска (`file://`) браузер блокирует загрузку `data.json` — открывайте сайт через GitHub Pages или локальный сервер.
