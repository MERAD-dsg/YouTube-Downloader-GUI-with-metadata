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
## Скачать можно из releases