## Сборщик мусора
## 1. Что такое сборщик мусора?
    Сборщик мусора — это низкоприоритетный процесс, который запускается периодически и освобождает память, использованную объектами, которые больше не нужны.
    Объект может подлежать утилизации в разных случаях:
    Если переменная ссылочного типа, которая ссылается на объект, установлена в положение "0", объект подлежит утилизации, в том случае, если на него нет других ссылок.
    Если переменная ссылочного типа, которая ссылается на объект, создана для ссылки на другой объект, объект подлежит утилизации, в том случае, если на него нет других ссылок.
    Объекты, созданные локально в методе, подлежат утилизации, когда метод завершает работу, если только они не экспортируются из этого метода (т.е, возвращаются или генерируются как исключение).
    Объекты, которые ссылаются друг на друга, могут подлежать утилизации, если ни один из них не доступен живому потоку.
## 2. Способы взаимодействия со сборщиком мусора для разработчика
    Методы System.gc() или Runtime.getRuntime().gc() не запускают процесс сборки мусора, а лишь «рекомендуют» это сделать.
    Вызов метода System.runFinalization() приведет к запуску метода finalize() для объектов, утративших все ссылки.
## 3. Strong, Soft, Weak, Phantom references
    В Java есть несколько видов ссылок:
1. StrongReference — это самые обычные ссылки.
2. SoftReference – мягкие ссылочные объекты, которые очищаются по усмотрению сборщика мусора в ответ на потребность в памяти. SoftReference чаще всего используются для реализации кэшей, чувствительных к памяти.  GC гарантировано удалит с кучи все объекты, доступные только по soft-ссылке, перед тем как бросит OutOfMemoryError..
3. WeakReference – слабые ссылочные объекты, которые могут быть финализированными и затем возвращенными. Слабые ссылки чаще всего используются для реализации канонических отображений.
4. PhantomReference – фантомные ссылочные объекты, которые поставлены в очередь после того, как сборщик определил, что их ссылки могут быть возвращены. Фантомные ссылки чаще всего используются для планирования предварительных действий по очистке более гибким способом, чем это возможно с помощью механизма финализации Java. В отличие от мягких и слабых ссылок, фантомные ссылки не удаляются автоматически сборщиком мусора, когда они помещаются в очередь. Объект, доступный через фантомные ссылки, останется таковым до тех пор, пока все такие ссылки не будут удалены или сами не станут недоступными.
## 4. Распределение памяти в JVM
   Память процесса делится на Stack (стек) и Heap (куча) и включает 5 областей :
1)     Stack
a)     Permanent Generation — используемая JVM память для хранения метаинформации; классы, методы и т.п.
b)     Code Cache — используемая JVM память при включенной JIT-компиляции; в этой области памяти кешируется скомпилированный платформенно-зависимый код.
2) 	Heap
      a)     Eden Space — в этой области выделяется память под все создаваемые программой объекты. Жизненный цикл большей части объектов, к которым относятся итераторы, объекты внутри методов и т.п., недолгий.
      b)     Survivor Space — здесь хранятся перемещенные из Eden Space объекты после первой сборки мусора. Объекты, пережившие несколько сборок мусора, перемещаются в следующую сборку Tenured Generation.
      c)     Tenured Generation хранит долгоживущие объекты. Когда данная область памяти заполняется, выполняется полная сборка мусора (full, major collection).
## 4.1 Permanent Generation
      Область памяти Permanent Generation используется виртуальной машиной JVM для хранения необходимых для управления программой данных, в том числе метаданные о созданных объектах. При каждом создании объекта JVM будет сохранять некоторый набор данных об объекте в области Permanent Generation. Соответственно, чем больше создается в программе объектов, тем больше требуется «пространства» в Permanent Generation.
      Размер Permanent Generation можно задать двумя параметрами виртуальной машины JVM :
      -XX:PermSize – минимальный размер выделяемой памяти для Permanent Generation;
      -XX:MaxPermSize – максимальный размер выделяемой памяти для Permanent Generation.
      Для «больших» Java-приложений можно при запуске определить одинаковые значения данных параметров, чтобы Permanent Generation была создана с максимальным размером. Это может увеличить производительность, поскольку динамическое изменение размера Permanent Generation является «дорогостоящей» (трудоёмкой) операцией. Определение одинаковых значений этих параметров может избавить JVM от выполнения дополнительных операций, связанных с проверкой необходимости изменения размера Permanent Generation.
