# Как добавить расписание в свой waybar

Пошаговая инструкция: от установки скрипта до готового модуля в панели. Займёт 5–10 минут.

В итоге в панели будет следующая пара:

```
К_2 · 15:05 · Лек. Приёмопередающие устройства
```

а при наведении — все пары на сегодня:

```
Среда, 21 октября
К0609-25/1 · 4 пары

●  11:55–13:15  3 пара · сейчас
   Основы проектирования баз данных
   Семинар · К_4 · Бахтин А. А.

○  15:05–16:25  5 пара · через 2 ч 53 мин
   Приемопередающие устройства, линейные сооружения связи и источники электропитания
   Лекция · К_2 · Здоровцов А. Д.
```

---

## Шаг 0. Что понадобится

- waybar (`sudo pacman -S waybar`);
- python (на Arch почти всегда уже установлен: `python --version`);
- шрифт с кириллицей в waybar — подойдёт любой обычный (JetBrains Mono, Noto Sans и т.п.).

Больше ничего не нужно: модуль использует только стандартную библиотеку Python.

## Шаг 1. Установите скрипт

**Вариант А — пакетом** (ставит и модуль, и окно с расписанием):

```sh
git clone https://github.com/Nokzes/sirius-schedule-arch-hyprland-waybar
cd sirius-schedule-arch-hyprland-waybar
makepkg -si
```

Скрипт окажется в `/usr/bin/sirius-waybar`, в конфиге waybar можно писать просто `sirius-waybar`.

**Вариант Б — одним файлом**, без пакета:

```sh
mkdir -p ~/.local/bin
curl -o ~/.local/bin/sirius-waybar https://raw.githubusercontent.com/Nokzes/sirius-schedule-arch-hyprland-waybar/main/sirius-waybar
chmod +x ~/.local/bin/sirius-waybar
```

В этом случае в конфиге waybar указывайте полный путь: `~/.local/bin/sirius-waybar`.

## Шаг 2. Укажите свою группу

