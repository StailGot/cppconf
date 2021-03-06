#### Ускорение сборки КОМПАСа с помощью pch


![compile](img/compile.jpg)

Началось всё с того, что нам потребовался linux, который за собой потянул переход на CMake. Используемая система сборки умеет только под Windows, точнее позволяет её лицензия. Для всего остального и без того немалая стоимость удваивается. А это уже получается хороший билд сервер. Можно конечно генерировать обратно в sln и собирать им, но уж очень хотелось поменьше лишних телодвижений. Поэтому решили что-то делать. 

На руках были результаты прошлогодней давности, когда пытались сделать что-то [похожее](https://habr.com/ru/company/ascon/blog/585702/). 

Тогда все уперлось во время парсинга и решили оставить ~~пока не припечет~~ до лучших времен.

К счастью пока делали CMake, прибыл новенький билд сервер на 128 ядер (256 потоков) и 1 ТиБ оперативной памяти.

Было где развернуться.

Переход на CMake в качестве бонуса дал доступ к [Ninja](https://ninja-build.org/manual.html), которых значительно лучше распределяет задачи сборки, по сравнению с MSBuild.

Создаем RAM drive, кладем туда исходники, tmp и прочее. Отрубаем все лишние процессы.
Первый запуск сборки и ... получаем время более 10 минут и только половину загрузки CPU. Сказать что это медленно для такой машины, ничего не сказать.

Тут явно что-то не так. Для начала попробуем разобраться, почему так долго.
Посмотрим на исходные данные.

### Что есть ?
- ~3 миллиона строк
- ~4500 .cpp & ~4000 .h
- 55 проектов
- boost, stl, winapi


Проект достаточно давний, при этом активно развивается.

Начнем с самого простого.

Зная что в основном тормозит парсинг, попробуем найти что именно.

Первое что приходит на ум, распарсить все include, посчитать строки и попытаться избавиться от самых тяжелых headers. 

Парсить все руками, разбирать все макросы, учитывать настройки компилятора. Задача выглядит достаточно распространенной, может уже есть готовый инструмент ? 

Оказывается что есть
- Resharper C++ Analyze Includes 


И делает именно то что требуется.
Запускаем и ждем, ждем и еще немного. Парсинг c++ дело не быстрое.


### Resharper C++ Analyze Includes
![resharper_cpp](img/multi_index_container.hpp.png)


И что мы видим, boost. его много и он всюду. Один хедер дает в семь строчек больше чем весь исходный код проекта.

Пробуем с этим что-то сделать. Пока что трогать исходники не сильно хочется. 

### PCH


[PCH](https://ru.wikipedia.org/wiki/%D0%9F%D1%80%D0%B5%D0%B4%D0%B2%D0%B0%D1%80%D0%B8%D1%82%D0%B5%D0%BB%D1%8C%D0%BD%D0%BE_%D0%BE%D1%82%D0%BA%D0%BE%D0%BC%D0%BF%D0%B8%D0%BB%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%BD%D1%8B%D0%B5_%D0%B7%D0%B0%D0%B3%D0%BE%D0%BB%D0%BE%D0%B2%D0%BA%D0%B8) позволяет распарсить один раз включаемые заголовочные файлы, а после при каждом обращении выдавать уже готовое внутреннее представление (обычно это сериализованный ast).

Замеры будем проводить на типичной машине простых смертных, т.к. на ней собирать каждый день.

Попробуем добавить самые жирные хедера. И Получаем результат.
Ускорение примерно в 2 раза. Похоже на чудо. Ничего особо не делая, ускоряем компиляцию в 2 раза, с 50 минут до 25.

Но этого мало. Кажется что можно быстрее.

Смотрим чтобы еще ускорить. Обращаем внимание на сами pch. Размер мягко говоря немаленький, 500 МиБ и более.


Тут уже придется поработать скальпелем. Начинаем с самых жирных кусков и смотрим, действительно ли они там нужны.


Как оказалось что нет. multi_index_container.hpp используется в интерфейсном заголовке, но почему то включен полностью. Сам интерфейс используется достаточно часто, точнее почти везде. Попробуем немного подсократить. 


Для многих тяжелых хедеров существует версия с предварительным объявлением, с суффиксом _fwd. Это справедливо как для boost, так и stl и прочих.


Заменяем, пробуем собрать. Расставляем в .cpp полный хедер по необходимости и спустя несколько часов пыток добиваемся компиляции проекта.


Смотрим на топ 10 тяжелых хедеров и видим что boost там уже стало меньше. Повторяем расстановку хедеров в pch и снова собираем. И видим что время сборки уменьшилось до 20 минут. 


Уже лучше. Относительно небольшими усилиями сократили время сборки. Теперь оно сопоставимо использованию системы параллельной сборки. 


Хм. А можно еще лучше ? Конечно. Повторяем операцию по замене топовых хедеров на fwd версию.


Но чем дальше, тем сложней. В некоторых местах используется реализация. И тут на помощь приходит [pimpl](https://en.cppreference.com/w/cpp/language/pimpl). Суть метода в том, что вместо тяжелой структуры используем указатель на предварительно объявленную структуру. Компилятору достаточно знать размер данных. А размер указателя всегда известен.


И тут натыкаемся на неприятный момент. С сырыми указателями работать совсем не хочется, есть же умные указатели.


Пробуем поставить std::unique_ptr. И Неожиданно получаем ошибку компиляции. 


```c++
class Foo
{
private:
  std::unique_ptr<struct Bar> _bar;
public:
  Foo();
  ~Foo() = default;
}
```
В чем же дело ? Вроде бы нигде не создаем ничего лишнего, но все равно ругается на incomplete type. Присмотримся внимательней. И что мы видим ? Деструктор то остался в заголовке, а значит что для всех полей также должны быть деструкторы. Поэтому верно ругается что не может удалить, т.к. тип неизвестен. Решением является перенос реализации деструктора в .cpp файл.



Подобным образом удаляем остальные тяжелые хедера. Спустя пару недель... получаем ускорение еще на 5 минут. 15 минут, почти как в лучшие времена с использованием incredibuild, но это на типовой локальной машине.


А что дальше ? Можем ведь лучше. Точно знаю.


А дальше остаются более мелкие boost заголовки, stl и собственные хедера и прочая мелочь, которая в совокупности набирает немалый вес. Их уже слишком много чтобы разность по pimpl и fwd. Вот их бы и запихнуть в pch да поплотнее.


И тут Resharper C++  начинает нехватать. Он выдает статистику по числу строк, но это не всегда соответствуем времени компиляции.


Нужно что-то помощней. Вот бы взять какой нибудь полноценный профилировщик, который выдавал все как есть. И как оказалось для msvc относительно недавно microsoft выкатили такой.


### Include What You Use (iwyu)
- удаляет лишние include
- добавляет forward declaration

Также для автоматизации есть инструмент [iwyu](https://github.com/include-what-you-use/include-what-you-use), который поможет немного сократить число лишних инклюдов, расставляя forward declaration и удаляя лишние include

Для работы нужно чтобы проект компилировался под clang, иначе результат будет некорректным. Собираем сам iwyu. Собираем llvm, с включенными опциями.

- `llvm`
- `-DLLVM_ENABLE_PROJECTS="clang;clang-tools-extra"`


Теперь собираем сам iwyu. Для работы так же нужна compile_commands.json.
Запускаем.

```bash
// найти лишние include
iwyu_tool -j 256 -p compile_commands.json -- -w > iwyu_res.cpp
```

На выходе получаем список исправлений. Напрямую его применять не советую. Сама программа находится в beta версии и может поудалять чего лишнего. 

```bash
// применить исправления
fix_includes.py < iwyu_res.cpp
```

Применять лучше по проектно, просматривая предлагаемые изменения. Всяко это будет быстрее чем руками, но полностью автоматически результат не гарантирован.



[C++ Build Insights](https://devblogs.microsoft.com/cppblog/introducing-c-build-insights/)

Инструмент, позволяющий заглянуть в процесс сборки, понять что все это время делает компилятор и линковщик. Начиная от времени парсинга отдельных файлов вплоть до анализа времени генерации и отдельных функций.

Прежде чем воспользоваться, нужно настроить систему и установить несколько инструментов.

Для этого нам понадобится свежий ADK и Visual Sdtudio 2019.

После установки ADK нужно настроить. Для этого лезем в нутрь студии и ищем `perf_msvcbuildinsights.dll`. Он отвечает за отображение результатов. Теперь заходим в WPA копируем его туда. Так же нужно прописать в настройках `perfcore.ini` add `perf_msvcbuildinsights.dll`

### C++ Build Insights
- Visual Studio 2019
- [latest Windows ADK](https://docs.microsoft.com/en-us/windows-hardware/get-started/adk-install)
- `perf_msvcbuildinsights.dll` -> WPA directory
- `perfcore.ini` add `perf_msvcbuildinsights.dll`


Начнем замерять.
Для этого от имени администратора стартуем консоль разработчика x64 `Native Tools Command Prompt for VS 2019` (vcvars64) от имени администратоа. И стартуем сессию `vcperf /start MySessionName`

Сам процесс выглядит следующим образом. Запускаем профилировщик. Глобально. Неважно из какой директории, главное чтобы хватало место и прав на запись.

Далее стартуем сборку. Захватываются все события системы. Поэтому не должно быть ничего лишнего. Также потребуется немало оперативной памяти, чем больше проект тем больше. В данном случае хватило 32 ГиБ.

После завершения компиляции устанавливаем профилировщик `vcperf /stop MySessionName outputFile.etl`

### C++ Build Insights
- `vcperf /start MySessionName`
- compile your project
- `vcperf /stop MySessionName outputFile.etl`


Если все было установлено и настроено правильно, то при открытии отчета видим дополнительные вкладки

###### C++ Build Insights
![profiler step 1](img/linux-feature-draft-boost-hpps-optimization.png)


В данном случае нас интересует время парсинга файлов. 
Смотрим и видим что boost еще в топ 10, хотя по числу строк его там нет. Почему так ?
Смотрим, а чего же там такого тяжелого ? Вроде ничего. Один boost хедер включает другой, а потом еще один и еще и еще... И так до 1500. 

Вспомним как работает компилятор. С берем настройки и начинаем парсить. На каждый файл производится операция открытия, чтения и закрытия. И так каждый раз. А если таких файлов очень много ? Windows просто утопает в создании процесса и открытии файлов. Т.е. получается что наши хедера попали в том потому что застреваем в IO.


Используя предыдущий опыт, так же пробуем избавиться от самых тяжелых, пока не останется много мелочевки. Её попытаемся затолкать в pch.

Так же берем топ N хедеров по проектам и распихиваем в pch. 

Но так делать каждый раз мягко говоря утомительно. Проект стремительно развивается. Что-то добавляется а что-то удаляется. Вот бы ~~сани ехали сами~~, pch сам генерировался.

Для этого нужно как-то достать инклюды из etl, и записать в pch.

Оказывается всё уже есть.

https://devblogs.microsoft.com/cppblog/analyze-your-builds-programmatically-with-the-c-build-insights-sdk/

Собираем https://github.com/microsoft/cpp-build-insights-samples/tree/master/TopHeaders

Натравливаем на получившийся etl и получаем заветные хедера, которые после обработки записываем в pch
![top_headers](img/top_headers.png)

Остается дело за малым. Собираем каждый проект по отдельности, сохраняем отчет.
После чего извлекаем топ N хедеров и вставляем их pch.


### Попытка 1
Берем топ 50 для каждого узла. Делаем еще один замер и получаем ускорение еще в несколько минут.
Неплохо, но кажется что можно еще лучше.

Внимательней посмотрим на список. Оказывается в топе попадаются хедера которые включены в другие хедера из того же топа. Т.е. топ 50 на самом деле вовсе не 50, а как повезет.

К тому же, как оказалось не все можно взять и просто так включить. Где-то попадаются одинаковые typedef с разными значениями или внезапно обнаружились определения макросов, которые пересекаются с именами классов. Проект все таки давний и вряд ли авторы задумывались что кто-то захочет свалить столько хедеров в кучу. 

По идее нам нужны только те хедера, которые включают в себя все остальные.

Оказывается необходимая информация в виде дерева инклюдов есть в etl отчете, которую также можно извлечь.

Далее все просто. Берем полное дерево инклюдов, так же берем топ N инклюдов.
Строим граф. Ищем топовые вершины первого уровня. 
В итоге из 50 осталось около 10, остальные так или иначе включают друг друга.

![with_pch](img/with_pch.png)

### Попытка 2
Пробуем, еще минус пару минут. Теперь можно поиграться с настройками. Нужно понять сколько же брать. Чем больше хедеров пихаем в pch, тем дольше время сборки на машине со множество ядер (Epyc на 256 потоков), но на обычной наоборот, получается быстрее. Дело в том, создание pch происходит в один поток, в это время остальные ждут, но зато парсинг происходит практически мгновенно. Тут нужно подбирать в зависимости от проекта или написать скрипт, который сделает это сам )

Итак, делаем еще один замер и еще минус пару минут. 

Итого с более чем 50 минут, получили около 10 на типовой машине. Естественно результаты могут и отличаются на каждой машине из-за наличия фоновых процессов, работающего антивируса, количества открытых вкладок в хроме. Но в целом получилось быстрее чем с использованием системы параллельной сборки, при этом намного проще чем у коллег из [питера](https://habr.com/ru/company/ascon/blog/585702/) :)

Еще одним приятным бонусом стала возможность работать не используя всю вычислительную мощь компании, что актуально для удаленных сотрудников, которых стало значительно в такое время.


### Результаты
![result](img/result.png)


### Спустя полгода
За все время использования скорость сборки почти не изменилась. Делалось несколько перегенераций pch. Об этом несколько подробней.

Для генерации нужно делать замеры на чистой сборке, без pch.
Т.к. туда попадают внутренние хедера, то происходит некоторое смешение и каждой единице трансляции становится видно чуть больше чем раньше. При компиляции без pch те места, где символы попадали из pch, начинают отваливатся. Такие случаи пока что прихоится править руками. Хорошо что не часто, раз в пару месяцев или после крупных изменений.


### Что дальше ?

Поддерживать инклуды все же сложновато. Вся надежда остается на модули. Пока что нет полноценной поддержки со стороны cmake [(C++ modules support?)](https://gitlab.kitware.com/cmake/cmake/-/issues/18355), поэтому о результатах говорить рано. Все равно нужно будет переработать код и нет полных гарантий что это будет сильно быстрее pch, но зато можно надеяться что после правильной разбивки регресс будет происходить значительно реже.


- [Analyze your builds programmatically with the C++ Build Insights SDK](https://devblogs.microsoft.com/cppblog/analyze-your-builds-programmatically-with-the-c-build-insights-sdk/)
- [C++ Build Insights SDK samples](https://github.com/microsoft/cpp-build-insights-samples)
- [Faster builds with PCH suggestions from C++ Build Insights](https://devblogs.microsoft.com/cppblog/faster-builds-with-pch-suggestions-from-c-build-insights/)
- [Finding build bottlenecks with C++ Build Insights](https://devblogs.microsoft.com/cppblog/finding-build-bottlenecks-with-cpp-build-insights/)
- [LLVM](https://github.com/llvm/llvm-project)
- [Include What You Use (IWYU)](https://github.com/include-what-you-use/include-what-you-use)
