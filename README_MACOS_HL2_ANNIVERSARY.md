# Half-Life 2 (20th Anniversary) на macOS (Apple Silicon)

Ветка `hl2-anniversary-macos` позволяет играть с контентом Half-Life 2
**после юбилейного обновления 20th Anniversary** (ноябрь 2024), включая
Episode One, Episode Two и Lost Coast, а также в нынешнюю Steam-версию Portal.
Проверено на MacBook с M3, macOS 27.

Без этих изменений модели рисуются сплошным синим и зелёным, а текст меню
превращается в белые блоки: в юбилейной версии другие шейдеры, и старый
движок выбирает не те варианты.

## Что изменено

- **Сборка на новых macOS.** `alloca.h` подключается принудительно, иначе не собирается `ivp`.
- **Шейдеры.** Таблицы вариантов (combo) 18 шейдеров приведены к юбилейной версии (по Source SDK 2013).
- **Бикубическая фильтрация лайтмапов.** Страницы лайтмапов 1024×512, переменная `r_lightmap_bicubic` (по умолчанию 1).
- **Радиальный туман.** Поддержка ключа `fogradial` у `env_fog_controller` и `sky_camera` (используется на 20 картах).
- **Главное меню.** Строки меню больше не слипаются.
- **Шрифты.** Текст размечается как в Windows: Tahoma и Windows-размеры, без сдвига вверх. Длинные русские подписи на кнопках помещаются.
- **Retina.** В полноэкранном режиме доступны разрешения выше «точечного» разрешения macOS (например, 1920×1248 или 2560×1664 вместо максимума 1470×956), картинка выводится в реальных пикселях. Полноэкранный режим занимает весь экран, на MacBook с вырезом камеры — и область вокруг него, так что пропорции не искажаются; в списке разрешений только режимы с пропорциями экрана (вернуть режим через Spaces: `SDL_VIDEO_MAC_FULLSCREEN_SPACES=1 ./hl2.sh ...`).
- **Настройки мыши.** Убраны «висящие» в углу элементы джойстика: в юбилейной версии их перенесли в меню `gamepadui`.
- **Язык озвучки в настройках.** Без Steam в Options → Audio показывался язык текста. Теперь там язык из `-audiolanguage`.
- **Склеенные диалоги в локализованной озвучке.** Длинные диалоги HL2 (Кляйнер, Илай, вокзал, Цитадель) в русской и других озвучках записаны цельными файлами со своими таймингами. Раньше они включались только при неанглийском тексте, и с `-language english -audiolanguage russian` фразы шли по одной по английскому расписанию и обрывали друг друга. Теперь режим выбирается по языку озвучки.

## 1. Зависимости

```sh
xcode-select --install
brew install python sdl2-compat freetype fontconfig jpeg-turbo libpng
```

Пакеты Homebrew нужны и для запуска игры, не удаляйте их после сборки.

## 2. Сборка

```sh
git clone --recursive -b hl2-anniversary-macos https://github.com/yurypapeka/source-engine
cd source-engine

# Движок и Half-Life 2 (+ Lost Coast)
python3 waf configure -T release --prefix= --build-games=hl2
python3 waf build

# Episode One и Episode Two: отдельная сборка в своей папке
WAFLOCK=.lock-waf_episodic python3 waf configure -T release --prefix= --build-games=episodic -o build-episodic
WAFLOCK=.lock-waf_episodic python3 waf build --targets=client,server

# Portal: тоже отдельная сборка
WAFLOCK=.lock-waf_portal python3 waf configure -T release --prefix= --build-games=portal -o build-portal
WAFLOCK=.lock-waf_portal python3 waf build --targets=client,server
```

## 3. Файлы игры

Возьмите из установленной через Steam Half-Life 2 (Windows- или Linux-версии)
эти папки и сложите в одну папку игры, например `~/Desktop/Half-Life 2`:

| Папка | Что это |
|---|---|
| `hl2` | Half-Life 2 (обязательно) |
| `platform` | Общие ресурсы (обязательно) |
| `episodic` | Episode One |
| `ep2` | Episode Two |
| `lostcoast` | Lost Coast |
| `portal` | Portal (из установленного через Steam Portal, см. ниже) |

`hl2_complete`, `hl2mp`, `bin` и `steam_input` из Steam-версии не нужны.

### Portal

Portal в Steam ставится в отдельную папку, но оттуда нужна только папка
`portal`. Положите её рядом с `hl2`. Свои `hl2`, `platform`, `bin` и
`steam_input` у Portal можно не брать: его копия общих файлов HL2 совпадает
с юбилейной байт в байт, и Portal берёт их из вашей папки `hl2`, как эпизоды.

Английские реплики GLaDOS лежат в `portal_pak`, русские в
`portal_sound_vo_russian`.

