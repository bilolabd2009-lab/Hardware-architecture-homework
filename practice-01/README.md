# Практика: BIOS/UEFI в Lenovo BIOS Simulator Center

Абдуллаев Билолиддин Низомидинович, группа 9/2-РПО-25/3

Симулятор: https://download.lenovo.com/bsco/index.html
Устройство: **ThinkPad T14s Gen 5 (21LS, 21LT)**, графический режим (Graphics Interface Mode).

Отчёт в Word: [uefi-bios.docx](uefi-bios.docx)

Верхнее меню UEFI: **Main · Config · Date/Time · Security · Startup · Restart**.
Внизу всегда подсказки клавиш: F1 — справка, **F9 — Setup Defaults**, Esc — выход, **F10 — Save and Exit**.

---

## Задача 1. Навигация по основным разделам

| Раздел | Что внутри |
|---|---|
| Main | Сведения о системе, только для чтения |
| Config | Оборудование: Network, USB, Keyboard/Mouse, Display, Power, Beep and Alarm, CPU, Thunderbolt 4, Intel AMT, Intel Standard Manageability |
| Date/Time | Системные дата и время |
| Security | Password, Fingerprint, Security Chip, UEFI BIOS Update Option, Memory Protection, Virtualization, I/O Port Access, Internal Device Access, Secure Boot, ThinkShield secure wipe, Reset system to Factory Defaults и др. |
| Startup | Порядок загрузки (Boot), Network Boot, Boot Mode, F12-меню, Boot Order Lock |
| Restart | Выход с сохранением или без, загрузка и сохранение настроек по умолчанию |

Что показывает **Main**:

| Параметр | Значение в симуляторе |
|---|---|
| UEFI BIOS Version | N46ET09W |
| UEFI BIOS Date | 2024-01-22 |
| Embedded Controller Version | N46HT10W |
| Machine Type Model | 21LS |
| System-unit serial number | PF12345 |
| System board serial number | L112345678 |
| CPU Type | Intel(R) Core(TM) CPU |
| CPU Speed | 1.X00GHz |
| Installed memory | 8192 MB |
| UUID | 79498acc-3133-11b2-a85c-123412341234 |
| UEFI Secure Boot | On |

Где находятся настройки задач 2–5: порядок загрузки — **Startup**, Secure Boot и пароль — **Security**,
сброс настроек — **Restart** (или клавиша F9).

## Задача 2. Смена порядка загрузки

Путь: **Startup → Boot**.

Исходный порядок: USB CD, USB FDD, NVMe0, USB HDD, PXE BOOT, LENOVO CLOUD, ON-PREMISE, Other CD, Other HDD.

Новый порядок: **USB HDD** (флешка) — первым, **NVMe0** — вторым. Элемент перемещается
клавишами **+ / −** (или F5/F6). Сохранение — **F10 → Yes**. После повторного входа порядок
виден там же, в Startup → Boot.

## Задача 3. Включение/выключение Secure Boot

Путь: **Security → Secure Boot → Secure Boot**. Исходное состояние — **On**, режим User Mode, ключи Standard Mode.

Переключил On → Off → On. Изменение вступает в силу только после сохранения (F10)
и **перезагрузки**: Secure Boot проверяется при старте, до загрузки ОС.

**Зачем нужен Secure Boot:** при включении UEFI загружает только загрузчики, подписанные
доверенными ключами (Microsoft, производитель). Это защищает от буткитов и руткитов, которые
подменяют загрузчик до старта ОС. Отключать его приходится для неподписанных систем
(некоторые Linux, старые LiveCD).

## Задача 4. Установка пароля супервизора

Путь: **Security → Password → Supervisor Password** (исходно Disabled).

1. Enter → ввести новый пароль `Test1234` и подтвердить. Статус стал Enabled.
2. F10, повторный вход: при входе в Setup и при попытке изменить настройку запрашивается пароль.
3. Сброс: снова Supervisor Password → ввести текущий `Test1234` → новый пароль оставить пустым
   (Enter, Enter). Статус вернулся в Disabled.

Рядом есть Power-On Password (запрос при включении), System Management Password
и NVMe1 Password (пароль на сам диск).

## Задача 5. Сброс на заводские настройки

Путь: **Restart → Load Factory Defaults** → Yes, затем **F10**. Быстрая клавиша — **F9 (Setup Defaults)**.

После сброса порядок загрузки и Secure Boot вернулись к исходным значениям.
Пароль супервизора **не сбрасывается** через Load Factory Defaults: его снимают отдельно (задача 4).

Не путать с **Security → Reset system to Factory Defaults**: этот пункт сбрасывает ещё и ключи,
пароли и данные TPM, то есть делает полный сброс устройства.

## Задача 6. Управление встроенными устройствами

Путь: **Config → USB → Always On USB** (исходно On). Переключил Off → On.

Always On USB подаёт питание на USB-порт, когда ноутбук выключен или спит
(зарядка телефона). Рядом — Charge in Battery Mode (Off).

Отдельного пункта USB BIOS Support на T14s Gen 5 нет: USB-загрузка управляется в Startup
(USB HDD/USB CD в списке Boot), а сами порты отключаются в **Security → I/O Port Access**.

## Задача 7. Режим работы SATA/NVMe

На T14s Gen 5 раздела **Config → Storage / SATA Controller Mode нет**: в ноутбуке только
NVMe-накопитель, который работает напрямую через PCIe, переключать режим нечего.

Теория по режимам:
- **AHCI** — стандартный режим, драйвер есть в любой ОС;
- **RAID / Intel RST** — объединение дисков и кэширование через драйвер Intel.

Риск: если сменить режим после установки ОС, Windows не найдёт драйвер контроллера
и упадёт в синий экран `INACCESSIBLE_BOOT_DEVICE`. Режим выбирают **до** установки ОС.

## Задача 8. Временное меню загрузки (F12)

Путь: **Startup → Boot device List F12 Option** = **On**.

При включении ноутбука нажатие **F12** открывает список устройств, из которого можно
загрузиться **один раз**. Постоянный порядок в Startup → Boot не меняется.

Рядом: **Startup → Boot Mode** (Quick / Diagnostics), Option key Display (подсказка клавиш при
старте) и **Boot Order Lock** (запрет менять порядок загрузки).

## Задача 9. Просмотр системных логов

Путь: **Main → BIOS Event log**. В симуляторе этот пункт помечен «Simulator not support»,
поэтому сам журнал и кнопку Clear Event Log открыть нельзя. На реальном ThinkPad там хранятся
события POST, ошибки и изменения настроек, а очищает их пункт Clear в этом же журнале.

## Задача 10. Полный сценарий: подготовка к переустановке ОС

1. **Restart → Load Factory Defaults** → Yes.
2. **Security → Secure Boot → Secure Boot = On**.
3. **Security → Password → Supervisor Password** = `Test1234`.
4. **Startup → Boot**: USB HDD первым.
5. **F10 → Yes**: сохранить и выйти.
6. Снова войти (с вводом пароля), проверить Secure Boot и порядок загрузки, затем снять пароль
   и вернуть NVMe0 первым, сохранить F10.

## Вывод

UEFI устроен одинаково на всех ThinkPad. В Main — информация, в Config — железо,
в Security — пароли и Secure Boot, в Startup — загрузка, в Restart — сохранение и сброс.
Главные клавиши — F9 (по умолчанию) и F10 (сохранить и выйти). Перед установкой ОС
важно проверить порядок загрузки и Secure Boot, а режим накопителя не менять после установки.
