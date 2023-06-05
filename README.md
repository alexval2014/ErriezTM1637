# Оптимизированная библиотека TM1637 для Arduino
[![Build Status](https://travis-ci.org/Erriez/ErriezTM1637.svg?branch=master)](https://travis-ci.org/Erriez/ErriezTM1637)

Это двухконтактная библиотека последовательных микросхем TM1637 для Arduino, оптимизированная по размеру и скорости. Она поддерживает комбинированный контроллер драйвера светодиода и интерфейс сканирования клавиш для обнаружения одного нажатия клавиши.

![TM1637 chip](https://raw.githubusercontent.com/Erriez/ErriezTM1637/master/extras/TM1637_pins.jpg)


## Особенности чипа

- КМОП (CMOS) технология
- Режим отображения (8 сегментов × 6 цифр), для светодиодов с общим анодом
- Сканирование клавиш (8 x 2 бита), улучшенная схема распознавания кнопок с защитой от помех
- Схема регулировки яркости (8 ступеней яркости)
- Двухпроводной последовательный интерфейс (CLK, DIO)
- Режим колебаний: встроенный RC-генератор
- Встроенная схема сброса при включении питания
- Встроенная схема автоматического отключения
- Корпус: DIP20/SOP20


## Аппаратное обеспечение

Подключите питание и 2 вывода данных к цифровым выводам платы Arduino:
* VDD (питание 3.3V - 5V)
* GND (общий провод)
* CLK (тактовые импульсы)
* DIO (двунаправленный ввод/вывод данных)

Следующие контакты TM1637 должны быть подключены к светодиодам и кнопкам в матрице:  
* K1~K2 (ввод данных сканирования клавиш для считывания одного нажатия клавиш друг за другом)
* SEG/GRID (вывод для светодиодной матрицы)


## Контакты

| Вывод  | TM1637 | Arduino UNO / Nano / Micro / Pro Micro / Leonardo / Mega2560 | WeMos D1 & R2 / Node MCU | WeMos LOLIN32 |
| :----: | :----: | :----------------------------------------------------------: | :----------------------: | :-----------: |
|   1    |  VCC   |                         5V (или 3.3V)                        |           3V3            |      3V3      |
|   2    |  GND   |                             GND                              |           GND            |      GND      |
|   3    |  CLK   |                       2 (DIGITAL pin)                        |            D2            |       0       |
|   4    |  DIO   |                       3 (DIGITAL pin)                        |            D3            |       4       |

* Проверьте максимальный ток регулятора/диода, чтобы предотвратить перегорание при использовании большого количества светодиодов. Некоторые платы могут обеспечивать только 100 мА, другие - максимум 800 мА.


## Двухпроводной последовательный интерфейс

TM1637 обменивается данными с последовательным микроконтроллером с помощью двух проводов:

* DIO (двунаправленный вывод ввода/вывода)
* SCL (вывод синхронизации)

Примечание: последовательный интерфейс несовместим с I2C или TWI, поскольку не используется адрес устройства с битом чтения / записи.


## Пример

Arduino IDE | Examples | Erriez TM1637 button and LED driver:

[ErriezTM1637](https://github.com/Erriez/ErriezTM1637/blob/master/examples/ErriezTM1637/ErriezTM1637.ino)


## Документация

- [Doxygen online HTML](https://Erriez.github.io/ErriezTM1637)
- [Doxygen PDF](https://github.com/Erriez/ErriezTM1637/raw/master/ErriezTM1637.pdf)
- [TM1637 Datasheet (Chinese)](https://github.com/Erriez/ErriezTM1637/blob/master/extras/TM1637_datasheet_chinese.pdf)


## Использование

**Инициализация**

```c++
// Подключим библиотеку TM1637
#include <ErriezTM1637.h>
  
// Подключить контакты дисплея к цифровым контактам Arduino
#define TM1637_CLK_PIN   2
#define TM1637_DIO_PIN   3

// Создадим объект tm1637
TM1637 tm1637(TM1637_CLK_PIN, TM1637_DIO_PIN);

void setup()
{
    tm1637.begin(); // Инициализируем TM1637
}
```

**Дисплей вкл/выкл**

```c++
// Выключим дисплей
tm1637.displayOff();
  
// Включим дисплей
tm1637.displayOn();
```

**Выключите все светодиоды**

```c++
// Выключите все светодиоды
tm1637.clear();
```

**Опрос клавиатуры**

```c++
// Получаем 8-бит состояния кнопок
uint8_t keys = tm1637.getKeys();
```

**Записать байт в регистр дисплея**

```c++
// Отображает цифры и символы. Просто пример, который зависит от аппаратного обеспечения:
// Запишите сегментные светодиоды в первые регистры отображения 0x00 ..0x0F со значением 0x00 ..0xff 
tm1637.writeData(0x01, 0x01);
```

**Запись буфера в регистры отображения**

```c++
// Создать буфер с состоянием светодиодов
uint8_t buf[] = { 0b10000110, 0b00111111, 0b00111111, 0b00111111, 0b00111111, 0b00111111};

// Записать буфер в TM1637
tm1637.writeData(0x00, buf, sizeof(buf));
```

## Оптимизированное время

Библиотека использует оптимизированное управление выводами для контролеров AVR. Другие контроллеры используют функции управления выводами digitalRead() и digitalWrite() по умолчанию.

Вывод эталонного примера: [Benchmark](https://github.com/Erriez/ErriezTM1637/blob/master/examples/Benchmark/Benchmark.ino):

| Board                |  CLK   | Read keys | Write Byte | Write 16 Bytes buffer | Clear display |
| -------------------- | :----: | :-------: | :--------: | :-------------------: | :-----------: |
| Pro Mini 8MHz        | 84kHz  |   352us   |   344us    |        1080us         |    1072us     |
| UNO 16MHz            | 170kHz |   156us   |   152us    |         496us         |     480us     |
| WeMos D1 & R2 80MHz  | 205kHz |   261us   |   137us    |         396us         |     396us     |
| WeMos D1 & R2 160MHz | 300kHz |   233us   |    96us    |         275us         |     271us     |

#### Arduino Pro-Mini 8MHz

![TM1637 Arduino Pro-Mini 8MHz timing](https://raw.githubusercontent.com/Erriez/ErriezTM1637/master/extras/TM1637_timing_ProMini_8MHz.png)

#### Arduino UNO 16MHz

![TM1637 Arduino UNO 16MHz timing](https://raw.githubusercontent.com/Erriez/ErriezTM1637/master/extras/TM1637_timing_Arduino_UNO_16MHz.png)

#### WeMos D1 & R2 80MHz

![TM1637 WeMos D1 & R2 40MHz timing](https://raw.githubusercontent.com/Erriez/ErriezTM1637/master/extras/TM1637_timing_WeMos_D1_R2_80MHz.png)

#### WeMos D1 & R2 160MHz

![TM1637 WeMos D1 & R2 160MHz timing](https://raw.githubusercontent.com/Erriez/ErriezTM1637/master/extras/TM1637_timing_WeMos_D1_R2_160MHz.png)

## Зависимости библиотеки

- В примере [Benchmark](https://github.com/Erriez/ErriezTM1637/blob/master/examples/ErriezTM1637Benchmark/ErriezTM1637Benchmark.ino) используется [Erriez Timestamp](https://github.com/Erriez/ErriezTimestamp) библиотека временных меток.


## Установка библиотеки

Пожалуйста, обратитесь к [Wiki](https://github.com/Erriez/ErriezArduinoLibrariesAndSketches/wiki) странице.
