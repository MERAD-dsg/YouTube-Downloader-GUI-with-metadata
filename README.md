# Патч: добавление метаданных в YouTube Downloader GUI
### Проект: [MarkAdderly/YouTube-Downloader-GUI](https://github.com/MarkAdderly/YouTube-Downloader-GUI)

---

## Что изменено

В оригинальное приложение добавлена одна функция — **встраивание метаданных в скачанный файл**.

Новый элемент интерфейса — галочка под блоком «Настройки загрузки»:

```
☑  Добавить метаданные (обложка, название, автор)
```

Когда галочка **включена**, `yt-dlp` получает флаги:
- `--embed-thumbnail` — вшивает обложку (картинку трека/видео) прямо в файл
- `--embed-metadata` — вшивает ID3/MP4-тэги: название, автор, альбом, год и т.д.
- `--embed-chapters` — вшивает главы (для видео с таймстампами)

Для **MP3** дополнительно добавляется `--add-metadata` для совместимости с ID3v2.

Состояние галочки **сохраняется** между сессиями через `Properties.Settings`.

---

## Файлы патча

| Файл | Что изменено |
|---|---|
| `MainWindow.xaml` | Добавлен `<CheckBox x:Name="EmbedMetadataCheckBox">` в Grid.Row="6"; высота окна увеличена с 650 до 685 |
| `MainWindow.xaml.cs` | Чтение/сохранение настройки `embedMetadata`; формирование `metaArgs`; обработчик строки `[EmbedThumbnail]`/`[Metadata]` в прогрессе; поддержка `music.youtube.com` в `IsYouTubeUrl` |
| `Properties/Settings.settings` | Добавлено поле `embedMetadata` (тип `bool`, значение по умолчанию `True`) |

---

## Как применить патч

### 1. Клонировать репозиторий

```bash
git clone https://github.com/MarkAdderly/YouTube-Downloader-GUI.git
cd YouTube-Downloader-GUI
```

### 2. Заменить файлы

Скопируй из этой папки:

```
MainWindow.xaml        →  <корень проекта>/MainWindow.xaml
MainWindow.xaml.cs     →  <корень проекта>/MainWindow.xaml.cs
Settings.settings      →  <корень проекта>/Properties/Settings.settings
```

> **Важно:** после замены `Settings.settings` нужно пересобрать автогенерируемый
> `Settings.Designer.cs`. Visual Studio сделает это автоматически при открытии проекта.
> Если используешь командную строку — достаточно просто запустить сборку (`dotnet build`),
> MSBuild вызовет `SettingsSingleFileGenerator` самостоятельно.

---

## Сборка проекта

### Требования

| Инструмент | Версия | Откуда |
|---|---|---|
| .NET SDK | **8.0+** | https://dotnet.microsoft.com/download |
| Visual Studio | **2022** (v17.8+) с воркспейсом «.NET Desktop Development» | https://visualstudio.microsoft.com |

---

### Вариант A — через Visual Studio (рекомендуется)

1. Открой `YT-DLP-GUI.slnx` в Visual Studio 2022.
2. В Solution Explorer → ПКМ на `Properties/Settings.settings` → **Run Custom Tool** (пересоздаст `Settings.Designer.cs`).
3. Нажми **Build → Publish Selection**.
4. В диалоге Publish выбери **Folder**, затем настрой:
   - **Deployment mode:** `Self-contained` ✅
   - **Target runtime:** `win-x64`
   - **Produce single file:** ✅
5. Нажми **Publish**.

Результат появится в папке вида `bin\Release\net8.0-windows\win-x64\publish\`.

---

### Вариант B — командная строка (без Visual Studio)

```powershell
# В корне проекта:
dotnet publish YT-DLP-GUI.csproj `
  -c Release `
  -r win-x64 `
  --self-contained true `
  -p:PublishSingleFile=true `
  -p:IncludeNativeLibrariesForSelfExtract=true `
  -o ./publish
```

Готовый `YT-DLP-GUI.exe` появится в папке `./publish`.

---

## Структура папки с готовым приложением

Приложение ищет `yt-dlp.exe` и `ffmpeg` **рядом с собой** (в той же директории).
Итоговая папка должна выглядеть так:

```
📁 YouTubeDownloader\
├── YT-DLP-GUI.exe          ← само приложение (сборка выше)
├── yt-dlp.exe              ← https://github.com/yt-dlp/yt-dlp/releases/latest
├── ffmpeg.exe              ← https://www.gyan.dev/ffmpeg/builds/ (ffmpeg-release-essentials.zip)
├── ffprobe.exe             ← идёт в одном архиве с ffmpeg
└── AtomicParsley.exe       ← нужен для встраивания обложки в MP3
                               https://github.com/wez/atomicparsley/releases/latest