## 4.2. Область памяти Heap
      Куча Heap является основным сегментом памяти, где хранятся создаваемые объекты. Heap делится на два подсегмента : Tenured (Old) Generation и New Generation. New Generation в свою очередь делится на Eden Space и Survivor.
      Размер сегмента Heap можно определить двумя параметрами : Xms (минимум) и -Xmx (максимум).
## 4.3 В чем отличие между сегментами Stack и Heap?
      Heap (куча) используется всеми частями приложения, а Stack используется только одним потоком исполнения программы.
      Новый объект создается в Heap, а в памяти Stack'a размещается ссылка на него. В памяти стека также размещаются локальные переменные примитивных типов.
      Объекты в куче доступны из любого места программы, в то время, как стековая память не доступна для других потоков.
      Если память стека полностью занята, то Java Runtime вызывает исключение java.lang.StackOverflowError, а если память кучи заполнена, то вызывается исключение java.lang.OutOfMemoryError: Java Heap Space.
      Размер памяти стека, как правило, намного меньше памяти в куче. Из-за простоты распределения памяти (LIFO), стековая память работает намного быстрее кучи.
## 5. Сравнение Serial, Parallel, Cms, C1 GC
      Слабая гипотеза о поколениях – вероятность смерти как функция от возраста снижается очень быстро. Ее приложение к сборке мусора в частности означает, что подавляющее большинство объектов живут крайне недолго. Также это означает, что чем дольше прожил объект, тем выше вероятность того, что он будет жить и дальше. Из этого возникает идея разделения объектов на младшее поколение (young generation) и старшее поколение (old generation). В соответствии с этим разделением и процессы сборки мусора разделяются на малую сборку (minor GC), затрагивающую только младшее поколение, и полную сборку (full GC), которая может затрагивать оба поколения. Малые сборки выполняются достаточно часто и удаляют основную часть мертвых объектов. Полные сборки выполняются тогда, когда текущий объем выделенной программе памяти близок к исчерпанию и малой сборкой уже не обойтись.
      Факторы работы сборщика мусора:
      Максимальная задержка — максимальное время, на которое сборщик приостанавливает выполнение программы для выполнения одной сборки. Такие остановки называются stop-the-world (или STW).
      Пропускная способность — отношение общего времени работы программы к общему времени простоя, вызванного сборкой мусора, на длительном промежутке времени.
      Потребляемые ресурсы — объем ресурсов процессора и/или дополнительной памяти, потребляемых сборщиком.
      Java HotSpot VM предоставляет разработчикам на выбор четыре различных сборщика мусора:
      Serial (последовательный) — самый простой вариант для приложений с небольшим объемом данных и не требовательных к задержкам. Редко когда используется, но на слабых компьютерах может быть выбран виртуальной машиной в качестве сборщика по умолчанию.
      Parallel (параллельный) — наследует подходы к сборке от последовательного сборщика, но добавляет параллелизм в некоторые операции, а также возможности по автоматической подстройке под требуемые параметры производительности.
      Concurrent Mark Sweep (CMS) — нацелен на снижение максимальных задержек путем выполнения части работ по сборке мусора параллельно с основными потоками приложения. Подходит для работы с относительно большими объемами данных в памяти.
      Garbage-First (G1) — создан для постепенной замены CMS, особенно в серверных приложениях, работающих на многопроцессорных серверах и оперирующих большими объемами данных.