По умолчанию используется `К0609-25/1`. Если у вас другая — запишите её точно так, как она называется на [сайте расписания](https://schedule.siriusuniversity.ru):

```sh
mkdir -p ~/.config/sirius-schedule
echo -n 'К0609-25/1' > ~/.config/sirius-schedule/group
```

## Шаг 3. Проверьте скрипт в терминале

Прежде чем трогать waybar, убедитесь, что скрипт работает:

```sh
sirius-waybar            # или ~/.local/bin/sirius-waybar
```

Должна вывестись одна строка JSON:

```json
{"text": "К_2 · 15:05 · Лек. Приёмопередающие устройства", "tooltip": "...", "class": "next"}
```

Если в `text` написано `нет связи` — см. раздел [«Если что-то не работает»](#если-что-то-не-работает).

## Шаг 4. Найдите свой конфиг waybar

Конфиг лежит в `~/.config/waybar/`. Файл называется `config` или `config.jsonc`:

```sh
ls ~/.config/waybar/
```

Если есть **оба** файла, узнать, какой из них реально используется, можно так — запустите waybar из терминала и посмотрите первые строки:

```sh
pkill waybar; waybar 2>&1 | grep -i "config"
```

Там будет строка вида `Using configuration file /home/user/.config/waybar/config`. После проверки нажмите `Ctrl+C` и запустите waybar как обычно (`waybar & disown` или перезайдите в Hyprland).

> Некоторые готовые конфиги (например, ML4W) выносят описания модулей в отдельный файл и подключают его строкой `"include": ["~/.config/waybar/modules.json"]`. Тогда описание модуля (шаг 6) можно положить в `modules.json`, а в список (шаг 7) добавить в основной `config`.

## Шаг 5. Сделайте резервную копию

```sh
cp ~/.config/waybar/config ~/.config/waybar/config.bak
cp ~/.config/waybar/style.css ~/.config/waybar/style.css.bak
```

(Замените `config` на `config.jsonc`, если у вас он.) Если что-то сломается — просто верните файлы обратно.

## Шаг 6. Добавьте описание модуля

Откройте конфиг и добавьте блок `custom/schedule` **внутрь главных фигурных скобок** — рядом с описаниями других модулей (например, перед `"clock": {`):

```jsonc
"custom/schedule": {
  "exec": "sirius-waybar",
  "return-type": "json",
  "interval": 60,
  "signal": 8,
  "max-length": 60,
  "tooltip": true,
  "on-click": "pkill -RTMIN+8 waybar"
},
```

Важно:

- **запятая** после закрывающей `}` нужна, если дальше идёт ещё один модуль; если блок последний в файле — запятую уберите;
- если ставили скрипт без пакета, замените `"exec": "sirius-waybar"` на `"exec": "~/.local/bin/sirius-waybar"`.

Что значат параметры:

| Параметр | Зачем |
|---|---|
| `exec` | какую программу запускать |
| `return-type: json` | скрипт отдаёт текст, подсказку и класс для CSS |
| `interval: 60` | обновлять раз в минуту (сайт при этом опрашивается не чаще раза в 3 часа — остальное берётся из кэша) |
| `signal: 8` | позволяет обновить модуль командой `pkill -RTMIN+8 waybar` |
| `max-length: 60` | обрезать слишком длинный текст в панели |
| `on-click` | по клику — мгновенное обновление |

## Шаг 7. Поставьте модуль в панель

В том же файле найдите списки `modules-left`, `modules-center` или `modules-right` и добавьте `"custom/schedule"` туда, где хотите видеть расписание. Порядок в списке = порядок слева направо.

Слева, сразу после рабочих столов:

```jsonc
"modules-left": [
  "hyprland/workspaces",
  "custom/schedule"
],
```

По центру, перед заголовком окна:

```jsonc
"modules-center": [
  "custom/schedule",
  "hyprland/window"
],
```

Справа, перед раскладкой клавиатуры:

```jsonc
"modules-right": [
  "custom/schedule",
  "hyprland/language",
  "clock"
],
```

## Шаг 8. Добавьте стили

Допишите в конец `~/.config/waybar/style.css`:

```css
#custom-schedule {
  padding: 0 10px;
  margin-left: 40px;   /* отступ слева; уберите или поменяйте по вкусу */
  color: #b7bdf8;
}

#custom-schedule.next {
  color: #a6da95;
}

#custom-schedule.error {
  color: #ed8796;
}
```

Если в вашей теме определены цвета Catppuccin (`@define-color green ...`, например через `@import "macchiato.css";`), можно писать `@green`, `@lavender`, `@red` вместо кодов.

Классы, которые выставляет модуль:

| Класс | Когда |
|---|---|
| `next` | сегодня ещё есть пара |
| `done` | пары на сегодня закончились |
| `free` | сегодня пар нет |
| `error` | нет связи с сайтом и нет кэша, или не указана группа |

**Необязательно:** красивое окошко подсказки. Учтите, что это стиль для подсказок **всех** модулей waybar:

```css
tooltip {
  background-color: #1e2030;
  border: 1px solid #494d64;
  border-radius: 12px;
}

tooltip label {
  color: #cad3f5;
  font-size: 15px;
  padding: 6px 8px;
}
```

Готовый файл со стилями: [`waybar/style.css`](../waybar/style.css).

## Шаг 9. Перезагрузите waybar

```sh
pkill -SIGUSR2 waybar
```

Waybar перечитает конфиг и стили без перезапуска. Модуль появится в панели; наведите на него, чтобы увидеть все пары.

---

## Настройка

- **Свои сокращения предметов.** Длинные названия в панели сокращаются автоматически. Чтобы задать своё, создайте `~/.config/sirius-schedule/short` и пишите по строке:

  ```
  Основы проектирования баз данных = Базы данных
  Физическая культура = Физра
  ```

- **Показывать больше текста** — увеличьте `max-length` (или удалите его совсем).
- **Сдвинуть модуль** — меняйте `margin-left` / `margin-right` в `#custom-schedule`.
- **Сигнал 8 уже занят** другим вашим модулем — поставьте любой свободный номер (например, `9`) и в `signal`, и в `on-click`: `pkill -RTMIN+9 waybar`.

## Если что-то не работает

**Модуль не появился.**
Скорее всего, ошибка в JSON (чаще всего — лишняя или пропущенная запятая). Запустите waybar из терминала и посмотрите ошибки:

```sh
pkill waybar; waybar
```

Также проверьте, что имя в списке модулей совпадает с описанием: `"custom/schedule"` в обоих местах.

**В панели `нет связи`.**
Сайт расписания не открывается через зарубежные VPN. Отключите VPN или добавьте в нём прямое подключение (direct) для домена `siriusuniversity.ru`, затем кликните по модулю.

**В панели `пар нет`, хотя пары есть.**
Проверьте название группы в `~/.config/sirius-schedule/group` — оно должно совпадать с сайтом. Затем удалите кэш и обновите:

```sh
rm ~/.cache/sirius-waybar.json; pkill -RTMIN+8 waybar
```

**Пары отображаются неправильно.**
Посмотрите сырой ответ сайта и приложите его к issue:

```sh
sirius-waybar --dump | head -50
```

**В подсказке видны теги вроде `<span>`.**
Значит, waybar не разобрал разметку — откройте issue и приложите вывод `sirius-waybar`.

## Как убрать

1. Удалите `"custom/schedule"` из списка модулей и блок `"custom/schedule": {...}` из конфига.
2. Удалите стили `#custom-schedule` (и `tooltip`, если добавляли) из `style.css`.
3. Удалите скрипт и данные:

   ```sh
   sudo pacman -R sirius-schedule        # если ставили пакетом
   rm ~/.local/bin/sirius-waybar          # если ставили одним файлом
   rm -rf ~/.config/sirius-schedule ~/.cache/sirius-waybar.json
   ```

4. `pkill -SIGUSR2 waybar`.
