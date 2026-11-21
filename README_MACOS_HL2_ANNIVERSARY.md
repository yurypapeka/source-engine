# Half-Life 2 (20th Anniversary) на macOS (Apple Silicon)

Ветка `hl2-anniversary-macos` позволяет играть с контентом Half-Life 2
**после юбилейного обновления 20th Anniversary** (ноябрь 2024), включая
Episode One, Episode Two и Lost Coast. Проверено на MacBook с M3, macOS 27.

Без этих изменений модели рисуются сплошным синим и зелёным, а текст меню
превращается в белые блоки: в юбилейной версии другие шейдеры, и старый
движок выбирает не те варианты.

## Что изменено

- **Сборка на новых macOS.** `alloca.h` подключается принудительно, иначе не собирается `ivp`.
- **Шейдеры.** Таблицы вариантов (combo) 18 шейдеров приведены к юбилейной версии (по Source SDK 2013).
- **Бикубическая фильтрация лайтмапов.** Страницы лайтмапов 1024×512, переменная `r_lightmap_bicubic` (по умолчанию 1).
- **Радиальный туман.** Поддержка ключа `fogradial` у `env_fog_controller` и `sky_camera` (используется на 20 картах).
- **Главное меню.** Строки меню больше не слипаются.

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

`hl2_complete`, `hl2mp`, `bin` и `steam_input` из Steam-версии не нужны.

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
```

В `episodic/bin` и `lostcoast/bin` из Steam лежат `client.*`, `server.*` и
`gamepadui.*` для Windows, Linux и старых 32-битных Mac. Их можно удалить:
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
```

Полезные параметры:

| Параметр | Что делает |
|---|---|
| `-fullscreen` / `-windowed` | Полный экран / окно |
| `-w 2560 -h 1664` | Разрешение |
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
