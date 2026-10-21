# sirius-schedule — Arch Linux · Hyprland · Waybar

Сегодняшнее расписание группы [Сириус колледжа](https://schedule.siriusuniversity.ru) для Arch Linux + Hyprland:

- **`sirius-waybar`** — модуль waybar: текущая (или следующая) пара и аудитория, при наведении — все пары на сегодня с временем, адресом и преподавателем;
- **`sirius-schedule`** — небольшое окно GTK4 с расписанием на сегодня.

Только стандартная библиотека Python + GTK4, без `requests`, Node и браузера. Расписание кэшируется, поэтому всё открывается мгновенно.
Протокол сайта взят из [PerovXP/pyrinium](https://github.com/PerovXP/pyrinium) и [PerovXP/sirinium](https://github.com/PerovXP/sirinium).

## Установка

```sh
git clone https://github.com/Nokzes/sirius-schedule-arch-hyprland-waybar
cd sirius-schedule-arch-hyprland-waybar
makepkg -si
```

### Без пакета

Модулю waybar нужен только `python` — достаточно скачать один скрипт:

```sh
mkdir -p ~/.local/bin
curl -o ~/.local/bin/sirius-waybar https://raw.githubusercontent.com/Nokzes/sirius-schedule-arch-hyprland-waybar/main/sirius-waybar
chmod +x ~/.local/bin/sirius-waybar
```

В конфиге waybar тогда укажите полный путь: `"exec": "~/.local/bin/sirius-waybar"`.

Окно тоже можно запускать без пакета (нужны `python-gobject` и `gtk4`):

```sh
sudo pacman -S --needed python-gobject gtk4
curl -o ~/.local/bin/sirius-schedule https://raw.githubusercontent.com/Nokzes/sirius-schedule-arch-hyprland-waybar/main/sirius-schedule
chmod +x ~/.local/bin/sirius-schedule
```

Удаление в этом случае — просто `rm ~/.local/bin/sirius-waybar ~/.local/bin/sirius-schedule`.

### Группа

По умолчанию используется группа `К0609-25/1`. Чтобы выбрать другую (как на сайте расписания):

```sh
mkdir -p ~/.config/sirius-schedule
echo -n 'К0609-25/1' > ~/.config/sirius-schedule/group
```

Или просто запустите `sirius-schedule К0609-25/1` — группа запомнится. В окне её можно поменять в поле сверху (Enter).

## Модуль waybar

1. Добавьте модуль в `~/.config/waybar/config` (содержимое [`waybar/module.jsonc`](waybar/module.jsonc)):

   ```jsonc
   "custom/schedule": {
     "exec": "sirius-waybar",
     "return-type": "json",
     "interval": 60,
     "signal": 8,
     "max-length": 60,
     "tooltip": true,
     "on-click": "pkill -RTMIN+8 waybar"
   }
   ```

2. Поставьте `"custom/schedule"` в нужный список, например:

   ```jsonc
   "modules-left": ["hyprland/workspaces", "custom/schedule"],
   ```

3. Стили — в `~/.config/waybar/style.css` (пример в [`waybar/style.css`](waybar/style.css)). Классы: `now` (идёт пара), `next`, `done`, `free`, `error`.

4. Перезапустите waybar: `pkill -SIGUSR2 waybar`.

Что показывается в панели:

| Ситуация | Текст |
|---|---|
| Идёт пара | `301 · Математика` |
| Перерыв / до первой пары | `Далее: 302 · Физика` |
| Пары закончились | `пары закончились` |
| Пар нет | `пар нет` |
| Нет сети и нет кэша | `нет связи` |

Клик по модулю — принудительное обновление. Сайт опрашивается не чаще раза в 3 часа, остальное время данные берутся из кэша `~/.cache/sirius-waybar.json`.

## Окно в Hyprland

`~/.config/hypr/hyprland.conf`:

```ini
windowrulev2 = float, class:^(ru.sirius.schedule)$
windowrulev2 = size 440 560, class:^(ru.sirius.schedule)$
windowrulev2 = center, class:^(ru.sirius.schedule)$
bind = $mainMod, S, exec, sirius-schedule
```

`Esc` или `q` закрывает окно.

## VPN

Сайт расписания не открывается через зарубежные VPN. Если у вас VPN (Throne, sing-box и т.п.), добавьте прямое подключение (direct) для домена `siriusuniversity.ru`.

## Отладка

Если пары отображаются неправильно, посмотрите сырой ответ сайта:

```sh
sirius-waybar --dump | head -50
```

## Удаление

```sh
sudo pacman -R sirius-schedule
rm -rf ~/.config/sirius-schedule ~/.cache/sirius-schedule.json ~/.cache/sirius-waybar.json
```

И уберите `custom/schedule` из конфига waybar.
