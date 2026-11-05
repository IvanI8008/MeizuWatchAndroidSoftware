# MeizuWatchAndroidSoftware
## English

### About

Meizu Watch is a 2016 smartwatch. Its official companion app (`com.meizu.watch`, v1.6.3) was built for the Android 6–7 era and is effectively broken on modern phones: watch discovery never started, the app crashed on launch or hung, the watch refused to connect, and some screens were dead. Meizu's cloud, which the app relied on, has long been shut down.

This project restores the app **without rewriting it from scratch**. The original 1.6.3 APK has been carefully patched at the bytecode level: all critical incompatibilities with modern Android are fixed, plus a few quality-of-life improvements the stock app never had. Tested on a Samsung Galaxy S25 Ultra (One UI 9): the watch is discovered, pairs and syncs; notifications, alarms and "find phone" all work.

Meizu never released sources (the app is closed-source), so the "source code" here is a full decompiled APK build (smali + resources + manifest) from which the app can be rebuilt one-to-one.

### Language & stack

- **Original app** — Java (Android SDK, 2016): BLE/GATT link to the watch, a custom binary protocol (~120 commands), EventBus, plus era-typical social SDKs (Facebook/Weibo/QQ — unused today).
- **Patches** — smali (Dalvik bytecode assembler), applied by automated Python scripts that verify each replaced block.
- **Toolchain** — APKEditor (APK decode/build), uber-apk-signer (v1+v2+v3 signing), zipalign; root-free installation via InstallerX + Shizuku.

### Differences from stock 1.6.3

1. **Watch discovery works.** Stock code aborted BLE scanning on a location-provider check under modern Android, so the watch was never found. Fixed.
2. **The watch connects automatically.** The BLE engine (LocalManagerService) used to be started only by a 32-bit native daemon that cannot exist on modern 64-bit phones. The service now starts with the app itself — connection and sync happen on their own.
3. **Modern Android compatibility.** targetSdk raised to 30 (Android 11); the startup crash caused by `getDeviceId` (IMEI) is gone — replaced with `ANDROID_ID`, which needs no forbidden permissions.
4. **Alarm: a "Stop" button.** When the watch rang its alarm or searched for the phone, the ringtone on the phone could not be dismissed at all (2016 design — it rang forever; only a reboot helped). Every ring now shows a heads-up notification with a **Stop** button, the sound auto-stops after 60 seconds, and re-triggers within 10 minutes no longer restart it.
5. **Full app list in reminders.** On Android 11+ the stock "remind me from apps" picker showed only 2–3 apps. With the `QUERY_ALL_PACKAGES` permission all installed apps are visible.
6. **The back arrow works.** The title-bar arrow synthesized a system Back press (`injectInputEvent`), forbidden since Android 9, so the button was dead. It now performs a regular `onBackPressed()`, same as the edge-swipe gesture.
7. **Double vibration length.** The protocol has no duration field (the vibration pattern lives in the watch firmware), so the notification command is now sent twice, 1.2 s apart — the watch vibrates for two consecutive cycles. Applies to app notifications, SMS and calls.
8. **32-bit native libraries removed** — they cannot run on 64-bit devices and broke installation; their job is done by the app's own code.
9. **Hobbyist-friendly diagnostics:** uncaught exceptions are logged to logcat under the `MZWatch` tag, and a hung process is restarted automatically.
10. **Quiet logcat.** The app's and bundled SDKs' debug/info/verbose logging (Log.d/v/i) is disabled; logcat keeps only warnings, errors and crashes — including the project's diagnostics (`MZWatch` tag).

### Still not working

Meizu's cloud (`meiklife.cn`) is discontinued: Meizu account login, cloud sync and over-the-app watch firmware updates are unavailable. Local features (connection, notifications, alarms, find-phone, pedometer) are unaffected.

### Installation