## 5.1 Serial GC
      Принципы работы:
      Куча разбивается на четыре региона, три из которых относятся к младшему поколению (Eden, Survivor 0 и Survivor 1), а один (Tenured) — к старшему
      Среднестатистический объект начинает свою жизнь в регионе Eden. Акселераты помещаются сразу в Tenured.
      Когда перестает хватать места в Eden, запускается малая сборка мусора
      Сборка находит и удаляет мертвые объекты из Eden. Оставшиеся живые объекты переносятся в пустой регион Survivor.
      JVM постоянно следит за тем, как долго объекты перемещаются между Survivor 0 и Survivor 1, и выбирает подходящий порог для количества таких перемещений, после которого объекты перемещаются в Tenured.
      В случае, когда места для новых объектов не хватает уже в Tenured, в дело вступает полная сборка мусора, работающая с объектами из обоих поколений. При этом старшее поколение не делится на субрегионы по аналогии с младшим, а представляет собой один большой кусок памяти, поэтому после удаления мертвых объектов из Tenured производится не перенос данных (переносить уже некуда), а их уплотнение, то есть размещение последовательно, без фрагментации. Такой механизм очистки называется Mark-Sweep-Compact.
      Ситуации STW
      В начале каждой сборки мусора работа основных потоков приложения останавливается и возобновляется только после окончания сборки. Причем всю работу по очистке Serial GC выполняет в одном потоке, последовательно.
      Достоинства и недостатки
      Достоинство —непритязательность по части ресурсов компьютера.
      Недостаток — долгие паузы на сборку мусора при заметных объемах данных.
## 5.2. Parallel GC
      Принципы работы:
      Применяются те же принципы построения памяти, что и у Serial GC, но сборкой мусора занимаются несколько потоков параллельно и данный сборщик может самостоятельно подстраиваться под требуемые параметры производительности.
      По умолчанию и малая и полная сборка задействуют многопоточность. Малая пользуется ею при переносе объектов в старшее поколение, а полная — при уплотнении данных в старшем поколении.
      Каждый поток сборщика получает свой участок памяти в регионе Old Gen, так называемый буфер повышения (promotion buffer), куда только он может переносить данные, чтобы не мешать другим потокам. Такой подход ускоряет сборку мусора, но имеет и небольшое негативное последствие в виде возможной фрагментации памяти.
      Ситуации STW
      Как и в случае с последовательным сборщиком, на время операций по очистке памяти все основные потоки приложения останавливаются. Разница только в том, что пауза, как правило, короче за счет выполнения части работ в параллельном режиме.
      Достоинства и недостатки
      Достоинства по сравнению с Serial GC является возможность автоматической подстройки под требуемые параметры производительности и меньшие паузы на время cборок. При наличии нескольких процессорных ядер выигрыш в скорости будет практически во всех приложениях.
      Недостаток - определенная фрагментация памяти.
## 5.3. CMS (Concurrent Mark Sweep) GC
      Принципы работы:
      CMS GC использует ту же организацию памяти, что и Serial / Parallel GC: регионы Eden + Survivor 0 + Survivor 1 + Tenured.
      CMS GC использует такой же принцип малой сборки мусора, что и Serial / Parallel GC.
      CMS вместо процесса полной (full) сборки реализует процесс старшей (major) сборки, так как он не затрагивает объекты младшего поколения. В результате, малая и старшая сборки здесь всегда разделены.
      CMS разносить во времени малые и старшие сборки мусора, чтобы они совместно не создавали продолжительных пауз в работе приложения.
      Важным отличием сборщика CMS от Serial и Parallel является то, что он не дожидается заполнения Tenured для того, чтобы начать старшую сборку. Вместо этого он трудится в фоновом режиме постоянно, пытаясь поддерживать Tenured в компактном состоянии.
      Major сборка
      Остановка основных потоков приложения и пометка всех объектов, напрямую доступных из корней.
      Приложение возобновляет свою работу, а сборщик параллельно с ним производит поиск всех живых объектов, доступных по ссылкам из помеченных корневых объектов (эту часть он делает в одном или в нескольких потоках).
      Сборщик еще раз приостанавливает работу приложения и просматривает кучу для поиска живых объектов, ускользнувших от него за время первого прохода.
      Работа основных потоков приложения возобновляется, а сборщик производит очистку памяти от мертвых объектов в нескольких параллельных потоках.
      Если сборщик не успевает очистить Tenured до того момента, как память полностью заканчивается, работа приложения останавливается, и вся сборка производится в последовательном режиме – сбой конкурентного режима (concurrent mode failure).
      Ситуации STW
      Малая сборка мусора. Эта пауза ничем не отличается от аналогичной паузы в Parallel GC.
      Начальная фаза поиска живых объектов при старшей сборке (так называемая initial mark pause). Эта пауза обычно очень короткая.
      Фаза дополнения набора живых объектов при старшей сборке (известная также как remark pause). Она обычно длиннее начальной фазы поиска.
      Достоинства и недостатки
      Достоинства – ориентированность на минимизацию времен простоя, что является критическим фактором для многих приложений.
      Недостатки:
      для выполнения минимизации времени простоя приходится жертвовать ресурсами процессора и зачастую общей пропускной способностью.
      данный сборщик не уплотняет объекты в старшем поколении, что приводит к фрагментации Tenured. Этот факт в совокупности с наличием плавающего мусора приводит к необходимости выделять приложению (конкретно — старшему поколению) больше памяти, чем потребовалось бы для других сборщиков (Oracle советует на 20% больше).
      долгие паузы при потенциально возможных сбоях конкурентного режима могут стать неприятным сюрпризом. Хотя они не частые, и при наличии достаточного объема памяти CMS’у удается их полностью избегать.
