## Использование инструкций SSE (Streaming SIMD Extensions) для ускорения вычислений в программе

### Задание
Исследовать влияние применения SSE (Streaming SIMD Extensions) инструкций на производительность программы алгоритма построения [множества Мандельброта](https://ru.wikipedia.org/wiki/Множество_Мандельброта). Ожидается, что использование SSE инструкций улучшит эффективность вычислений.

### Введение
Тесты были произведены на процессоре Intel Core i7, он поддерживает SSE-инструкции.

Инструкции SSE (Streaming SIMD Extensions) предоставляют SIMD (Single Instruction, Multiple Data) возможности для ускорения параллельной обработки значений на процессорах Intel. SSE инструкции позволяют выполнять одну операцию над несколькими данными одновременно.

Для измерения производительности используется инструкция rdtsc, которая подсчитывает количество тактов процессора, затраченных на выполнение определенной части кода. Функция вывода изображения отключается во время замеров, чтобы сравнивать именно затраты времени на вычисление множества, а не на отрисовку. Применяется функция __rdtsc() в языке программирования С++, которая предоставляет доступ к инструкции rdtsc. Она является инструкцией ассемблера, предоставляемой процессором Intel x86 для чтения значения тактовых счетчиков процессора

Визуализация множества Мандельброта с помощью SFML:

<img src = "Picture/1.png" width="500" height="250">

<img src = "Picture/2.png" width="500" height="250">

<img src = "Picture/3.png" width="500" height="250">


### Запуск программы
```
make
./app
```

### Флаги компиляции, используемые в задании
-O0 (Отсутствие оптимизаций):
Этот флаг указывает компилятору g++ не выполнять практически никаких оптимизаций при компиляции.

-O3 (Высокий уровень оптимизации):
Этот флаг указывает компилятору на максимальный уровень оптимизации кода.
Компилятор будет производить различные виды оптимизаций, включая инлайнинг функций, удаление недостижимого кода, улучшение распределения регистров, развертывание циклов и т. д.


### Первая программа
Используются обычные операции над числами с плавающей точкой (float)

```
while (x * x + y * y <= RADIUS && iteration < MAX_ITERATIONS) {
        float xtemp = x * x - y * y + x0;
        y = 2 * x * y + y0;
        x = xtemp;
        iteration++;
    }
```

| Номер запуска программы  |  Флаг компиляции  | Число тактов |
|:------------------------:|:-----------------:|:------------:|
|             1            |        -O0        | 545786612    |
|             2            |        -O0        | 536227482    |
|             3            |        -O0        | 532077848    |
|             1            |        -O3        | 265515064    |
|             2            |        -O3        | 258017412    |
|             3            |        -O3        | 263746198    |

Число тактов (среднее):
- 536 173 152 (-O0)
- 278 407 292 (-O3)

### Вторая программа
В этой программе в основном цикле каждая инструкция обрабатывает четыре точки одновременно, что позволяет использовать векторизацию данных и распараллеливать вычисления между ними за одну итерацию. Используются обычные операции над числами с плавающей точкой (float). В этом коде используются локальные массивы для хранения векторов.

```
for (int n = 0; n < MAX_ITERATIONS; n++) {
    float x2[4] = {}, y2[4] = {}, xy[4] = {}, r2[4] = {};
    for (int i = 0; i < 4; i++) {
        x2[i] = X[i] * X[i];
        y2[i] = Y[i] * Y[i];
        xy[i] = X[i] * Y[i];
        r2[i] = x2[i] + y2[i];
    }
...
```


| Номер запуска программы  |  Флаг компиляции  | Число тактов |
|:------------------------:|:-----------------:|:------------:|
|             1            |        -O0        | 109993435    |
|             2            |        -O0        | 116749835    |
|             3            |        -O0        | 115149833    |
|             1            |        -O3        | 1177766041   |
|             2            |        -O3        | 1056734357   |
|             3            |        -O3        | 1071467434   |

Число тактов (среднее):
- 1 101 989 277 (-O0)
-   113 964 367 (-O3)

### Третья программа
В коде определены функции (mm_set_ps, mm_set_ps1, mm_cpy_ps, mm_add_ps, mm_sub_ps, mm_mul_ps, mm_add_epi32, mm_cmple_ps, mm_movemask_ps), которые выполняют различные операции над векторами и массивами, позволяя упростить код и повысить его читаемость. Код использует массивы для хранения координат X и Y нескольких точек фрактала одновременно, что позволяет проводить параллельные вычисления и ускоряет процесс генерации множества.

```
for (int n = 0; n < MAX_ITERATIONS; n++) {
    mm_mul_ps(x2, X, X);
    mm_mul_ps(y2, X, X);
    mm_mul_ps(xy, X, Y);
..., где

inline void mm_add_ps(float mm[4], const float mm1[4], const float mm2[4]) {
    for (int i = 0; i < 4; i++)
        mm[i] = mm1[i] + mm2[i];
}
```

| Номер запуска программы  |  Флаг компиляции  | Число тактов |
|:------------------------:|:-----------------:|:------------:|
|             1            |        -O0        | 1175882246   |
|             2            |        -O0        | 1092456782   |
|             3            |        -O0        | 1071467434   |
|             1            |        -O3        | 111453729    |
|             2            |        -O3        | 118462758    |
|             3            |        -O3        | 107364748    |

Число тактов (среднее):
-   113 705 544
- 1 124 246 245

### Четвертая программа
Регистры, используемые в этом коде, - это регистры XMM (размером 128 бит). Они используются для хранения векторов числами с плавающей точкой и для выполнения SIMD-инструкций.

SIMD (Single Instruction, Multiple Data): SIMD позволяет одной инструкции выполнять одну операцию над несколькими данными сразу. В этом коде используются 128-битные регистры SIMD (такие как __m128), которые содержат четыре числа с плавающей точкой.

Вместо обработки каждого элемента по отдельности, как это было бы в скалярной реализации, здесь операции производятся сразу над четырьмя числами одновременно, что повышает производительность.

Векторные инструкции, такие как _mm_mul_ps (умножение двух векторов), _mm_add_ps (сложение двух векторов), _mm_sub_ps (вычитание двух векторов) используются для выполнения вычислений над несколькими числами одновременно.

Этот код использует параллельные вычисления для ускорения процесса вычисления множества Мандельброта. Каждая итерация цикла обрабатывается независимо друг от друга, что позволяет распараллелить вычисления.

```
for (int n = 0; n < MAX_ITERATIONS; n++) {
    __m128 x2 = _mm_mul_ps(X, X);
    __m128 y2 = _mm_mul_ps(Y, Y);
    __m128 xy = _mm_mul_ps(X, Y);
...
```

| Номер запуска программы  |  Флаг компиляции  | Число тактов |
|:------------------------:|:-----------------:|:------------:|
|             1            |        -O0        | 329786809    |
|             2            |        -O0        | 343013129    |
|             3            |        -O0        | 337310137    |
|             1            |        -O3        | 81857244     |
|             2            |        -O3        | 78251907     |
|             3            |        -O3        | 79356745     |

Число тактов (среднее):
- 336 703 358 (-O0)
-  79 921 965 (-O3)


### Результаты

|     Версия     |    Файл       | Флаг компиляции   | Число тактов | Рост скорости | FPS   | Рост скорости |
|:--------------:|:-------------:|:-----------------:|:------------:|:-------------:|-------|:-------------:|
|      base      | version1.cpp  |        -O0        | 545786612    |       1       | 4.68  |       1       |
|      base      | version1.cpp  |        -O3        | 278407292    |       2.11    | 9.42  |       2.01    |
| vectorized(4x) | version2.cpp  |        -O0        | 1101989277   |       0.46    | 2.13  |       0.46    |
| vectorized(4x) | version2.cpp  |        -O3        | 113964367    |       4.73    | 20.89 |       4.46    |
| vec(4x) + func | version3.cpp  |        -O0        | 1124246245   |       0.48    | 3.19  |       1.46    |
| vec(4x) + func | version3.cpp  |        -O3        | 113705544    |       4.8     | 21.6  |       0.22    |
| SIMD optimized | version4.cpp  |        -O0        | 336703358    |       1.59    | 7.21  |       1.54    |
| SIMD optimized | version4.cpp  |        -O3        |  79921965    |       7.01    | 30.23 |       6.45    |

### Вывод
Сравнение производительности четырех программ, реализующих алгоритм построения множества Мандельброта, показывает значительное преимущество использования инструкций SSE (Streaming SIMD Extensions).

Сравнивая результаты, можно увидеть, что использование инструкций SSE (четвертая программа) демонстрирует лучшую производительность и быстродействие по сравнению с другими подходами. Это происходит за счет эффективного использования векторизации данных и параллельных вычислений, что позволяет обрабатывать несколько элементов одновременно и значительно сокращает время выполнения алгоритма.