**Option A — no PC (recommended):**
1. Install [Shizuku](https://shizuku.rikka.app/) (enable wireless debugging in Developer options).
2. Install [InstallerX](https://github.com/RikkaApps/InstallerX/releases).
3. Open the APK from this package with InstallerX and confirm.

**Option B — via ADB:** `adb install -r "Meizu_Watch_v1_skia81.apk"`

**About the signature.** The build is signed with uber-apk-signer's built-in debug key (valid until 2044; everyone who builds with this tool uses the same key). Consequences:
- updates install **on top of** stock 1.6.3 or previous builds of this project — pairing, settings and data are preserved;
- if you build your own version with a **different** key, you must uninstall the old app first and re-pair the watch.

### Building from source

1. Install JDK 8+.
2. Download [APKEditor](https://github.com/Re/APKEditor/releases) and [uber-apk-signer](https://github.com/patrickfav/uber-apk-signer/releases); put the JARs into `build/tools/`.
3. Run `build/build.sh` (Linux/macOS/WSL). The signed, aligned APK appears next to the script.

The `source/` tree already contains all patches. The scripts in `patches/` are the automation that applied them (idempotent: no-op on an already-patched tree); useful if you want to reproduce the patches on another stock version.

### Package layout

```
Meizu_Watch_v1_skia81/
├── Meizu_Watch_v1_skia81.apk     — the ready app
├── README.md                      — this file
├── build/
│   └── build.sh                   — builds the APK from source/
├── patches/                       — Python patch scripts (v8…v12 + logtrim)
└── source/                        — full APK build project
    ├── AndroidManifest.xml        — manifest (targetSdk 30, QUERY_ALL_PACKAGES, AlarmStop receiver)
    ├── smali/                     — app code (Dalvik bytecode, 4298 files)
    ├── resources/                 — decoded resources
    ├── root/                      — assets and other APK-root files
    └── signatures/                — APKEditor signature metadata
```

### Disclaimer

This is a non-commercial community project for Meizu Watch owners. All rights to the original app belong to Meizu Technology Co., Ltd. Provided as is, with no warranty of any kind.

# Meizu Watch — Revival Edition · v1 (skia81)

> Приложение-компаньон для смарт-часов **Meizu Watch (2016)**, возвращённое к жизни на современных Android-смартфонах.
> The companion app for the **2016 Meizu Watch**, brought back to life on modern Android phones.

**Первая стабильная сборка / First stable release — v1 (skia81), ноябрь 2026 / November 2026**

---

## Русский

### О проекте

Meizu Watch — смарт-часы 2016 года. Их штатное приложение-компаньон (`com.meizu.watch`, версия 1.6.3) было рассчитано на Android 6–7 эры и на современных смартфонах фактически не работало: поиск часов не запускался, приложение падало при старте или висло, часы не подключались, а часть экранов оказалась «мёртвой». Облако Meizu, на которое опиралось приложение, давно отключено компанией.

Этот проект восстанавливает работоспособность приложения **без переписывания с нуля**. Оригинальный APK 1.6.3 аккуратно пропатчен на уровне байткода: исправлены все критические несовместимости с современным Android и добавлено несколько удобств, которых не было в стоке. Проверено на Samsung Galaxy S25 Ultra (One UI 9): часы находятся, сопрягаются, синхронизируются, уведомления, будильники и поиск телефона работают.

Исходников у Meizu никогда не было (приложение закрытое), поэтому «исходный код» здесь — это полная декомпилированная сборка APK (smali + ресурсы + манифест), из которой приложение собирается обратно один в один.

### Язык и стек

- **Оригинальное приложение** — Java (Android SDK 2016 года): BLE/GATT-связь с часами, собственный бинарный протокол (~120 команд), EventBus, набор SDK соцсетей тех лет (Facebook/Weibo/QQ — сейчас не используются).
- **Правки** — smali (ассемблер Dalvik-байткода), вносились автоматизированными Python-скриптами с точной проверкой каждого заменяемого блока.
- **Инструментарий** — APKEditor (декод/сборка APK), uber-apk-signer (подпись v1+v2+v3), zipalign; установка без root — через InstallerX + Shizuku.

### Отличия от стоковой версии 1.6.3

1. **Поиск часов работает.** Стоковый код на новых Android прерывал BLE-сканирование на проверке геолокации — часы никогда не находились. Исправлено.
2. **Часы подключаются автоматически.** Движок BLE-связи (LocalManagerService) в стоке запускался только из 32-битного нативного демона, которого на современных 64-битных телефонах не существует. Теперь сервис стартует при запуске приложения — соединение и синхронизация устанавливаются сами.
3. **Совместимость с современным Android.** Поднят targetSdk до 30 (Android 11); убрано падение при старте из-за `getDeviceId` (IMEI) — заменено на `ANDROID_ID`, не требующее запрещённых прав.
4. **Будильник: кнопка «Остановить».** Когда часы звонят будильником или ищут телефон, в стоке выключить мелодию на телефоне было невозможно (дизайн 2016 года — «звенело вечно», спасала только перезагрузка). Теперь при каждом срабатывании появляется уведомление с кнопкой «Остановить», мелодия автоматически гасится через 60 секунд, а повторные срабатывания в течение 10 минут её заново не включают.
5. **Полный список приложений в напоминаниях.** В стоке на Android 11+ в «напоминании из приложений» были видны только 2–3 программы. Добавлено разрешение `QUERY_ALL_PACKAGES` — видны все установленные приложения.
6. **Кнопка «назад» работает.** Стрелка в левом верхнем углу синтезировала системное нажатие Back (`injectInputEvent`), что на Android 9+ запрещено, — кнопка была мёртвой. Теперь она вызывает штатный «назад», как жест свайпа от края.
7. **Вибрация уведомлений вдвое дольше.** Протокол не имеет поля длительности (рисунок вибрации зашит в прошивке часов), поэтому команда уведомления отправляется дважды с интервалом 1,2 с — часы вибрируют два цикла подряд. Действует для приложений, SMS и звонков.
8. **Убраны 32-битные нативные библиотеки** — на 64-битных устройствах они не работают и ломали установку; их роль выполняет код приложения.
9. **Диагностика для энтузиастов:** необработанные исключения пишутся в logcat с тегом `MZWatch`, при зомби-зависании процесс перезапускается.
10. **Тихий logcat.** Отладочные логи приложения и встроенных SDK (Log.d/v/i) отключены; в logcat остаются только предупреждения, ошибки и краши — включая диагностику проекта (тег `MZWatch`).

### Что по-прежнему не работает

Облачные сервисы Meizu (`meiklife.cn`) отключены компанией: вход в аккаунт Meizu, облачная синхронизация и обновление прошивки часов через приложение недоступны. На локальные функции (подключение, уведомления, будильники, поиск телефона, шагомер) это не влияет.

### Установка

**Вариант А — без компьютера (рекомендуется):**
1. Установите [Shizuku](https://shizuku.rikka.app/) (нужна беспроводная отладка в настройках разработчика).
2. Установите [InstallerX](https://github.com/RikkaApps/InstallerX/releases).
3. Откройте APK из этого пакета через InstallerX и подтвердите установку.

**Вариант Б — через ADB:** `adb install -r "Meizu_Watch_v1_skia81.apk"`

**Важно о подписи.** Сборка подписана встроенным debug-ключом uber-apk-signer (действует до 2044 года; у всех, кто собирает этим инструментом, ключ одинаковый). Это значит:
- обновление ставится **поверх** стоковой 1.6.3 или предыдущих сборок этого проекта — сопряжение, настройки и данные сохраняются;
- если собрать свою версию **другим** ключом, перед установкой придётся удалить старое приложение и сопрягать часы заново.

### Сборка из исходников

1. Установите JDK 8+.
2. Скачайте [APKEditor](https://github.com/Re/APKEditor/releases) и [uber-apk-signer](https://github.com/patrickfav/uber-apk-signer/releases), положите JAR-файлы в каталог `build/tools/`.
3. Запустите `build/build.sh` (Linux/macOS/WSL). Готовый APK появится рядом — подписанный и выровненный.

Каталог `source/` уже содержит все правки. Скрипты в `patches/` — это автоматизация, которой они вносились (идемпотентны: на уже пропатченном дереве ничего не меняют); пригодятся, если захотите воспроизвести правки на другой версии стока.

### Структура пакета

```
Meizu_Watch_v1_skia81/
├── Meizu_Watch_v1_skia81.apk     — готовое приложение
├── README.md                      — этот файл
├── build/
│   └── build.sh                   — сборка APK из source/
├── patches/                       — Python-скрипты правок (v8…v12 + logtrim)
└── source/                        — полный проект сборки APK
    ├── AndroidManifest.xml        — манифест (targetSdk 30, QUERY_ALL_PACKAGES, ресивер AlarmStop)
    ├── smali/                     — код приложения (Dalvik-байткод, 4298 файлов)
    ├── resources/                 — декодированные ресурсы
    ├── root/                      — assets и прочие файлы из корня APK
    └── signatures/                — метаданные подписи APKEditor
```

### Дисклеймер

Проект некоммерческий и сделан сообществом для владельцев часов Meizu Watch. Все права на оригинальное приложение принадлежат Meizu Technology Co., Ltd. Поставляется «как есть», без каких-либо гарантий.

---



