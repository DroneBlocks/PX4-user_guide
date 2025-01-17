# Налаштування польотного контролера Porting Guide

Ця тема призначена для розробників, які хочуть портувати PX4 для роботи з _новим_ апаратним засобом керування польотом.

## Архітектура системи PX4

PX4 складається з двох основних рівнів: шару [підтримки плати та проміжного програмного забезпечення](../middleware/index.md) поверх операційної системи хоста (NuttX, Linux або будь-якої іншої платформи POSIX, такої як Mac OS), та додатків (Flight Stack у [src/modules](https://github.com/PX4/PX4-Autopilot/tree/main/src/modules)\). Будь ласка, зверніться до [Огляду архітектури PX4](../concept/architecture.md) для отримання додаткової інформації.

Цей посібник спрямований лише на операційну систему хоста та проміжне програмне забезпечення, оскільки програми/стек польоту будуть працювати на будь-якій цільовій платі.

## Flight Controller Configuration File Layout

Board startup and configuration files are located under [/boards](https://github.com/PX4/PX4-Autopilot/tree/main/boards/) in each board's vendor-specific directory (i.e. **boards/_VENDOR_/_MODEL_/**).

For example, for FMUv5:

- (All) Board-specific files: [/boards/px4/fmu-v5](https://github.com/PX4/PX4-Autopilot/tree/main/boards/px4/fmu-v5).<!-- NEED px4_version -->
- Build configuration: [/boards/px4/fmu-v5/default.px4board](https://github.com/PX4/PX4-Autopilot/blob/main/boards/px4/fmu-v5/default.px4board).<!-- NEED px4_version -->
- Board-specific initialisation file: [/boards/px4/fmu-v5/init/rc.board_defaults](https://github.com/PX4/PX4-Autopilot/blob/main/boards/px4/fmu-v5/init/rc.board_defaults) <!-- NEED px4_version -->
  - Файл ініціалізації, специфічний для плати, автоматично включається в сценарії запуску, якщо він знайдений у каталозі плат під шляхом **init/rc.board**.
  - Файл використовується для запуску сенсорів (та інших речей), які існують лише на конкретній платі. Це також може бути використано для встановлення параметрів за замовчуванням дошки, відображень UART та будь-яких інших виняткових випадків.
  - Для FMUv5 ви можете побачити, як запускаються всі датчики Pixhawk 4, а також встановлюється більший LOGGER_BUF.

## Конфігурація операційної системи хоста

Цей розділ описує призначення та місцезнаходження файлів конфігурації, необхідних для кожної підтримуваної операційної системи хоста, щоб перенести їх на нове апаратне засіб керування польотом.

### NuttX

Див. [Посібник з портування дошки NuttX](porting_guide_nuttx.md).

### Linux

Плати Linux не включають ОС та конфігурацію ядра. Ці дані вже надаються зображенням Linux, доступним для плати (яке повинно підтримувати інерційні сенсори з коробки).

- [boards/px4/raspberrypi/default.px4board](https://github.com/PX4/PX4-Autopilot/blob/main/boards/px4/raspberrypi/default.px4board) - RPI cross-compilation. <!-- NEED px4_version -->

## Компоненти та конфігурація проміжного програмного забезпечення

Цей розділ описує різноманітні компоненти проміжного програмного забезпечення та необхідні оновлення файлів конфігурації для перенесення їх на нове апаратне засіб керування польотом.

### QuRT / Шестикутник

- Початковий скрипт знаходиться в [posix-configs/](https://github.com/PX4/PX4-Autopilot/tree/main/posix-configs). <!-- NEED px4_version -->
- Конфігурація ОС є частиною стандартного образу Linux (TODO: Вказати місце розташування ОБРАЗУ LINUX та інструкції щодо прошивки).
- Конфігурація проміжного програмного забезпечення PX4 розташована в [src/boards](https://github.com/PX4/PX4-Autopilot/tree/main/boards). <!-- NEED px4_version --> TODO: ДОДАТИ НАЛАШТУВАННЯ АВТОБУСА

## Рекомендації з підключення RC UART

Зазвичай рекомендується підключати RC через окремі піни RX та TX до мікроконтролера. Якщо, проте, RX та TX з'єднані разом, UART повинен бути переведений в режим одножильного кабелю, щоб уникнути будь-яких конфліктів. Це робиться за допомогою конфігураційної дошки та файлів маніфесту. Один приклад - [px4fmu-v5](https://github.com/PX4/PX4-Autopilot/blob/main/boards/px4/fmu-v5/src/manifest.c). <!-- NEED px4_version -->

## Офіційно Підтримуване Обладнання

Проект PX4 підтримує та забезпечує стандартне посилання на апаратне забезпечення [FMU](../hardware/reference_design.md) та будь-які плати, які сумісні з цим стандартом. Це включає [серію Pixhawk](../flight_controller/pixhawk_series.md) (див. посібник користувача для [повного списку офіційно підтримуваного обладнання](../flight_controller/index.md)).

Кожна офіційно підтримувана дошка користується наступними перевагами:

- Порт PX4 доступний у сховищі PX4
- Автоматичні версії прошивки, які можна отримати з _QGroundControl_
- Сумісність з рештою екосистеми
- Автоматизовані перевірки через CI - безпека залишається найважливішою для цієї спільноти
- [Тестування польоту](../test_and_ci/test_flights.md)

Ми закликаємо виробників плат спрямовуватися на повну сумісність з [специфікацією FMU](https://pixhawk.org/). З повною сумісністю ви користуєтеся постійним розвитком PX4 щодня, але не маєте жодних витрат на обслуговування, які виникають у зв'язку з відхиленнями від специфікації.

:::tip
Виробники повинні ретельно розглянути витрати на обслуговування перед відхиленням від специфікації (витрати виробника пропорційні рівню відхилення).
:::

We welcome any individual or company to submit their port for inclusion in our supported hardware, provided they are willing to follow our [Code of Conduct](https://github.com/PX4/PX4-Autopilot/blob/main/CODE_OF_CONDUCT.md) and work with the Dev Team to provide a safe and fulfilling PX4 experience to their customers.

It's also important to note that the PX4 dev team has a responsibility to release safe software, and as such we require any board manufacturer to commit any resources necessary to keep their port up-to-date, and in a working state.

Якщо ви хочете, щоб ваша дошка була офіційно підтримана в PX4:

- Ваше обладнання повинно бути доступним на ринку (тобто його можна придбати будь-якому розробнику без обмежень).
- Апаратні засоби повинні бути доступні команді розробників PX4, щоб вони могли підтвердити порт (звертайтеся до [lorenz@px4.io](mailto:lorenz@px4.io) за допомогою, куди відправити апаратне забезпечення для тестування).
- Плата повинна пройти повний [набір тестів](../test_and_ci/index.md) та [польотне тестування](../test_and_ci/test_flights.md).

**Проект PX4 залишає за собою право відмовитися від прийняття нових портів (або видалити поточні порти), якщо вони не відповідають вимогам, встановленим проектом.**

Ви можете звернутися до команди основних розробників та спільноти на [офіційних каналах підтримки](../contribute/support.md).

## Пов'язана інформація

- [Драйвери пристроїв](../middleware/drivers.md) - Як підтримувати нове зовнішнє обладнання (драйвери пристроїв)
- [Збірка коду](../dev_setup/building_px4.md) - Як зібрати вихідний код та завантажити прошивку
- Підтримувані автопілоти включають:
  - [Автопілот апаратного забезпечення](../flight_controller/index.md)
  - [Список підтримуваних плат](https://github.com/PX4/PX4-Autopilot/#supported-hardware) (Github) - Плати, для яких PX4-Autopilot має конкретний код
- [Підтримувані периферійні пристрої](../peripherals/index.md)
