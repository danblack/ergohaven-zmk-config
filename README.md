# ergohaven-zmk-config

Личный ZMK-конфиг для клавиатур [Ergohaven](https://github.com/ergohaven).
Прошивки собираются в GitHub Actions, готовые `.uf2` лежат в артефактах сборки.

## Сборка

Сборкой управляет `build.yaml` (список целей) + workflow
`.github/workflows/build.yml`, который вызывает общий
`ergohaven/ergohaven-zmk` пайплайн. Чтобы добавить/включить цель — раскомментируй
нужный блок `board/shield` в `build.yaml`. Локального тулчейна нет: проверка =
успешная сборка в Actions после пуша.

Активные на данный момент цели — клавиатура **op36** (`op36_left`, `op36_right`,
`op36_qube`, `op36_left_qube`). Указательного устройства у op36 нет —
`CONFIG_ZMK_POINTING=y` нужен для эмуляции мыши с клавиш.

## Структура

| Файл | Назначение |
|---|---|
| `config/<board>.keymap` | раскладки и behaviors конкретной клавиатуры |
| `config/<board>.conf` | Kconfig-опции прошивки |
| `config/<board>.json` | описание физической раскладки (для ZMK Studio / редактора) |
| `config/keys_ru.h` | хелперы для русских раскладок (`RU_*`) |
| `config/*_qube.conf` | оверлеи для qube-донгла |

## op36: раскладка

Слои `op36.keymap` (индекс → имя):

| # | Слой | # | Слой |
|---|---|---|---|
| 0 | ENG | 9 | NUM_RU |
| 1 | ENG_UP | 10 | FUN |
| 2 | RUS | 11 | APP |
| 3 | NAV | 12 | IDE |
| 4 | LSM_EN | 13 | **MUS** (мышь) |
| 5 | LSM_RU | 14 | KAN |
| 6 | RSM_EN | 15 | SETUP |
| 7 | RSM_RU | 16 | SMUS (медленная мышь) |
| 8 | NUM_EN | 17 | SHOTCUTS |

### Назначение слоёв

- **ENG (0)** — базовая английская раскладка (QWERTY).
- **ENG_UP (1)** — английская с зафиксированным Shift (заглавные) + клавиши
  перехода на русский.
- **RUS (2)** — русская раскладка (ЙЦУКЕН на QWERTY-позициях, через системный
  layout).
- **NAV (3)** — навигация: стрелки, Home/End/PgUp/PgDn, цифры в верхнем ряду,
  буфер обмена; отсюда вход на мышь и приложения.
- **LSM_EN/LSM_RU (4/5)** — левый символьный слой (Left Sym), RU-вариант шлёт
  символы через переключение раскладки.
- **RSM_EN/RSM_RU (6/7)** — правый символьный слой (Right Sym).
- **NUM_EN/NUM_RU (8/9)** — цифровой слой и мод-клавиши.
- **FUN (10)** — функциональные клавиши F1–F12 + моды.
- **APP (11)** — запуск/переключение приложений (глобальные хоткеи).
- **IDE (12)** — хоткеи IDE. *Не привязан биндингами — только через ZMK Studio.*
- **MUS (13)** — мышь (движение/клики/скролл с клавиш).
- **KAN (14)** — раскладка для режима Kanata. *Не привязан — только ZMK Studio.*
- **SETUP (15)** — Bluetooth/USB, громкость, медиа, bootloader/studio-unlock.
- **SMUS (16)** — «медленная» (точная) мышь.
- **SHOTCUTS (17)** — слой с home-row-mods, поднимается макросами мод-шорткатов
  (`scl`/`lscl`/`rscl`) с символьных слоёв.

### Карта переходов

Большие пальцы базовых слоёв (ENG/ENG_UP/RUS), слева направо:

```
[NUM]   [NAV / Space]   [RSM(GUI)]  ||  [LSM(ALT)]  [NUM(SHIFT)]  [NAV]
&mo 8/9  &lt 3 SPACE     &mosk 6/7       &mosk 4/5    &mosk 8/9     &mo 3
```

(`&mosk N MOD` = удержание → слой N, тап → sticky-модификатор; `&lt 3 SPACE` =
удержание → NAV, тап → пробел.)

Переключение языка:

- ENG → ENG_UP: `&TO_EN_UP` (на слое NAV) · ENG_UP → RUS: `&TO_RU` · любой → ENG: `&TO_EN`

Переходы вглубь:

```
NAV (3) ──&tomo 13 13──> MUS (13)      (тап = полное &to, удержание = momentary &mo)
NAV (3) ──&mo 11───────> APP (11)
LSM (4/5) ──&mo 10─────> FUN (10)
LSM (4/5) ──&to 15─────> SETUP (15)
RSM (6/7) ──&mo 11─────> APP (11)
NUM (8/9) ──&mo 10─────> FUN (10)
MUS (13) ──&to/&mo L_SMUS──> SMUS (16) · MUS ──&TO_EN──> ENG
SMUS (16) ──&to 13─────> MUS (13)
символьные слои ──scl/lscl/rscl──> SHOTCUTS (17)   (momentary)
```

### Раскладки слоёв (ASCII)

Источник истины — `config/op36.keymap`; ниже упрощённые схемы ключевых слоёв
(op36 = 3×5 на половину + 3 больших пальца). Стрелки `↑↓←→` — движение мыши.

**ENG (0)**

```
Q  W  E  R  T  |  Y  U  I  O  P
A  S  D  F  G  |  H  J  K  L  ;
Z  X  C  V  B  |  N  M  ,  .  /
      NUM  NAV/Spc  GUI | ALT  SHFT  NAV
```

**MUS (13) — мышь**

```
TO_EN  cut  copy  paste  paste+  |  ·       →SMUS   ↑       ·      scrl↑
LCTRL  LSFT LALT  LGUI   undo    |  BSPC    ←       ↓       →      scrl↓
→SMUS  ·    selА  LCTRL  →SMUS   |  ·       ·       ESC     ·      ·
              ·  LClick  RClick  | →SMUS  ENTER  ·
```

**SMUS (16) — медленная/точная мышь**

```
TO_EN  cut  copy  paste  paste+ |  ·       ·       ↑(сл.)  →MUS    scrl↑
LCTRL  LSFT LALT  LGUI   undo   |  BSPC    ←(сл.)  ↓(сл.)  →(сл.)  scrl↓
·      ·    selА  LCTRL  ·      |  ·       ·       ESC     ·       ·
              ·  LClick  RClick |  ·     ENTER  ·
```

### Мышиный слой

У op36 нет трекбола — движение/скролл/клики эмулируются клавишами (`&mmv`,
`&msc`, `&mkp`) на слоях `MUS` (13) и `SMUS` (16). Чтобы отправить событие
мыши/скролла, соответствующий слой обязан быть активен.

Вход на мышиный слой со слоя **NAV** — клавиша `&tomo 13 13` (двухрежимная):

- **короткий тап** → `&to 13`: полное переключение на `MUS`, слой остаётся
  активным (возврат — `&TO_EN` внутри `MUS`);
- **удержание** → `&mo 13`: momentary-режим — пока клавиша зажата, мышь/скролл
  работают, при отпускании возврат на исходный слой. Удобно «быстро
  проскроллить», не переключаясь в мышиный слой полностью.

Поведение `tomo` — это `zmk,behavior-hold-tap` с `bindings = <&mo>, <&to>` и
`flavor = "hold-preferred"` (см. блок `behaviors` в `config/op36.keymap`).
