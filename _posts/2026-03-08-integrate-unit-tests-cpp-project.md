---
layout: post
title: "Интеграция модульных тестов<br> в проект на C++"
---

Предыдущие посты в этой серии постов:
- [Как начать изучать тестирование программ на C++](https://ilyachalov.livejournal.com/386716.html);
- [Начало изучения системы сборки программ CMake]({% post_url 2026-02-23-cmake-build-system-learning %}).

Я работаю в операционной системе Windows 10. У меня установлена среда Visual Studio Community с рабочей нагрузкой «Desktop Development with C++». Но в рамках этой статьи я не использую графический интерфейс среды Visual Studio Community, а работаю из [командной строки разработчика](https://learn.microsoft.com/en-us/visualstudio/ide/reference/command-prompt-powershell) этой среды. (Обычная командная строка не подойдет, так как в ней без дополнительной настройки недоступны нужные инструменты.)

Такой способ работы выбран для того, чтобы студенты, использующие операционные системы из семейства Linux (у нас в колледже есть такие), могли выполнить те же операции, что и студенты, использующие системы Windows.

По этой же причине в качестве тестирующего фреймворка выбран Google Test в связке с системой сборки CMake. Оба эти инструмента кроссплатформенные, то есть работают в системах Windows и Linux, а кроме этого они входят в рабочую нагрузку «Desktop Development with C++» среды Visual Studio Community, что удобно для программистов, использующих Windows: нет необходимости в дополнительной установке этих двух инструментов.

## Создание подпапок и файлов проекта

Папке проекта я дал название `ProjectFolderWithTests`. (Название папки рабочего [не учебного] проекта, конечно, должно быть более осмысленным.) Папки и файлы, перечисленные здесь, создаю вручную.

```
ProjectFolderWithTests
│   CMakeLists.txt
│   main.cpp
│
├───modules
│       CMakeLists.txt
│       lib.cpp
│       lib.h
│
└───tests
        CMakeLists.txt
        tests-for-lib.cpp
```

Пока что все перечисленные файлы пустые. Для заполнения открываю все файлы одновременно в редакторе кода (мы используем VS Code). Поскольку все эти файлы работают в связке, имеет смысл писать их одновременно.

Файлы я делю на три группы:
- файлы, выполняющие задачу, для решения которой создан проект: `main.cpp`, `lib.cpp`, `lib.h`;
- файлы с тестами: пока только один `tests-for-lib.cpp`;
- файлы со скриптом для сборки: три файла `CMakeLists.txt`, каждый в своей папке.

Для сборки у нас будет две цели (target): получение исполняемого файла из файлов первой группы (решение задачи проекта), получение исполняемого файла для тестирования модулей проекта.

## Первая цель: исполняемый файл, решающий задачу

Заполняем исходные файлы:

```cpp
// main.cpp (в кодировке UTF-8)

#include <iostream>
#include "modules/lib.h"

int main()
{
    std::cout << "Факториал числа 8 равен " << factorial(8) << '\n';

    return 0;
}
```

```cpp
// modules/lib.h

long long factorial(int n);
```

```cpp
// modules/lib.cpp

long long factorial(int n)
{
    if (n < 0) return -1;
    long long fact{ 1 };
    for (int i{ 1 }; i <= n; i++)
    {
        fact *= i;
    }
    return fact;
}
```

Заполняем файлы для сборки:

```cmake
# CMakeLists.txt

cmake_minimum_required(VERSION 3.23)
project(ProjectFolderWithTests)

add_subdirectory(modules)

add_executable(app main.cpp)
target_link_libraries(app lib)
```

```cmake
# modules/CMakeLists.txt

project(lib)

add_library(lib lib.cpp)
```

Создаем [генератор](https://cmake.org/cmake/help/latest/manual/cmake-generators.7.html) в дополнительной подпапке `build` с помощью следующей команды из командной строки:

```
cmake -B build
```

Создаем с помощью генератора из подпапки `build` исполняемый файл приложения с помощью следующей команды из командной строки:

```
cmake --build build --target app
```

После этого приложение (оно в данном случае консольное) я запускаю из папки проекта из командной строки так:

```
build\Debug\app
```

Вообще исполняемый файл у меня получился `app.exe`, но расширение можно при запуске из командной строки не указывать. Предварительно я включил для командной строки кодовую страницу 65001 (кодировка UTF-8), чтобы русские буквы отобразились корректно. Это можно сделать с помощью команды `chcp 65001`. По умолчанию используется конфигурация сборки `Debug` (в случае наличия нескольких конфигураций сборки можно выбрать конкретную с помощью параметра `--config` [[подробнее](https://cmake.org/cmake/help/latest/guide/user-interaction/index.html#invoking-the-buildsystem)] системы сборки cmake).

## Вторая цель: тестирование модулей

Заполняем исходные файлы с тестами, используя инструменты, предоставляемые тестовым фреймворком Google Test. (Чтобы получить понимание этих инструментов хотя бы на начальном уровне, рекомендую прочитать [учебник для начинающих](https://google.github.io/googletest/primer.html) [primer] на официальном сайте этого фреймворка.) Пока что у меня один исходный файл с тестами:

```cpp
// tests/tests-for-lib.cpp

#include <gtest/gtest.h>    // заголовочный файл фреймворка
#include "../modules/lib.h" // заголовочный файл тестируемого модуля

TEST(FactorialTestSuite, NegativeInputs)
{
    EXPECT_EQ(-1, factorial(-5));
}
```

В этом файле пока только один тест, созданный с помощью макроса `TEST` фреймворка. В будущем можно добавить сколько угодно других тестов. Макрос `TEST` принимает в скобках название набора тестов (test suit) и название конкретного теста из этого набора. В данном случае набор тестов называется `FactorialTestSuite` (набор тестов для тестирования модуля/функции, вычисляющей факториал числа), а конкретный тест из этого набора называется `NegativeInputs` (тестируем работу указанной функции в случае, когда входящий параметр функции является отрицательным числом).

Внутри теста могут содержаться утверждения (assertion). В актуальной версии фреймворка Google Test названия утверждений начинаются либо с последовательности `ASSERT_`, либо с последовательности `EXPECT_`. Отличие между ними в следующем. Если тестируемый код не соответствует утверждению `ASSERT_`, тестирование прерывается. Если тестируемый код не соответствует утверждению `EXPECT_`, тестирование не прерывается. В данном случае у меня только одно утверждение, поэтому разницы не будет видно. Эта разница имеет значение, когда есть ряд утверждений. Оба варианта имеют свою область применения, а также могут применяться совместно. В [учебнике для начинающих](https://google.github.io/googletest/primer.html#assertions) есть хороший пример их совместного применения.

Редактор Visual Studio Code, который мы используем, по умолчанию не находит заголовочный файл фреймворка `gtest/gtest.h` и подчеркивает эту строку красной линией, сигнализируя об ошибке в коде. Я пока не стал разбираться, как с этим справиться, поэтому просто игнорирую это подчеркивание. Тестирование я запускаю из командной строки разработчика Visual Studio, там эта ошибка не возникает.

Дополним главный файл с настройками сборки для системы CMake:

```cmake
# CMakeLists.txt

cmake_minimum_required(VERSION 3.23)
project(ProjectFolderWithTests)

include(FetchContent)
string(CONCAT GTEST_URL "https://github.com/google/googletest/archive/"
                        "52eb8108c5bdec04579160ae17225d66034bd723.zip")
FetchContent_Declare(
  googletest
  URL ${GTEST_URL}
)
# For Windows: Prevent overriding the parent project's compiler/linker settings
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
FetchContent_MakeAvailable(googletest)

add_subdirectory(modules)
add_subdirectory(tests)

add_executable(app main.cpp)
target_link_libraries(app lib)
```

По сравнению с предыдущей версией этого файла в этой статье добавился блок кода, начинающийся с инструкции `include(FetchContent)` и заканчивающийся инструкцией `FetchContent_MakeAvailable(googletest)`. Таким образом мы загружаем из интернета и подключаем к нашему проекту тестирующий фреймворк Google Test. Этот код взят с официального сайта этого фреймворка, из руководства «[Quickstart: CMake](https://google.github.io/googletest/quickstart-cmake.html)».

Я добавил в этот код инструкцию, начинающуюся с `string(CONCAT GTEST_URL`, чтобы уложить длинный URL в традиционную ширину программ (около 80 символов). Для этого есть ряд способов, я выбрал этот, его [советуют](https://stackoverflow.com/questions/7637539/how-to-split-strings-across-multiple-lines-in-cmake) на сайте Stack Overflow.

Еще я поменял уникальный идентификатор коммита (commit hash) на более актуальный, для самой свежей версии фреймворка Google Test. На данный момент это версия [1.17.0](https://github.com/google/googletest/releases/tag/v1.17.0), ей соответствует коммит `52eb8108c5bdec04579160ae17225d66034bd723`.

Фреймворк Google Test, встроенный в Visual Studio, у меня не получилось использовать при тестировании из командной строки разработчика Visual Studio. Думаю, встроенный в Visual Studio фреймворк Google Test предназначен только для работы из графического (оконного) интерфейса Visual Studio. Поэтому пришлось использовать вышеописанный способ, использующий модуль `FetchContent` ([подробнее](https://cmake.org/cmake/help/latest/module/FetchContent.html)) системы сборки CMake.

Также я добавил в файл `CMakeLists.txt` инструкцию `add_subdirectory(tests)` для подключения файла `tests/CMakeLists.txt`. Заполним этот файл:

```cmake
# tests/CMakeLists.txt

project(tests)

add_executable(tests tests-for-lib.cpp)
target_link_libraries(tests
    PUBLIC
        lib
        gtest
        gtest_main
)
```

Вообще в программе с тестами тоже должна быть своя функция `main`, как и в любой программе на языках Си или C++. Однако, насколько я понимаю, для удобства работы с фреймворком Google Test ее убрали в библиотеку `gtest_main`, поэтому мы подключаем эту библиотеку, а саму функцию `main` самостоятельно не пишем.

Генератор был создан ранее, поэтому для компиляции (сборки) тестов использую следующую команду в командной строке (при указании цели вместо `app` указываю `tests`):

```
cmake --build build --target tests
```

В выданной в консоль информации можно будет увидеть путь к исполняемому файлу. Запускаем этот исполняемый файл из командной строки (у меня он называется `tests.exe`, но расширение при запуске указывать необязательно):

```
build\tests\Debug\tests
```

Результат:

![](/images/cmd-gtest.png)

## Полезные ссылки

При подготовке этого поста использовал следующие источники:

- статья «[Google Test: интеграция модульных тестов в C/C++ проекты](https://nuancesprog.ru/p/15603/)»;
- статья «[Getting started with GoogleTest and CMake](https://dev.to/yanujz/getting-started-with-googletest-and-cmake-1kgg)».
