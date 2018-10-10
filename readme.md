## Распределенные системы. Akka.

### Модель акторов

Последние десятилетия показали, что распределенные системы и многозадачность в классическом императивном подходе - это 
очень сложная задача. Возможно, именно это дало начало очередному витку бума функциональных языков/фич и неизменяемых структур данных.

Один из подходов к решению проблем построения распределенных систем предлагает [модель акторов](https://ru.wikipedia.org/wiki/модель_акторов)
 (actor model) пришедшая из языка `Erlang` разработанного компанией Ericsson для телекоммуникаций. Кроме того, на языке Erlang построен Whatsapp.

Вкратце описать модель акторов можно как систему состоящую из одновременно функционирующих объектов, коммуникация между 
которыми происходит исключительно обменом сообщениями. В каком-то роде можно рассматривать ее как самую чистую ООП-модель, 
где каждый объект функционирует одновременно с остальными.

**Актор** является вычислительной сущностью, которая в ответ на полученное сообщение может одновременно:

* отправить конечное число сообщений другим акторам;

* создать конечное число новых акторов;

* выбрать тип поведения, которое будет использоваться для следующего сообщения в свой адрес.

В скала модель акторов представлена библиотекой [akka](http://akka.io/).


[Пример 1 - CofeeShop](https://github.com/steklopod/akka/blob/storage_app_starter/src/main/resources/cofeeshop.md)  

[Пример 2 - StorageApp](https://github.com/steklopod/akka/blob/storage_app_starter/src/main/resources/storage_example.md)  