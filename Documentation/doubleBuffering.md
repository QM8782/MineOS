
О библиотеке
======
DoubleBuffering - низкоуровневая библиотека для эффективного использования ресурсов GPU и отрисовки содержимого экрана с предельной скоростью. К примеру, с ее помощью реализован наш игровой движок с динамическим освещением сцен, а также небольшая игра на алгоритме рейкастинга, выдающие более чем достойные значения FPS:

![Imgur](http://i.imgur.com/YgL9fCo.png?1)

![Imgur](http://i.imgur.com/yHEwiNo.png?1)

Сама суть библиотеки очень проста: в оперативной памяти хранится два одномерных массива экрана, содержащих информацию о пикселях. Первый хранит то, что в данный момент отображается на экране, а второй - то, что пользователь желает отрисовать. После осуществления всех операций отрисовки пользователь вызывает метод buffer.**draw**(),  автоматически определяющий изменившиеся пиксели, группирующий их в промежуточный буфер, чтобы число GPU-операций было минимальным, а затем выводящий изменения на экран.

Цена таких космических скоростей - повышенный расход оперативной памяти. Чтобы предельно уменьшить расход памяти, мы реализовали одномерную структуру хранения данных вместо двумерной - так что в среднем библиотека "съедает" около 260 Кбайт.

Число операций же по сравнению с стандартной отрисовкой сокращается в сотни и тысячи раз. На рисунке ниже наглядно показана эффективность библиотеки:

![meow](http://i60.fastpic.ru/big/2015/1026/8a/4c72bfcbe8fbee5993bfd7a058a5f88a.png)



Установка
======

| Библиотека | Функционал |
| ------ | ------ |
| *[doubleBuffering](https://github.com/IgorTimofeev/OpenComputers/blob/master/lib/doubleBuffering.lua)* | Данная библиотека |
| *[color](https://github.com/IgorTimofeev/OpenComputers/blob/master/lib/color.lua)* | Низкоуровневая библиотека для работы с цветом, предоставляющая методы получения цветовых каналов, различные палитры и конверсию цвета в 8-битный формат |
| *[image](https://github.com/IgorTimofeev/OpenComputers/blob/master/lib/image.lua)* | Библиотека, реализующая стандарт изображений для OpenComputers и методы их обработки: транспонирование, обрезку, поворот, отражение и т.д. |
| *[OCIF](https://github.com/IgorTimofeev/OpenComputers/blob/master/lib/ImageFormatModules/OCIF.lua)* | Модуль формата изображения OCIF (OpenComputers Image Format) для библиотеки image, написанный с учетом особенностей мода и реализующий эффективное сжатие пиксельных данных |

Вы можете использовать имеющиеся выше ссылки для установки зависимостей вручную или запустить автоматический [установщик](https://pastebin.com/vTM8nbSZ), загружающий все необходимые файлы за вас:

    pastebin run vTM8nbSZ

Свойства библиотеки
======

| Тип свойства | Свойство |Описание |
| ------ | ------ | ------ |
| *int* | buffer.**width**| Текущее разрешение буфера по ширине |
| *int* | buffer.**height**| Текущее разрешение буфера по высоте |
| *table* | buffer.**currentFrame**| Таблица с пиксельными данными, содержащая то, что в данный момент отображено на экране. Она имеет структуру ```{ 0xFFFFFF, 0x000000, "Q", 0xFFFFFF, 0x000000, "W", ... }```, где первый элемент - цвет фона, второй - цвет текста, третий - символ. И так до конца размера буфера |
| *table* | buffer.**newFrame**| Таблица с пиксельными данными, содержащая то, что пользователь отрисовывает в него в данный момент. Структура аналогична предыдущей |

Методы библиотеки
======

buffer.**flush**( [width, height] )
-----------------------------------------------------------
| Тип | Аргумент | Описание |
| ------ | ------ | ------ |
| [*int* | width] | Опциональная новая ширина буфера |
| [*int* | height] |  Опциональная новая высота буфера |

Установить разрешение экранного буфера до указанного и заполнить его черными пикселями с символом пробела. Если разрешение не указано, то разрешение буфера станет эквивалентным текущему разрешению GPU.

buffer.**setResolution**( width, height )
-----------------------------------------------------------
| Тип | Аргумент | Описание |
| ------ | ------ | ------ |
| *int* | width | Ширина буфера |
| *int* | height | Высота буфера |

Установить текущее разрешение буфера и GPU равным указанному. Все пиксельные данные при этом очищаются.

buffer.**setDrawLimit**( x1, y1, x2, y2 )
-----------------------------------------------------------
| Тип | Аргумент | Описание |
| ------ | ------ | ------ |
| *int* | x1 | Координата первой точки лимита отрисовки по оси x |
| *int* | y1 | Координата первой точки лимита отрисовки по оси y |
| *int* | x2 | Координата второй точки лимита отрисовки по оси x |
| *int* | y2 | Координата второй точки лимита отрисовки по оси y |

Установить лимит отрисовки буфера до указанного. При этом любые операции, выходящие за границы лимита, будут игнорироваться. По умолчанию буфер всегда имеет лимит отрисовки в диапазонах **x ∈ [1; buffer.width]** и **y ∈ [1; buffer.height]** 

buffer.**getDrawLimit**( ): *int* x1, *int* y1, *int* x2, *int* y2
-----------------------------------------------------------
Получить текущий лимит отрисовки.

buffer.**draw**( [force] )
-----------------------------------------------------------
| Тип | Аргумент | Описание |
| ------ | ------ | ------ |
| [*boolean* | force] | Принудительная отрисовка |

Отрисовать содержимое буфера на экран в диапазоне текущего лимита отрисовки. Если имеется опциональный аргумент *force*, то содержимое буфера будет полностью отрисовано вне зависимости от изменившихся пикселей.

buffer.**set**( x, y, background, foreground, symbol )
-----------------------------------------------------------
| Тип | Аргумент | Описание |
| ------ | ------ | ------ |
| *int* | x | Координата по оси x |
| *int* | y | Координата по оси y |
| *int* | background | Цвет фона |
| *int* | foreground | Цвет символа |
| *char* | symbol | Символ |

Установить значение конкретного пикселя на экране. Полезно для мелкого и точного редактирования.

buffer.**get**( x, y ): *int* background, *int* foreground, *char* symbol
-----------------------------------------------------------
| Тип | Аргумент | Описание |
| ------ | ------ | ------ |
| *int* | x | Координата по оси x |
| *int* | y | Координата по оси y |

Получить значение конкретного пикселя на экране.

buffer.**copy**( x, y, width, height ): *table* pixelData
-----------------------------------------------------------
| Тип | Аргумент | Описание |
| ------ | ------ | ------ |
| *int* | x | Координата копируемой области по оси x |
| *int* | y | Координата копируемой области по оси y |
| *int* | width | Ширина копируемой области |
| *int* | height | Высота копируемой области |

Скопировать содержимое указанной области из буфера и выдать в виде таблицы. Впоследствии можно использовать с buffer.**paste**(...).

buffer.**paste**( x, y, pixelData )
-----------------------------------------------------------
| Тип | Аргумент | Описание |
| ------ | ------ | ------ |
| *int* | x | Координата вставки по оси x |
| *int* | y | Координата вставки по оси y |
| *table* | pixelData | Таблица со скопированной ранее областью буфера |

Вставить скопированное содержимое буфера по указанным координатам.

buffer.**square**( x, y, width, height, background, foreground, symbol, transparency )
-----------------------------------------------------------
| Тип | Аргумент | Описание |
| ------ | ------ | ------ |
| *int* | x | Координата прямоугольника по оси x |
| *int* | y | Координата прямоугольника по оси y |
| *int* | width | Ширина прямоугольника |
| *int* | height | Высота прямоугольника |
| *int* | background | Цвет фона прямоугольника |
| *int* | foreground | Цвет символов прямоугольника |
| *char* | symbol | Символ, которым будет заполнен прямоугольник |
| [*int* | transparency] | Опциональная прозрачность прямоугольника |

Заполнить прямоугольную область указанными данными. При указании прозрачности в диапазоне [0; 100] прямоугольник будет накладываться поверх существующей информации, словно прозрачное стеклышко.

buffer.**clear**( [background, foreground, symbol] )
-----------------------------------------------------------
| Тип | Аргумент | Описание |
| ------ | ------ | ------ |
| [*int* | background] | Опциональный цвет фона |
| [*int* | foreground] | Опциональный цвет символов |
| [*char* | symbol] | Опциональный символ |

Работает как buffer.**square**(...), однако применяется сразу ко всей области экрана. Если аргументов не передается, то буфер заполняется стандартным черным цветом и символом пробела. Удобно для быстрого заполнения содержимого буфера.

buffer.**text**( x, y, color, text, transparency )
-----------------------------------------------------------
| Тип | Аргумент | Описание |
| ------ | ------ | ------ |
| *int* | x | Координата текста по оси x |
| *int* | y | Координата текста  по оси y |
| *int* | foreground | Цвет текста |
| *string* | text | Текст |
| [*int* | transparency] | Опциональная прозрачность текста |

Нарисовать текст указанного цвета поверх имеющихся пикселей. Цвет фона при этом остается прежним. Можно также указать опциональную прозрачность текста текста в диапазоне [0; 100].

buffer.**image**( x, y, picture )
-----------------------------------------------------------
| Тип | Аргумент | Описание |
| ------ | ------ | ------ |
| *int* | x | Координата изображения по оси x |
| *int* | y | Координата изображения  по оси y |
| *table* | picture | Загруженное изображение |

Нарисовать загруженное через image.**load**(*string* path) изображение. Альфа-канал изображения также поддерживается.

Полупиксельные методы
======

Все полупиксельные методы позволяют избежать эффекта удвоения высоты пикселя консольной графики, используя специальные символы наподобие "▄". При этом передаваемые координаты по оси **Y** должны принадлежать промежутку **[0; 100]**. 


buffer.**semiPixelSet**( x, y, color )
-----------------------------------------------------------
| Тип | Аргумент | Описание |
| ------ | ------ | ------ |
| *int* | x1 | Координата пикселя по оси x |
| *int* | y1 | Координата пикселя по оси y |
| *int* | color | Цвет пикселя |

Установка пиксельного значения в указанной точке.

buffer.**semiPixelSquare**( x, y, width, height, color )
-----------------------------------------------------------
| Тип | Аргумент | Описание |
| ------ | ------ | ------ |
| *int* | x | Координата прямоугольника по оси x |
| *int* | y | Координата прямоугольника по оси y |
| *int* | width | Ширина прямоугольника |
| *int* | height | Высота прямоугольника |
| *int* | color | Цвет прямоугольника |

Отрисовка прямоугольника с указанными параметрами.

buffer.**semiPixelLine**( x1, y1, x2, y2, color )
-----------------------------------------------------------
| Тип | Аргумент | Описание |
| ------ | ------ | ------ |
| *int* | x1 | Координата первой точки линии по оси x |
| *int* | y1 | Координата первой точки линии по оси y |
| *int* | x2 | Координата второй точки линии по оси x |
| *int* | y2 | Координата второй точки линии по оси y |
| *int* | color | Цвет линии |

Растеризация отрезка указанного цвета

buffer.**semiPixelCircle**( xCenter, yCenter, radius, color )
-----------------------------------------------------------
| Тип | Аргумент | Описание |
| ------ | ------ | ------ |
| *int* | xCenter | Координата центральной точки окружности по оси x |
| *int* | yCenter | Координата центральной точки окружности по оси y |
| *int* | radius | Радиус окружности |
| *int* | color | Цвет окружности |

Растеризация окружности указанного цвета

Практический пример #1
======

```lua
-- Подключаем библиотеку
local buffer = require("doubleBuffering")

-- Рисуем 10 квадратиков, заполненных произвольным цветом
local x, y, xStep, yStep = 2, 2, 4, 2
for i = 1, 10 do
	buffer.square(x, y, 6, 3, math.random(0x0, 0xFFFFFF), 0x0, " ")
	x, y = x + xStep, y + yStep
end

-- Рисуем черную окружность
buffer.semiPixelCircle(22, 22, 10, 0x0)
-- Рисуем белую линию
buffer.semiPixelLine(2, 36, 35, 3, 0xFFFFFF)

-- Выводим содержимое буфера на экран
buffer.draw()
```

Результат: 

![Imgur](http://i.imgur.com/4jxCxXG.png)