## 5.4. G1 (Garbage First) GC
      Принципы работы
      Здесь память разбивается на множество регионов одинакового размера. Размер этих регионов зависит от общего размера кучи и по умолчанию выбирается так, чтобы их было не больше 2048, обычно получается от 1 до 32 МБ. Исключение составляют только так называемые громадные (humongous) регионы, которые создаются объединением обычных регионов для размещения очень больших объектов.
      Малые сборки выполняются периодически для очистки младшего поколения и переноса объектов в регионы Survivor, либо их повышения до старшего поколения с переносом в Tenured.    	Над переносом объектов трудятся несколько потоков, и на время этого процесса работа основного приложения останавливается.
      При этом он выбирает для очистки те регионы, в которых, по его мнению, скопилось наибольшее количество мусора и очистка которых принесет наибольший результат.
      Полная сборка (смешанная (mixed)). В G1 существует процесс,  называемый циклом пометки (marking cycle), который работает параллельно с основным приложением и составляет список живых объектов:
      Initial mark. Пометка корней (с остановкой основного приложения) с использованием информации, полученной из малых сборок.
      Concurrent marking. Пометка всех живых объектов в куче в нескольких потоках, параллельно с работой основного приложения.
      Remark. Дополнительный поиск не учтенных ранее живых объектов (с остановкой основного приложения).
      Cleanup. Очистка вспомогательных структур учета ссылок на объекты и поиск пустых регионов, которые уже можно использовать для размещения новых объектов. Первая часть этого шага выполняется при остановленном основном приложении.
      Ситуации STW
      Процессы переноса объектов между поколениями. Для минимизации таких пауз G1 использует несколько потоков.
      Короткая фаза начальной пометки корней в рамках цикла пометки.
      Более длинная пауза в конце фазы remark и в начале фазы cleanup цикла пометки.
      Достоинства и недостатки
      Достоинства – сборщик G1 более аккуратно предсказывает размеры пауз и лучше распределяет сборки во времени, чтобы не допустить длительных остановок приложения, особенно при больших размерах кучи. При этом он лишен и некоторых других недостатков CMS, например, он не фрагментирует память.
      Недостатки – ресурсы процессора, которые он использует для выполнения достаточно большой части своей работы параллельно с основной программой. В результате страдает пропускная способность приложения.
## 6. Epsilon GC
      Эпсилон — это сборщик мусора, который обрабатывает выделение памяти, но не реализует какой-либо реальный механизм ее восстановления памяти. Как только доступная куча Java будет исчерпана, JVM закроется. То есть, если в бесконечном массиве запустить создание объекта без привязки к ссылке с данным сборщиком мусора, приложение упадёт с OutOfMemoryError (а если с любым другим нет, так как он будет подчищать объекты без ссылок). Зачем он нужен? А вот зачем:
      Тестирование производительности.
      Тестирование давления памяти.
      Тестирование интерфейса VM.
      Чрезвычайно недолгая работа.
      Улучшения латентности последней капли.
      Улучшения пропускной способности последней капли.