```

> `AtomicParsley` нужен только для формата MP3 (ID3 thumbnail).
> Для M4A и MP4 ffmpeg справляется сам.

### Откуда скачать утилиты

| Утилита | Ссылка |
|---|---|
| yt-dlp.exe | https://github.com/yt-dlp/yt-dlp/releases/latest → `yt-dlp.exe` |
| ffmpeg + ffprobe | https://www.gyan.dev/ffmpeg/builds/ → `ffmpeg-release-essentials.zip` → папка `bin\` |
| AtomicParsley.exe | https://github.com/wez/atomicparsley/releases/latest → `AtomicParsley-windows-x86_64.zip` |

---

## Создание установщика (опционально)

Для создания профессионального установщика `.exe` используй **Inno Setup** — бесплатный и простой инструмент.

### 1. Установи Inno Setup

Скачай с https://jrsoftware.org/isdl.php и установи.

### 2. Создай скрипт `installer.iss`

Создай файл `installer.iss` рядом с папкой `publish\`:

```iss
[Setup]
AppName=YouTube Downloader GUI
AppVersion=1.0.0
AppPublisher=MarkAdderly
DefaultDirName={autopf}\YouTubeDownloaderGUI
DefaultGroupName=YouTube Downloader GUI
OutputBaseFilename=YouTubeDownloaderGUI_Setup
Compression=lzma2
SolidCompression=yes
WizardStyle=modern
ArchitecturesInstallIn64BitMode=x64compatible

[Languages]
Name: "russian"; MessagesFile: "compiler:Languages\Russian.isl"
Name: "english"; MessagesFile: "compiler:Default.isl"

[Tasks]
Name: "desktopicon"; Description: "Создать ярлык на рабочем столе"; GroupDescription: "Дополнительно:"

[Files]
; Основное приложение
Source: "publish\YT-DLP-GUI.exe"; DestDir: "{app}"; Flags: ignoreversion

; Утилиты (должны лежать рядом с приложением)
Source: "tools\yt-dlp.exe";       DestDir: "{app}"; Flags: ignoreversion
Source: "tools\ffmpeg.exe";       DestDir: "{app}"; Flags: ignoreversion
Source: "tools\ffprobe.exe";      DestDir: "{app}"; Flags: ignoreversion
Source: "tools\AtomicParsley.exe"; DestDir: "{app}"; Flags: ignoreversion

[Icons]
Name: "{group}\YouTube Downloader GUI"; Filename: "{app}\YT-DLP-GUI.exe"
Name: "{group}\Удалить YouTube Downloader GUI"; Filename: "{uninstallexe}"
Name: "{autodesktop}\YouTube Downloader GUI"; Filename: "{app}\YT-DLP-GUI.exe"; Tasks: desktopicon

[Run]
Filename: "{app}\YT-DLP-GUI.exe"; Description: "Запустить YouTube Downloader GUI"; Flags: nowait postinstall skipifsilent
```

### 3. Подготовь папки

```
📁 (рядом с installer.iss)
├── 📁 publish\
│   └── YT-DLP-GUI.exe
├── 📁 tools\
│   ├── yt-dlp.exe
│   ├── ffmpeg.exe
│   ├── ffprobe.exe
│   └── AtomicParsley.exe
└── installer.iss
```

### 4. Скомпилируй установщик

- Открой `installer.iss` в Inno Setup Compiler
- Нажми **Build → Compile** (или `Ctrl+F9`)
- Готовый файл `YouTubeDownloaderGUI_Setup.exe` появится в папке `Output\`

---

## Итог

После всех шагов у тебя будет:

- ✅ `YouTubeDownloaderGUI_Setup.exe` — установщик с встроенными `yt-dlp`, `ffmpeg` и `AtomicParsley`
- ✅ Галочка «Добавить метаданные (обложка, название, автор)» в интерфейсе
- ✅ Поддержка `music.youtube.com` ссылок (кнопка вставки из буфера)
- ✅ Состояние галочки сохраняется между запусками
