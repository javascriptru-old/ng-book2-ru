# Встроенные директивы {#встроенные_директивы}

## Введение

В Angular предусмотрен ряд встроенных _директив_, фактически являющиеся атрибутами, которые мы добавляем к нашим HTML элементам для обеспечения динамического поведения. В этой главе мы рассмотрим все встроенные директивы и покажем вам, как ими пользоваться. 

К концу данной главы вы научитесь пользоваться основными встроенными директивами, которые нам предлагает Angular.

T> **Как читать эту главу**
T>
T> Вместо поэтапного создания приложения, в этой главе мы проведем экскурс по
T> встроенным директивам Angular. Поскольку мы находимся только в самом начале книги, то мы не будем вдаваться во
T> все тонкости, а сосредоточим наше внимание на большом количестве примеров.
T>
T> Запомните: в любой момент вы можете обратиться к примерам кода для данной главы.
T>
T> Если вы хотите запустить примеры из этой главы, то перейдите в папку `code/built-in-directives` и запустите их:
T>
T>        npm install
T>        npm start
T>
T> И затем откройте [http://localhost:4200](http://localhost:4200) в вашем браузере.

## `NgIf`

Директива `ngIf` используется для отображения или сокрытия элемента в зависимости от состояния. Состояние определяется на основе результата вычисления _выражения_ которое вы передаете в директиву.

Если результат вычисления выражения возвращает значение false, то элемент будет удален из DOM.

Вот несколько примеров:

```html
<div *ngIf="false"></div>        <!-- будет скрыт -->
<div *ngIf="a > b"></div>        <!-- будет отображен, если a > b -->
<div *ngIf="str == 'yes'"></div> <!-- будет отображен, если str является строкой "yes" -->
<div *ngIf="myFunc()"></div>     <!-- будет отображен, если функция myFunc возвратит значение true -->
```

I> **Примечание для пользователей AngularJS 1.x**
I>
I> Если вы ранее использовали AngularJS 1.x, то вероятно уже знакомы с директивой `ngIf`.
I> В Angular 4 вы можете рассматривать ее как непосредственную замену.
I>
I> С другой стороны, Angular 4 не предлагает встроенную альтернативу `ng-show`.
I> Поэтому, если вашей целью является изменение свойства видимости CSS элемента,
I> то вам следует использовать директивы `ngStyle` или `class`,
I> описанные далее в этой главе.

## `NgSwitch`

Иногда нам нужно отобразить различные элементы в зависимости от заданного условия.

Когда вы столкнетесь с этой ситуацией, вы можете многократно использовать `ngIf` , как это реализовано в примере ниже:

```javascript
<div class="container">
  <div *ngIf="myVar == 'A'">Var is A</div>
  <div *ngIf="myVar == 'B'">Var is B</div>
  <div *ngIf="myVar != 'A' && myVar != 'B'">Var is something else</div>
</div>
```

Следует отметить, что сценарий, в котором `myVar` не соответствует значениям `A` и `B`, является достаточно полным и пытается воспроизвести работу конструкции `else`.

Чтобы продемонстрировать возрастающую сложность добавления нового условия, попробуем обработать новое условие со значением `C`.

Для этого нам нужно будет добавить не только новый элемент с директивой `ngIf`, но а также изменить содержимое последнего условия:

```javascript
<div class="container">
  <div *ngIf="myVar == 'A'">Var is A</div>
  <div *ngIf="myVar == 'B'">Var is B</div>
  <div *ngIf="myVar == 'C'">Var is C</div>
  <div *ngIf="myVar != 'A' && myVar != 'B' && myVar != 'C'">Var is something else</div>
</div>
```

Для подобных случаев в Angular предусмотрена директива `ngSwitch`.

Если вы знакомы с конструкцией `switch` то у вас не должно возникнуть вопросов.

Идея, лежащая в основе этой директивы, заключается в принятии одного единственного результата вычисления выражения и последующем отображении вложенных элементов, которые ему соответствуют.

Как только мы получим результат, мы можем:

- Описать все известные варианты, используя директиву `ngSwitchCase`
- Обработать все другие неизвестные случаи используя директиву `ngSwitchDefault`

Давайте перепишем наш пример с использованием новых директив:

```javascript
<div class="container" [ngSwitch]="myVar">
  <div *ngSwitchCase="'A'">Var is A</div>
  <div *ngSwitchCase="'B'">Var is B</div>
  <div *ngSwitchDefault>Var is something else</div>
</div>
```

Далее, если мы захотим обработать новое значение `C`, мы должны будем добавить еще одну строку кода:

```javascript
<div class="container" [ngSwitch]="myVar">
  <div *ngSwitchCase="'A'">Var is A</div>
  <div *ngSwitchCase="'B'">Var is B</div>
  <div *ngSwitchCase="'C'">Var is C</div>
  <div *ngSwitchDefault>Var is something else</div>
</div>
```

В результате, нам не требуется изменять условие по умолчанию.

Использование элемента `ngSwitchDefault` является необязательным. Если мы не укажем данный элемент, то в случае, когда переменная `myVar` не будет соответствовать ни одному из ожидаемых значений, ничего не будет отображено.

Вы также можете объявить одинаковое `*ngSwitchCase` значение для нескольких разных элементов. Нет никаких ограничений. Например:

{lang=html}
<<[code/built-in-directives/src/app/ng-switch-example/ng-switch-example.component.html](code/built-in-directives/src/app/ng-switch-example/ng-switch-example.component.html)

В приведенном выше примере, когда значение `choice` равно `2`, будут отображены второй и пятый элементы списка `li`.

## `NgStyle`

С помощью директивы `NgStyle` вы можете установить CSS свойства для DOM элементов с помощью Angular выражений.

Самый простой способ использования данной директивы выглядит так: `[style.<cssproperty>]="value"`. Например:

{lang=html,crop-start-line=5,crop-end-line=7}
<<[code/built-in-directives/src/app/ng-style-example/ng-style-example.component.html](code/built-in-directives/src/app/ng-style-example/ng-style-example.component.html)

Этот фрагмент использует директиву `NgStyle`, чтобы установить в качестве значения CSS свойства `background-color` строковое значение `'yellow'`.

Второй способ установления фиксированного значения заключается в использовании атрибута `NgStyle` и пары ключ-значение для каждого свойства, которое необходимо установить. Например:

{lang=html,crop-start-line=13,crop-end-line=15}
<<[code/built-in-directives/src/app/ng-style-example/ng-style-example.component.html](code/built-in-directives/src/app/ng-style-example/ng-style-example.component.html)

T> Обратите внимание, что в спецификации `ng-style` мы используем одинарные кавычки вокруг `background-color` но не вокруг `color`. Почему? Аргументом `ng-style` является JavaScript объект и `color` является допустимым ключом, без кавычек. В случае `background-color`, символ дефиса недопустим в качестве ключа объекта, если это не строка, поэтому мы вынуждены заключить его в кавычки.
T>
T> Рекомендуем оставлять без кавычек как можно большое количество ключей объекта и использовать их только там, где это действительно необходимо.

Здесь мы устанавливаем два свойства: `color` и `background-color`.

Но реальная сила директивы `NgStyle` состоит в использовании динамических значений.

В нашем примере, мы определяем для поля ввода с кнопкой установки свойств:

{lang=html,crop-start-line=56,crop-end-line=66}
<<[code/built-in-directives/src/app/ng-style-example/ng-style-example.component.html](code/built-in-directives/src/app/ng-style-example/ng-style-example.component.html)

И затем, используя их значения, зададим свойства CSS для трех элементов.

В первом случае мы устанавливаем размер шрифта на основе введенного значения:

{lang=html,crop-start-line=21,crop-end-line=25}
<<[code/built-in-directives/src/app/ng-style-example/ng-style-example.component.html](code/built-in-directives/src/app/ng-style-example/ng-style-example.component.html)

Важно отметить, что мы должня указывать единицы измерения там, где это уместно. Например, неправильно указывать значение `font-size` равным `12` - мы должны указывать значения с единицами измерения, такие как `12px` или `1.2em`. Angular предоставляет удобный синтаксис для указания единиц измерения: в данном случае мы используем синтаксис `[style.font-size.px]`.

Суффикс `.px` обозначает, что мы устанавливаем значение свойства `font-size` в пикселях. Вы можете легко заменить его на `[style.font-size.em]` , чтобы установить значение шрифта в единицах em или даже в процентах, используя `[style.font-size.%]`.

Два других элемента используют `#colorinput` , чтобы установить значение цвета текста и фона:

{lang=html,crop-start-line=33,crop-end-line=50}
<<[code/built-in-directives/src/app/ng-style-example/ng-style-example.component.html](code/built-in-directives/src/app/ng-style-example/ng-style-example.component.html)

Таким образом, когда мы нажимаем кнопку **Apply settings** , мы вызываем метод, который устанавливает новые значения:

{lang=javascript,crop-query=.apply}
<<[code/built-in-directives/src/app/ng-style-example/ng-style-example.component.ts](code/built-in-directives/src/app/ng-style-example/ng-style-example.component.ts)

И при этом цвет и размер шрифта будут назначены элементам с помощью директивы `NgStyle`.

## `NgClass`

Директива `NgClass` , представленная атрибутом `ngClass` в вашем HTML шаблоне, позволяет динамически устанавливать или изменять CSS классы DOM элемента.

Первый способ использования этой директивы - это передача литерала объекта. Ожидается, что ключи объекта будут использоваться в качестве имен классов, а значения ключей должны быть вида true/false, чтобы указывать, следует ли применять класс или нет.

Предположим, у нас есть CSS класс `bordered` , который добавляет элементам черную пунктирную обводку:

{lang=CSS,crop-start-line=8,crop-end-line=11}
<<[code/built-in-directives/src/styles.css](code/built-in-directives/src/styles.css)

Давайте добавим два `div` элемента: одному будет присвоен класс `bordered` (и поэтому будет иметь обводку), а второму нет:

{lang=html,crop-start-line=2,crop-end-line=3}
<<[code/built-in-directives/src/app/ng-class-example/ng-class-example.component.html](code/built-in-directives/src/app/ng-class-example/ng-class-example.component.html)

Как и ожидалось, именно так эти два `div` и будут отображены:

![Простое использование директивы класса](images/built_in_directives/css-class-directive-simple.png)

Конечно, гораздо лучше использовать директиву `NgClass` для динамического назначения класса.

Чтобы сделать его динамическим мы добавим переменную в качестве значения объекта, как показано ниже:

{lang=html,crop-start-line=5,crop-end-line=7}
<<[code/built-in-directives/src/app/ng-class-example/ng-class-example.component.html](code/built-in-directives/src/app/ng-class-example/ng-class-example.component.html)

Кроме того, мы можем определить объект `classesObj` в нашем компоненте:

{lang=javascript,crop-query=decorators(.NgClassExampleComponent)-context(.bordered,0,2)}
<<[code/built-in-directives/src/app/ng-class-example/ng-class-example.component.ts](code/built-in-directives/src/app/ng-class-example/ng-class-example.component.ts)

И использовать непосредственно сам объект:

{lang=html,crop-start-line=9,crop-end-line=11}
<<[code/built-in-directives/src/app/ng-class-example/ng-class-example.component.html](code/built-in-directives/src/app/ng-class-example/ng-class-example.component.html)

W> Опять же, будьте осторожны, когда у вас есть имя класса, который содержит дефисы, например, `bordered-box`. JavaScript требует, чтобы ключи объектов литералов с символом дефиса были в кавычках, как в примере ниже:
W>
W> ```javascript
W> <div [ngClass]="{'bordered-box': false}">...</div>
W> ```

Можно также использовать список классов, чтобы указать, какие классы должны быть добавлены к элементу. Для этого, мы можем либо передать их в литерале массива:

{lang=html,crop-start-line=31,crop-end-line=34}
<<[code/built-in-directives/src/app/ng-class-example/ng-class-example.component.html](code/built-in-directives/src/app/ng-class-example/ng-class-example.component.html)

Либо присвоить массив значений свойству нашего компонента:

```javascript
this.classList = ['blue', 'round'];
```

И передать его:

{lang=html,crop-start-line=36,crop-end-line=39}
<<[code/built-in-directives/src/app/ng-class-example/ng-class-example.component.html](code/built-in-directives/src/app/ng-class-example/ng-class-example.component.html)

В последнем примере, `[ngClass]` присваивает значения наряду с существующими значениями, назначенными HTML атрибутом `class`.

В результате набор классов элемента будет состоять из классов назначенных, как обычным HTML атрибутом `class`, так и полученных в результате вычисления директивы `[class]`.

В этом примере:

{lang=html,crop-start-line=31,crop-end-line=34}
<<[code/built-in-directives/src/app/ng-class-example/ng-class-example.component.html](code/built-in-directives/src/app/ng-class-example/ng-class-example.component.html)

Элемент получит все три класса: `base` из HTML атрибута `class`, а также `blue` и `round` из директивы `[class]`:

![Классы из атрибута и директивы](images/built_in_directives/css-class-multiple-class-sources.png)

## `NgFor`

Роль этой директивы заключается в том, чтобы **повторять DOM элемент** (или коллекцию DOM элементов) и передавать элемент массива при каждой итерации.

Синтаксис: `*ngFor="let item of items"`.

- Синтаксис `let item` определяет переменную, которую получает каждый элемент массива `items`;
- `items` - это коллекция элементов вашего контролера.

Чтобы показать это, мы можем взглянуть на пример кода. Объявим массив с названиями городов в нашем компоненте контролере:

```javascript
this.cities = ['Miami', 'Sao Paulo', 'New York'];
```

И тогда, в нашем шаблоне мы можем получить следующий HTML фрагмент:

{lang=html,crop-start-line=1,crop-end-line=7}
<<[code/built-in-directives/src/app/ng-for-example/ng-for-example.component.html](code/built-in-directives/src/app/ng-for-example/ng-for-example.component.html)

Как и следовало ожидать, он будет отображать каждый город внутри `div`:

![Результат использования директивы ngFor](images/built_in_directives/ng-for-result1.png)

Мы также можем перебрать массив таких объектов:

{lang=javascript,crop-query=window(.ngOnInit .people, 0, 4)}
<<[code/built-in-directives/src/app/ng-for-example/ng-for-example.component.ts](code/built-in-directives/src/app/ng-for-example/ng-for-example.component.ts)

И затем отобразить таблицу на основе каждой строки данных:

{lang=html,crop-start-line=9,crop-end-line=26}
<<[code/built-in-directives/src/app/ng-for-example/ng-for-example.component.html](code/built-in-directives/src/app/ng-for-example/ng-for-example.component.html)

Получили следующий результат:

![Отображение массива объектов](images/built_in_directives/ng-for-objects.png)

Мы также можем работать со вложенными массивами. Если бы мы хотели получить ту же таблицу, что и выше, разбитую по городам, мы с легкостью могли бы объявить новый массив объектов:

{lang=javascript,crop-query=window(.ngOnInit .peopleByCity, 0, 16)}
<<[code/built-in-directives/src/app/ng-for-example/ng-for-example.component.ts](code/built-in-directives/src/app/ng-for-example/ng-for-example.component.ts)

И затем мы можем использовать директиву `NgFor` для отображения заголовка `h2` для каждого города:

{lang=html,crop-start-line=32,crop-end-line=33}
<<[code/built-in-directives/src/app/ng-for-example/ng-for-example.component.html](code/built-in-directives/src/app/ng-for-example/ng-for-example.component.html)

И использовать вложенную директиву для перебора людей по заданному городу:

{lang=html,crop-start-line=13,crop-end-line=26}
<<[code/built-in-directives/src/app/ng-for-example/ng-for-example.component.html](code/built-in-directives/src/app/ng-for-example/ng-for-example.component.html)

Результат представлен в следующем коде шаблона:

{lang=html,crop-start-line=28,crop-end-line=47}
<<[code/built-in-directives/src/app/ng-for-example/ng-for-example.component.html](code/built-in-directives/src/app/ng-for-example/ng-for-example.component.html)

Это позволит отобразить по одной таблице для каждого города:

![Отображение вложенных массивов](images/built_in_directives/ng-for-nested-arrays.png)

### Получение индекса

Иногда при итерации массива нам необходим индекс каждого элемента.

Мы можем получить индекс, добавив синтаксис `let idx = index` к значению нашей директивы `ngFor`, разделенной точкой с запятой. Когда мы это сделаем, ng2 присвоит текущий индекс переменной, которую мы объявим (в данном случае это переменная `idx`).

W> Обратите внимание, что, как и в JavaScript, индекс всегда равен нулю. Поэтому индекс первого элемента равен 0, 1 для второго и так далее...

Внесем некоторые изменения в наш первый пример, добавив фрагмент `let num = index`, как показано ниже:

{lang=html,crop-start-line=53,crop-end-line=55}
<<[code/built-in-directives/src/app/ng-for-example/ng-for-example.component.html](code/built-in-directives/src/app/ng-for-example/ng-for-example.component.html)

Это добавит позицию города перед названием:

![Использование индекса](images/built_in_directives/ng-for-using-index.png)

## `NgNonBindable`

Мы используем директиву `ngNonBindable` когда хотим сообщить Angular **не** компилировать какой-либо фрагмент нашей страницы.

Допустим, нам нужно вывести текст `{{ content }}` в нашем шаблоне. Как правило, этот текст _будет соответствовать_ значению переменной `content` поскольку мы используем синтаксис шаблона `{{` `}}`.

Как же нам тогда отобразить точный текст `{{ content }}`? Для этого используем директиву `ngNonBindable`.

Допустим, мы хотим получить элемент `div`, который отображает значение переменной `content` и сразу после значения переменной хотим вывести _<- это то, что отображает {{ content }}_ .

Пример, который позволяет это реализовать:

{lang=html}
<<[code/built-in-directives/src/app/ng-non-bindable-example/ng-non-bindable-example.component.html](code/built-in-directives/src/app/ng-non-bindable-example/ng-non-bindable-example.component.html)

А с атрибутом `ngNonBindable` ng2 не будет компилировать внутри второго элемента `span`, просто его пропустив:

![Результат использования ngNonBindable](images/built_in_directives/ng-non-bindable.png)

## Заключение

В Angular предусмотрено всего несколько основных директив, но в результате их совместного использования мы можем создавать мощниые и динамичные приложения. Тем не менее, все эти директивы помогают нам **выводить** динамические данные, но все еще не позволяют **взаимодействовать с пользователем**.

В следующей главе мы узнаем, как разрешить нашим **пользователям вводить данные с помощью форм**.