Подойдёт только обычный Portal. Portal with RTX рассчитан на NVIDIA RTX
Remix, который работает лишь в Windows с видеокартой RTX.

## 4. Установка собранного

Из папки `source-engine`:

```sh
GAME=~/Desktop/"Half-Life 2"

mkdir -p "$GAME/bin" "$GAME/hl2/bin"
cp build/launcher_main/hl2_launcher "$GAME/hl2_osx"
find build -name '*.dylib' ! -name libclient.dylib ! -name libserver.dylib -exec cp {} "$GAME/bin/" \;
cp build/game/client/libclient.dylib build/game/server/libserver.dylib "$GAME/hl2/bin/"

# Lost Coast использует библиотеки Half-Life 2
[ -d "$GAME/lostcoast" ] && mkdir -p "$GAME/lostcoast/bin" && \
  cp build/game/client/libclient.dylib build/game/server/libserver.dylib "$GAME/lostcoast/bin/"

# Episode One и Two используют общие библиотеки из episodic/bin
[ -d "$GAME/episodic" ] && mkdir -p "$GAME/episodic/bin" && \
  cp build-episodic/game/client/libclient.dylib build-episodic/game/server/libserver.dylib "$GAME/episodic/bin/"

# Portal
[ -d "$GAME/portal" ] && mkdir -p "$GAME/portal/bin" && \
  cp build-portal/game/client/libclient.dylib build-portal/game/server/libserver.dylib "$GAME/portal/bin/"
```

В `episodic/bin`, `lostcoast/bin` и `portal/bin` из Steam лежат `client.*`,
`server.*` и `gamepadui.*` для других систем. Их можно удалить:
движок всё равно сначала пробует их загрузить и пишет в лог `failed to dlopen`.

### Скрипт запуска `hl2.sh`

В Linux-версии игры `hl2.sh` уже есть, он ищет программу `hl2_osx`.
Если у вас его нет (например, файлы из Windows-версии), создайте
`$GAME/hl2.sh`:

```sh
#!/bin/bash
cd "$(dirname "$0")"
export DYLD_LIBRARY_PATH="$PWD/bin:$DYLD_LIBRARY_PATH"
exec ./hl2_osx "$@"
```

и сделайте его исполняемым: `chmod +x "$GAME/hl2.sh"`.

**Запускайте игру только через `./hl2.sh`.** Он указывает игре брать
библиотеки из `bin`. Если запустить `./hl2_osx` напрямую, игра будет искать
их по путям в папке сборки (`source-engine/build/...`) и перестанет
запускаться, если эту папку удалить.

## 5. Запуск

```sh
cd ~/Desktop/"Half-Life 2"
./hl2.sh                      # Half-Life 2
./hl2.sh -game episodic       # Episode One
./hl2.sh -game ep2            # Episode Two
./hl2.sh -game lostcoast      # Lost Coast
./hl2.sh -game portal         # Portal
```

Полезные параметры:

| Параметр | Что делает |
|---|---|
| `-fullscreen` / `-windowed` | Полный экран / окно |
| `-w 2560 -h 1664` | Разрешение. В полноэкранном режиме можно выше «точечного» разрешения macOS, вплоть до реального размера экрана в пикселях (на 13" MacBook Air — 2940×1912) |
| `-noborder` | Окно без рамки |
| `-novid` | Пропустить заставку Valve |
| `-language russian` | Язык текста |
| `-audiolanguage russian` | Язык озвучки (нужны `*_sound_vo_russian*.vpk`) |
| `-console` | Открыть консоль при старте |
| `+map d1_trainstation_01` | Сразу загрузить карту |

Например:

```sh
./hl2.sh -fullscreen -w 2560 -h 1664 -novid -language russian -audiolanguage russian
```

Консольные переменные: `r_lightmap_bicubic 0/1` (бикубические лайтмапы),
`fps_max 120`, `mat_vsync 1`, `closecaption 1` (субтитры).

## Ограничения

- **Нет нового меню и объединённой игры.** Меню `gamepadui` и игра `hl2_complete`
  (все части в одном меню) не поддерживаются: их кода нет в открытом доступе.
  Используется классическое меню, а части запускаются по отдельности через `-game`.
- **Шрифт GorDIN не нужен.** Им задан только `SteamDeckLoadingText`, а этот движок его не использует.
  Сообщения `Couldn't find custom font file ... GorDIN` в логе безвредны.
- **Без Steam язык озвучки в меню не переключается.** В Options → Audio
  виден только язык из `-audiolanguage`. Другой язык задаётся этим параметром.
- **Portal 2 не поддерживается.** Он работает на другой, закрытой ветке движка
  Source, поэтому собрать его из этого кода нельзя. Официальная Mac-версия
  32-битная и на современных macOS не запускается.
