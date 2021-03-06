## Как модель актора отвечает потребностям современных распределенных систем

Как описано в предыдущей теме, общая практика программирования не удовлетворяет должным образом требованиям современных 
систем. К счастью, нам не нужно забывать все, что мы знаем. Вместо этого модель актора устраняет эти недостатки принципиально, 
позволяя системам вести себя таким образом, который лучше соответствует нашей ментальной модели. Абстракция модели актора
 позволяет вам думать о своем коде с точки зрения коммуникации, в отличие от обменов, которые происходят между людьми в 
 большой организации.

Использование акторов позволяет нам:

* обеспечить инкапсуляцию, не прибегая к блокировкам;

* использовать модель сотрудничества объектов, реагирующих на сигналы, изменяя состояние и отправляя сигналы друг другу, 
чтобы продвинуть все приложение вперед;

* перестать беспокоиться о механизме выполнения, который является несоответствием нашему мировоззрению.

### Использование передачи сообщений позволяет избежать блокировки

Вместо вызова методов, участники отправляют сообщения друг другу. Отправка сообщения не переносит поток выполнения от 
отправителя к месту назначения. Актор может отправить сообщение и продолжить без блокировки. Таким образом, он может 
достичь большего за один и тот же промежуток времени.

С объектами, когда метод возвращается, он освобождает управление своему исполняемыму потокоу. В этом отношении акторы 
ведут себя как объекты, реагируют на сообщения и возвращают выполнение, когда они завершают обработку текущего сообщения. 
Таким образом, акторы фактически достигают исполнения, которое мы представляем для объектов:
 
![alt text](https://github.com/steklopod/akka/blob/akka_starter/src/main/resources/images/how-the-actor-model-meets-the-needs-of-modern-distributed-systems/actor_graph.png "actor_graph")

 Важным отличием между передачей сообщений и вызовами является то, что **сообщения не имеют возвращаемого значения**. Отправляя 
сообщение, **актор делегирует работу другому актору**. Как мы видели в [иллюзии стека вызовов](https://github.com/steklopod/akka/blob/akka_starter/src/main/resources/readmes/why-modern-systems-need-anew-programming-model.md)
 , если он ожидал возвращаемого значения, отправляющему актору нужно было либо заблокировать, либо выполнить работу другого 
 актора в том же потоке. Вместо этого принимающий актор передает результаты в ответное сообщение.

 Второе ключевое изменение, которое нам необходимо в нашей модели, - восстановить инкапсуляцию. Акторы реагируют на 
сообщения так же, как объекты «реагируют» на используемые им методы. Разница заключается в том, что вместо того, чтобы 
несколько потоков «внедрялись» в нашего актора и разрушали внутреннее состояние и инварианты, участники выполняли независимо 
от отправителей сообщения, и они последовательно реагировали на входящие сообщения, по одному за раз. В то время как каждый 
актор обрабатывает сообщения, отправленные ему последовательно, разные участники работают одновременно друг с другом, так 
что акторская система может обрабатывать столько сообщений одновременно, сколько будет поддерживать аппаратное обеспечение.

 Поскольку на актора всегда обрабатывается не более одного сообщения, инварианты актора могут сохраняться без синхронизации. 
Это происходит автоматически без использования блокировок:

![alt text](https://github.com/steklopod/akka/blob/akka_starter/src/main/resources/images/how-the-actor-model-meets-the-needs-of-modern-distributed-systems/serialized_timeline_invariants.png "serialized_timeline_invariants")

#### Что происходит, когда актор получает сообщение:

1. Актор добавляет сообщение в конец очереди;

2. Если актор не был запланирован для выполнения, он помечается как готовый к исполнению;

3. Объект (скрытый) планировщика принимает актора и начинает его выполнять;

4. Актор выбирает сообщение из передней части очереди;

5. Актор изменяет внутреннее состояние, отправляет сообщения другим участникам;

6. Актор незапланирован.

#### Чтобы добиться такого поведения, у акторов есть:

* **Почтовый ящик** (очередь, в которой заканчиваются сообщения);

* **Поведение** (состояние актора, внутренние переменные и т. Д.);

* **Сообщения** (фрагменты данных, представляющие сигнал, аналогичные вызовам методов и их параметрам);

* **Условия исполнения** (механизм, который принимает участников, у которых есть сообщения, которые реагируют на код обработки сообщений и ссылаются на него);

* **Адрес** (подробнее об этом позже).

Сообщения отправляются в почтовые ящики. Поведение актора описывает, как актор реагирует на сообщения (например, отправляет 
больше сообщений и / или меняет состояние). Среда выполнения объединяет пул потоков, чтобы полностью управлять всеми этими действиями.

Это очень простая модель, и она решает проблемы, перечисленные ранее:

* Инкапсуляция сохраняется путем отделения исполнения от сигнализации (передача вызова метода, передача сообщений не выполняется);

* Нет необходимости в замках. Изменение внутреннего состояния актора возможно только через сообщения, которые обрабатываются 
по одному за раз, исключая гонку при попытке сохранить инварианты;

* В любом месте нет замков, и отправители не блокируются. Миллионы акторов могут быть эффективно спланированы на дюжине 
потоков, которые в полной мере смогут использовать современные процессоры. Делегирование задач - это естественный режим 
работы для участников;

* Состояние участников является локальным и не разделяется, изменения и данные распространяются через сообщения, которые 
сопоставляются с тем, как работает современная иерархия памяти. Во многих случаях это означает передачу только строк кэша, 
содержащих данные в сообщении, при сохранении локального состояния и данных, кэшированных в исходном ядре. Эта же модель 
точно соответствует удаленной связи, когда состояние хранится в ОЗУ машин, а изменения/данные распространяются по сети 
в виде пакетов.

### Акторы корректно обрабатывают ошибки

Поскольку у нас больше нет общего стека вызовов между участниками, которые отправляют сообщения друг другу, нам нужно 
обрабатывать ситуации с ошибками по-разному. Мы должны учитывать два типа ошибок:

1. Первый случай - когда делегированная задача на целевом акторе завершилась неудачно из-за ошибки в задаче (как правило,
 проблема проверки, например несуществующий идентификатор пользователя). В этом случае служба, инкапсулированная целевым 
 игроком, остается нетронутой, а сама задача ошибочна. **Актор службы должен ответить отправителю с сообщением, представляющим 
 случай ошибки**. Здесь нет ничего особенного, ошибки являются частью домена и, следовательно, становятся обычными сообщениями.

2. Второй случай - когда сама служба сталкивается с внутренней ошибкой. Акка предусматривает, что все акторы организованы 
в древовидную иерархию, т.е. Актор, создающий другого актора, становится родителем этого нового актора. Это очень похоже 
на то, как операционные системы организуют процессы в дерево. Как и в случае процессов, когда актор терпит неудачу, 
его родительский актор уведомляется и может реагировать на сбой. Кроме того, **если родительский актор остановлен, все его 
дети также рекурсивно останавливаются**. Эта услуга называется надзором, и она занимает центральное место в Акке.

![alt text](https://github.com/steklopod/akka/blob/akka_starter/src/main/resources/images/how-the-actor-model-meets-the-needs-of-modern-distributed-systems/actor_tree_supervision.png "actor_tree_supervision")

[=> далее: Обзор библиотек и модулей Akka](https://github.com/steklopod/akka/blob/akka_starter/src/main/resources/readmes/overview-of-akka-libraries-and-modules.md)

_Если этот проект окажется полезным тебе - нажми на кнопочку **`★`** в правом верхнем углу._

[<= содержание](https://github.com/steklopod/akka/blob/akka_starter/readme.md)