## Обобщения и Интерфейсы

Еще одна из возможностей TypeScript - это _обобщения_ (generics).

Идея в том, что мы хотели бы создавать достаточно гибкие компоненты "по контракту", при этом не создавая отдельный тип для каждого. Суть в том, что вы по прежнему можете исползовать все преимущнства строгой типизации, но при этом не завязываясь на конкретный тип.


Аналогичный концепт - [Утиная типизация(duck typing)](http://ru.wikipedia.org/wiki/Утиная_типизация), когда тип определяется на лету по предоставленному объекту, вместо изначального задания типа перед присвоением переменной.

I> Такие языки как Java и C# также используют обобщения

Посмотрим на пример функции:

{lang='JavaScript'}
    function size(a: string): number {
      return a.length;
    }

В этом случае мы ожидаем строку, получаем ее длинну и возвращаем значение числового типа.

Но что если бы мы захотели сделать эту функцию немного более гибкой, например: чтобы она также работала с массивами?

Да, мы можем исползовать тип `any`, который мы рассматривали выше, и сделать функцию так:

{lang='JavaScript'}
    function size(a: any): number {
      return a.length;
    }

И, конечно, она работает. Однако, в данном случае, мы теряем важную информацию, которая была в предыдущей реализации: о том, что мы получаем строку как входящий параметр.

Исходя из этой сигнатуры функции, ничего не мешает нам передать параметр, который даже не содержит свойство `length`. Мы можем убедиться в этом передав число.

Если у вас еще не открыт `tsun`, откройте и наберите следующий код:

{lang='JavaScript'}
    function size(a: any): number {
      return a.length;
    }
    size('Felipe');
    size(['Felipe']);
    size(12);

Вы можете убедиться, что первые два случая работают, но последний возвращает undefined, так как число 12 не содержит свойства `length`.

Может быть даже хуже, если мы вызовем какой-то метод, что повлечет за собой ошибку выполнения.

Давайте исследуем новую функцию, которая ожидает получить параметром объект содержащий метод `.getDay()`. Наберите в TSUN этот код:

{lang='JavaScript'}
    function dayOf(d: any): number {
      return d.getDay()
    }
    var d = new Date();
    var n = 12;
    dayOf(d);
    dayOf(n);

И вы увидете следующее:

{lang='JavaScript'}
    >     function dayOf(d: any): number {
    ..      return d.getDay()
    ..    }
    undefined
    >     var d = new Date();
    undefined
    >     var n = 12;
    undefined
    >     dayOf(d);
    5
    >     dayOf(n);
    TypeError: Object 12 has no method 'getDay'
        at dayOf (evalmachine.<anonymous>:2:14)
        at evalmachine.<anonymous>:1:7
        at startEvaluate (/usr/local/lib/node_modules/tsun/bin/tsun.js:369:25)
        at replLoop (/usr/local/lib/node_modules/tsun/bin/tsun.js:396:9)
        at /usr/local/lib/node_modules/tsun/bin/tsun.js:465:9
        at Interface._onLine (readline.js:200:5)
        at Interface._line (readline.js:531:8)
        at Interface._ttyWrite (readline.js:760:14)
        at ReadStream.onkeypress (readline.js:99:10)
        at ReadStream.EventEmitter.emit (events.js:98:17)

Это то место, где мы должны задуматься об обобщениях(Generics). Мы должны каким-то способом указать на то, что в объекте параметре функции `dayOf` должен быть метод `getDay`.

#### Интерфейсы

Работая с дженериками вы постепенно приходите к пониманию еще одной возможности TypeScript - использования интерфейсов.

Опять таки, идея позаимствована из ООП языков, таких как Java, C# и других.

Фактически, когда вы создаете интерфейс, вы пишете контракт, которому должны будут следовать все кто подписал этот контракт.

В случае с функцией `dayOf` нам нужен контракт, чтобы показать что входящий параметр должен реализовать этот метод.

Для этого мы создадим следующий интерфейс:

{lang='JavaScript'}
    interface Datelike {
      getDay(): number;
    }

Тем самым мы обязуем все объекты типа _Date-like_ реализовать метод `getDay`.

TypeScript хорош тем, что он не требует строгой явного задания интерфейса для объекта, для того чтобы его исползовать. Например: встроенный тип `Date` не реализует наш новый интерфейс, но при этом он _соотвествует его требованиям_, так как он реализует метод `getDay`.

Давайте перепишем наш метод используя обобщения:

{lang='JavaScript'}
    function dayOf<T extends Datelike>(d: T): number {
      return d.getDay();
    }

Что мы знаем о этой функции:
- `dayOf` ожидает получить тип, который наследуется от нашего интерфейса
- параметр `d` должен быть типа, который мы объявили для нашей функции как обобщение

Давайте попробуем использовать функцию с теме же значениями. Перед запуском примера в TSUN запустите команду `:clear`, чтобы очистить весь введенный ранее код и избежать конфликтов.

Введите следующий код:

{lang='JavaScript'}
    :clear
    interface Datelike {
      getDay(): number;
    }
    function dayOf<T extends Datelike>(d: T): number {
      return d.getDay();
    }
    var d = new Date();
    var n = 12;
    dayOf(d);
    dayOf(n);

и вы должны получить следующее:

{lang='JavaScript'}
    > :clear
    > interface Datelike {
    ..  getDay(): number;
    ..}
    undefined
    > function dayOf<T extends Datelike>(d: T): number {
    ..  return d.getDay();
    ..}
    undefined
    > var d = new Date();
    undefined
    > var n = 12;
    undefined
    > dayOf(d);
    5
    > dayOf(n);
    Argument of type 'number' is not assignable to parameter of type 'Datelike'.
      Property 'getDay' is missing in type 'Number'.

Но теперь мы просто получем другую ошибку, верно? Вообще-то, не совсем. Эта ошибка - ошибка компиляции, но не ошибка во время выполнения.

Чтобы это лучше понять, перенесите весь код в файл с названием `datelike.ts`:

{lang='JavaScript'}
    interface Datelike {
      getDay(): number;
    }
    function dayOf<T extends Datelike>(d: T): number {
      return d.getDay();
    }
    var d = new Date();
    var n = 12;
    dayOf(d);
    dayOf(n);

И попробуйте скомпилировать используя  `tsc`:

{lang='shell'}
    $ tsc datelike.ts
    datelike.ts(10,11): error TS2345: Argument of type 'number' is not assignable to parameter of type 'Datelike'.
      Property 'getDay' is missing in type 'Number'.

В этом случае `datelike.js` файл даже не будет создан, и мы уже будем знать об этой ошибке еще даже не запуская браузер.

Если вы хотите написать свой класс с методом `getDay()`, чтобы использовать его потом в функции, это тоже легко сделать:

{lang='JavaScript'}
    class MyDate {
      getDay(): number {
        return 12;
      }
    }

Давайте посмотрим как это работает с нашим методом `dayOf`:

{lang='JavaScript'}
    > var x: MyDate = new MyDate();
    undefined
    > dayOf(x)
    12

Чтобы сделать класс `MyDate` более прозрачным и защитить от изменений в будующем, мы можем сделать:

{lang='JavaScript'}
    class MyDate implements Datelike {
      getDay(): number {
        return 12;
      }
    }

В данном случае при случаном удаленни метода `getDay` мы сразу же получим следующую ошибку:

{lang='JavaScript'}
    > interface Datelike {
    ..  getDay(): number;
    ..}
    undefined
    > class MyDate implements Datelike {
    ..}
    Class 'MyDate' incorrectly implements interface 'Datelike'.
      Property 'getDay' is missing in type 'MyDate'.



## Использование сторонних типов

Пример angular2.d.ts (файл определений).
--- http://www.typescriptlang.org/Handbook#writing-dts-files
