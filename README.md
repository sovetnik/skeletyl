# skeletyl (ZMK user config)

Конфигурация ZMK для сплит-клавиатуры **Skeletyl** на `nRF52840`:
- `skeletyl_left`
- `skeletyl_right`

Репозиторий содержит board definition, keymap и CI-сборку прошивок в `.uf2`.

## Что в репозитории

- `boards/arm/skeletyl/` — board-файлы, devicetree, defconfig, keymap.
- `config/west.yml` — манифест ZMK (пин на `zmk v0.3.0`).
- `build.yaml` — матрица сборки GitHub Actions.
- `.github/workflows/` — workflow сборки артефактов.
- `zephyr/module.yml` — board root для Zephyr.

## CI-сборка

При `push` / `pull_request` / `workflow_dispatch` запускается workflow:
- собирает `skeletyl_left` (с `ZMK Studio` snippet),
- собирает `skeletyl_right`,
- публикует артефакты прошивки (`.uf2`).

## Локальная сборка

Требуется `west` и окружение ZMK/Zephyr.

```bash
west init -l config
west update
west zephyr-export
west build -s zmk/app -b skeletyl_left -- -DZMK_CONFIG=$PWD/config
west build -s zmk/app -b skeletyl_right -- -DZMK_CONFIG=$PWD/config
```

Готовые файлы обычно появляются как:
- `build/zephyr/zmk.uf2`

## Прошивка

1. Переведите контроллер половины клавиатуры в bootloader.
2. Скопируйте соответствующий `.uf2`:
   - для левой половины — сборка `skeletyl_left`,
   - для правой — сборка `skeletyl_right`.
3. Повторите для второй половины.

## Настройка раскладки

- Основная раскладка: `boards/arm/skeletyl/skeletyl.keymap`.
- Если нужны разные раскладки по половинам, доступны:
  - `boards/arm/skeletyl/skeletyl_left.keymap`
  - `boards/arm/skeletyl/skeletyl_right.dts` / `skeletyl_left.dts` для аппаратных отличий.

## Отличия от стандартной wellum

Текущая раскладка в этом репозитории отклоняется от базовой wellum-конфигурации:
- отдельный recovery-блок в `COMMAND` слое (`sys_reset`, `bootloader`, `BT clear`, `USB output`);
- зеркальный recovery/RGB слой `MIRROR` (активация через `SYM + ALT`);
- матрица управления underglow (hue/saturation/speed/brightness/effect);
- `NAVIGATION` со стрелками на `H/J/K/L`;
- thumb mod-tap в `DEFAULT` слое:
  - левый thumb: `Shift` hold / `Space` tap;
  - правый thumb: `Shift` hold / `Delete` tap;
- Elixir-макросы в `ALT_CHARS` (`|>`, `%{`, `<-`, `->`, `<=`, `=>`, `insl`).

## Зависимость от Universal Layout

Для корректной работы символов и Elixir-макросов конфигурация предполагает использование `Universal Layout Ortho` в macOS.

Важно:
- если включить другую системную раскладку (US/Russian/другой custom layout), символы в макросах могут отличаться;
- часть макросов завязана на фактическое сопоставление символов в `Universal Layout`, а не только на стандартные HID-коды.

## Примечания

- В `config/info.json` указан референс на железо: <https://github.com/bastardkb/Skeletyl>.
- В репозитории есть `voltage.patch` как отдельный патч-файл для ручного применения при необходимости.
