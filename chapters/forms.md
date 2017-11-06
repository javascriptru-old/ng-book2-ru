# Формы в Angular {#forms}

## Такие важные и сложные формы

Вероятнее всего, формы являются наиболее важной частью веб-приложения.

Несмотря на большое количество нажатий по ссылкам и перемещений курсора, все же самую большую и важную часть данных от пользователей мы получаем через _формы_.

На первый взгляд, формы кажутся очень простыми: вы создаете тэг `input`, пользователь заполняет форму и отправляет ее, нажав кнопку submit. Неужели сложно?

Оказывается, формы могут быть очень сложными. И вот почему:

* Формы могут использоваться для изменения данных как на странице, так и на сервере
* Очень часто изменения должны быть отображены в другой части страницы
* Пользователи не ограничены в том, что они вводят. Следовательно, у нас должна быть возможность проверять эти данные
* Пользовательский интерфейс должен: быть предсказуемым и четко обрабатывать ошибки, в случае их появления
* Зависимые поля могут иметь сложную логику
* У нас должна быть возможность тестировать наши формы, не прибегая к DOM селекторам

К счастью, в Angular есть инструменты, которые помогут нам решить все эти проблемы.

* **`FormControl`** позволяет нам инкапсулировать данные формы и предоставляет объекты для работы с ними
* **`Validator`** позволяет нам проверять данные любым способом, которым мы захотим
* **`Observers`** позволяет нам отслеживать изменения в формах и реагировать на эти изменения соответствующим образом

В этой главе, шаг за шагом, мы рассмотрим, как создаются формы. А начнем мы с простых форм и дальше будем усложнять их логику.

## `FormControl` и `FormGroup`

Два основных объекта в Angular формах - это `FormControl` и `FormGroup`.

### `FormControl`

`FormControl` представляет собой одно поле ввода - это наименьшая из составных частей формы в Angular.

`FormControl` инкапсулирует данные формы и состояния, сообщающие о статусе проверки, изменениях или наличии ошибок.

Например, вот так мы можем использовать `FormControl` в TypeScript:

```javascript
// создаем новый FormControl со значением "Nate"
let nameControl = new FormControl("Nate");

let name = nameControl.value; // -> Nate

// теперь мы можем запросить информацию о состояниях:
nameControl.errors // -> StringMap<string, any> of errors
nameControl.dirty  // -> false
nameControl.valid  // -> true
// и т.д.
```

Для создания формы мы создаем `FormControl` (или несколько `FormControl`), а затем присоединяем к нему метаданные и логику.

Как и многое другое в Angular, мы добавляем класс (в данном случае `FormControl`) в DOM с помощью атрибута (в данном случае `formControl`). В итоге мы получим следующее:

```html
<!-- часть большой формы -->
<input type="text" [formControl]="name" />
```

Все это создаст новый объект `FormControl` в контексте нашей формы. Далее мы подробно разберем, как все это работает.

### `FormGroup`

Многие формы состоят из нескольких полей, поэтому нам нужен способ управления сразу несколькими `FormControl`. Если мы хотим проверить правильность нашей формы, то перебор в массиве всех `FormControl` с проверкой каждого элемента будет выглядеть достаточно громоздким и сложным. Но `FormGroup` решает эту проблему путем создания интерфейса обертки вокруг коллекции `FormControl`.

Создадим `FormGroup`:

```javascript
let personInfo = new FormGroup({
    firstName: new FormControl("Nate"),
    lastName: new FormControl("Murray"),
    zip: new FormControl("90210")
})
```

У `FormGroup` и `FormControl` есть общий предок ([`AbstractControl`](https://angular.io/docs/ts/latest/api/forms/index/AbstractControl-class.html)). Это означает, что мы можем проверить `status` или `value` у `personInfo` так же легко, как и у одного `FormControl`:

```javascript
personInfo.value; // -> {
//   firstName: "Nate",
//   lastName: "Murray",
//   zip: "90210"
// }

// теперь мы можем запросить информацию о состоянии группы,
// значение которой зависит от значений дочерних 'FormControl':
personInfo.errors // -> StringMap<string, any> of errors
personInfo.dirty  // -> false
personInfo.valid  // -> true
// и т.д.
```

Обратите внимание, когда если мы ппытаемся получить `value` из `FormGroup`, то мы получим **объект** вида ключ-значение. Согласитесь, очень удобно получать весь список значений нашей формы, не прибегая к перебору всех `FormControl`.

## Наша первая форма

Существует множество нюансов при создании формы, и некоторые важные из них мы еще не рассматривали. Поэтому, давайте перейдем к примеру и разберем его по частям.

%% BEGIN_BOOK

I> Все примеры кода для данной главы вы можете найти в папке `forms/`

%% END_BOOK

%% BLOG{full_code_listing}

Вот как выглядит форма, которую мы с вами собираемся создать:

![Пример простой формы Sku](images/forms/demo_form_sku_simple.png)

В нашем воображаемом приложении мы создаем сайт электронной коммерции, на котором мы будем размещать информацию о продаваемых товарах. В этом приложении нам нужно будет сохранять SKU товара. Поэтому, давайте создадим простую форму, в которой SKU будет единственным полем для ввода данных.

I> SKU - это артикул товара. Это уникальный id продукта, который мы будем отслеживать. В будущем, когда мы будем говорить о SKU, мы будем подразумевать ID продукта.

Получилась очень простая форма: всего одно поле ввода для `sku` (с меткой label) и кнопка submit.

Давайте превратим эту форму в компонент. Как вы помните, для создания компонента нам необходимо выполнить три шага:

* Сконфигурировать декоратор `@Component()`
* Создать шаблон
* Добавить пользовательские функции в классе определения компонента

### Загрузка `FormsModule`

Чтобы начать использовать библиотеку для создания форм, нам нужно сначала убедиться в том, что мы ее импортировали в `NgModule`.

Существует два способа работы с формами в Angular. В этой главе мы разберем каждый из них: с использованием `FormsModule` и `ReactiveFormsModule`. Поскольку мы будем использовать оба варианта, мы импортируем их в наш модуль. Для этого в `app.ts` мы сделаем следующее:

{lang=javascript}
    import {
      FormsModule,
      ReactiveFormsModule
    } from '@angular/forms';

    // дальше вниз...

    @NgModule({
      declarations: [
        FormsDemoApp,
        DemoFormSkuComponent,
        // ... наше объявление
      ],
      imports: [
        BrowserModule,
        FormsModule,         // <-- добавили это
        ReactiveFormsModule  // <-- добавили это
      ],
      bootstrap: [ FormsDemoApp ]
    })
    class FormsDemoAppModule {}


Это гарантирует, что мы сможем использовать директивы форм в наших шаблонах. Забегая вперед, `FormsModule` дает нам директивы:

- `ngModel` и
- `NgForm`

В то время как `ReactiveFormsModule` дает нам директивы:

- `formControl` и
- `ngFormGroup`

... и еще. Мы не говорили о том, как использовать эти директивы, и что они делают. Но мы скоро к этому вернемся. На данный момент, нам просто нужно запомнить, что импортируя `FormsModule` и `ReactiveFormsModule` в наш `NgModule`, мы можем использовать любую директиву из этого списка в нашем шаблоне или внедрять в наши компоненты.

### Простая форма SKU: декоратор @Component 

Теперь создадим компонент:

{lang=javascript,crop-query=1-.templateUrl}
<<[code/forms/src/app/demo-form-sku/demo-form-sku.component.ts](code/forms/src/app/demo-form-sku/demo-form-sku.component.ts)

Определим `selector` `app-demo-form-sku`. Если вы помните, `selector` сообщает Angular, что элементы связаны с этим компонентом. В этом случае мы можем использовать этот компонент с помощью тега `app-demo-form-sku`. Например:

{lang="html"}
    <app-demo-form-sku></app-demo-form-sku>

### Простая форма SKU: `template`

Давайте посмотрим на наш шаблон:

{lang=html}
<<[code/forms/src/app/demo-form-sku/demo-form-sku.component.html](code/forms/src/app/demo-form-sku/demo-form-sku.component.html)

#### `form` и `NgForm`

Теперь все стало интереснее: потому что мы импортировали `FormsModule`, который делает для нас доступным `NgForm`. Запомните, всякий раз, когда мы делаем директивы доступными для нашего шаблона, они **привязываются к любому элементу, который имеет соответствующий `selector`**.

`NgForm` удобен, но **неочевиден**: он вклюачет тэг `form` в его селектор (вместо того, чтобы явным образом добавить `ngForm`, как атрибут). Это означает, что если вы импортируете `FormsModule`, то `NgForm` будет *автоматически* добавлен ко всем формам с тэгом `<form>`, которые есть в вашем шаблоне. Это действительно удобно, но при этом запутанно, т.к. все происходит неочевидно.

Вот две важные функции, которые нам дает`NgForm`:

1. `FormGroup` названный `ngForm`
2. **`(ngSubmit)`** вывод

Вы можете заметить, что мы используем обе в теге `<form>` в нашем шаблоне:

{lang=html,crop-start-line=3,crop-end-line=4}
<<[code/forms/src/app/demo-form-sku/demo-form-sku.component.html](code/forms/src/app/demo-form-sku/demo-form-sku.component.html)

Изначально у нас есть `#f="ngForm"`. Синтаксис `#v=thing` говорит, что мы хотим создать локальную переменную для этого шаблоа.

Создадим alias для `ngForm` относящийся к переменной `#f` для этого шаблона. Откуда появился `ngForm`? Из директивы `NgForm`.

К какому типу объекта относится `ngForm`? Он относится к `FormGroup`. Значит мы можем использовать `f` как `FormGroup` в нашем шаблоне. Это как раз то, что мы делаем при `(ngSubmit)`.

W> Внимательные читатели могли заметить, что мы говорили, что `NgForm` автоматически привязывается к тэгам `<form>` (из-за селектора `NgForm`). Это означает, что нам не нужно добавлять атрибут `ngForm`, чтобы использовать `NgForm`. Но мы все же помещаем `ngForm` в качестве атрибута тэга. Неужели опечатка?
W>
W> Нет, это не опечатка. Если бы `ngForm` был _значением_ атрибута, то тем самым мы бы сообщили Angular, что мы хотим использовать `NgForm` для этого атрибута. В этом случае мы используем `ngForm` как _атрибут_, когда мы назначем _ссылку_. То есть, мы говорим, что значение выражения `ngForm` должно быть присвоено локальной переменной шаблона `f`.
W>
W> `ngForm` уже находится в этом элементе и вы можете считать, что мы "экспортируем" `FormGroup`, чтобы иметь возможность сослаться на нее в другом месте нашего шаблона.

Далее мы связываем событие `ngSubmit` нашей формы с помощью синтаксиса: `(ngSubmit)="onSubmit(f.value)"`.

* `(ngSubmit)` - поступает из `NgForm`
* `onSubmit()` - будет реализован в нашем классе определения компонента (см. ниже)
* `f.value` - `f` - это `FormGroup`, как было указано выше. И `.value` вернет пары ключ/значение этой `FormGroup`

Если обобщить, то все это означает: "когда я отправляю форму, вызов `onSubmit` в экземпляре компонента передает значение формы в качестве аргументов".

#### `input` и `NgModel`

В тэге `input` есть некоторые вещи, на которые мы должны обратить внимание, прежде чем заговорим о `NgModel`:

{lang=html,crop-start-line=3,crop-end-line=14}
<<[code/forms/src/app/demo-form-sku/demo-form-sku.component.html](code/forms/src/app/demo-form-sku/demo-form-sku.component.html)

* `class="ui form"` и `class="field"` -  эти классы необязательны. Они из [CSS framework Semantic UI](http://semantic-ui.com/). Мы добавили их в некоторые из наших примеров для придания хорошего вида. Они не являются частью Angular.
* Атрибут `label` `for`" и атрибут `input` `id`" должны совпадать [согласно стандарту W3C](http://www.w3.org/TR/WCAG20-TECHS/H44.html)
* Мы устанавили в `placeholder` значение "SKU" только в качестве подсказки для пользователя.

Директива `NgModel` определяет `selector` `ngModel`. Это значит, что мы можем прикрепить его к тэгу `input`, добавив этот атрибут: `ngModel="whatever"`. В этом случае мы определяем `ngModel` без значения атрибута.

Существует несколько способов указать `ngModel` в шаблонах, и это один из них. Когда мы используем `ngModel` без значения атрибута, мы определяем:

1. _одностороннюю_ связь данных
2. мы создаем `FormControl` в этой форме с именем `sku`

`NgModel` **создает новый `FormControl`**, который **автоматически добавляется** в родительский `FormGroup` (в данном случае  в форму) и затем привязывает элемент DOM к новому `FormControl`. То есть, он устанавливает связь между тэгом `input` в нашем шаблоне и `FormControl`, как следствие асспоциация совпала с именем, в данном случае `"sku"`.

I> **`NgModel` и `ngModel`**: в чем разница? Обычно, когда мы используем PascalCase, например, `NgModel`, мы указываем _class_ и ссылаемся на объект, который определен в коде. В случае (CamelCase), например, `ngModel`, поступающий из `selector` директивы, он используется только в DOM / шаблоне.
I>
I> Также стоит отметить, что `NgModel` и `FormControl` являются отдельными объектами. `NgModel` - это _директива_, которую вы используете в вашем шаблоне, в то время как `FormControl` - объект, используемый для представления данных и проверок в форме.

T> Иногда мы хотим сделать _двустороннюю_ связь с помощью `ngModel`, как это реализовано в Angular 1. Мы рассмотрим, как это сделать в конце этой главы.

### Простая форма SKU: класс, определяющий компонент

Теперь давайте посмотрим на наше определение класса:

{lang=javascript,crop-query=.DemoFormSkuComponent}
<<[code/forms/src/app/demo-form-sku/demo-form-sku.component.ts](code/forms/src/app/demo-form-sku/demo-form-sku.component.ts)

Здесь наш класс определяет одну функцию: `onSubmit`. Это функция, которую мы вызываем, когда отправляем нашу форму. Сейчас мы просто выводим в консоль значение, которое передаем.

### Давайте попробуем!

Соединим все вместе. Вот как будет выглядеть наш код:

{lang=javascript}
<<[code/forms/src/app/demo-form-sku/demo-form-sku.component.ts](code/forms/src/app/demo-form-sku/demo-form-sku.component.ts)

и шаблон:

{lang=html}
<<[code/forms/src/app/demo-form-sku/demo-form-sku.component.html](code/forms/src/app/demo-form-sku/demo-form-sku.component.html)

Если мы откроем это в браузере, то это будет выглядеть вот так:

![Пример простой формы Sku: статус отправлено](images/forms/demo_form_sku_simple_submitted.png)


## Используем `FormBuilder`

Создание `FormControl` и `FormGroup` с неявным использованием `ngForm` и `ngControl` является удобным способом, но не предоставляет нам вариаций для настройки. Более гибким и распространенным способом cоздания форм является использование `FormBuilder`.

`FormBuilder` - это удобный класс, который помогает нам создавать формы. Как вы помните, формы состоят из `FormControl` и `FormGroup`, а `FormBuilder` помогает нам их создавать (вы можете рассматривать его как объект "фабрику").

Давайте добавим `FormBuilder` к нашему предыдущему примеру. Обратим наше внимание на то:

* как использовать `FormBuilder` в нашем классе, определяющем компонент
* как использовать нашу пользовательскую `FormGroup` в `form` в нашем шаблоне

## Реактивные формы с `FormBuilder`

Для этого компонента мы будем использовать директивы `formGroup` и `formControl`. Из этого следует, что нам нужно импортировать соответствующие классы. Сначала мы импортируем их следующим образом:

{lang=javascript,crop-query=1-6}
<<[code/forms/src/app/demo-form-sku-with-builder/demo-form-sku-with-builder.component.ts](code/forms/src/app/demo-form-sku-with-builder/demo-form-sku-with-builder.component.ts)

### Используем `FormBuilder`

Мы можем инжектировать `FormBuilder` путем создания аргумента в `constructor` класса нашего комопнента:

%% BEGIN_BOOK

W> **Что значит `инжектировать`?** Ранее мы не уделяли внимание понятию dependency injection (DI) и положению DI в иерархии, поэтому последнее предложение может показаться вам несколко непонятным. Многое о dependency injection мы рассматриваем в главе [Dependency Injection](#di). Поэтому, используйте эту главу, если хотите в этом детально разобраться.
W>
W> Если коротко, то Dependency Injection - это способ сообщить Angular, какие зависимости необходимы этому компоненту для нормальной работы.

%% END_BOOK

{lang=javascript}
<<[code/forms/src/app/demo-form-sku-with-builder/demo-form-sku-with-builder.component.ts](code/forms/src/app/demo-form-sku-with-builder/demo-form-sku-with-builder.component.ts)

В процессе инжектирования созданный экземпляр `FormBuilder` будет присвоен переменной `fb` (в конструкторе).

Две основные функции, которые мы будем использовать в `FormBuilder`:

* `control` - создает новый `FormControl`
* `group` - создает новую `FormGroup`

Обратите внимание, что мы установили новую _переменную экземпляра_ `myForm` в этом классе. (Мы могли бы назвать ее `form`, но мы хотим различать нашу `FormGroup` и `form`, которая у нас была ранее.)

`myForm` определен стать `FormGroup`. Мы создаем `FormGroup` вызывая `fb.group()`. `.group` принимает объект с парой ключ-значение, которые определяют `FormControl` в этой группе.

В этом случае, мы создаем один элемент управления `sku` со значением `["ABC123"]` - это означает, что значение по умолчанию для этого элемента будет `"ABC123"`. (Вы могли заметить, что это массив. Это потому, что позже мы добавим дополнительные параметры.)

Теперь, когда у нас есть `myForm`, мы должны ее использовать в нашем шаблоне (т.е. мы должны _связать_ ее в нашим элементом `form`).

### Используем `myForm` в шаблоне

Изменим нашу `<form>`, чтобы использовать `myForm`. Если вы помните, в последнем разделе мы говорили, что `ngForm` применяется автоматически при использовании `FormsModule`. Мы также упомянули, что `ngForm` создает свою собственную `FormGroup`. Тогда, в этом случае, мы  **не** хотим использовать внешнюю `FormGroup`. Вместо этого, мы хотим ипользовать нашу переменную экземпляра `myForm`, которую мы создали с помощью `FormBuilder`. Как мы можем это реализовать?

Angular предоставляет другую директиву, которые мы можем использовать **когда у нас уже есть `FormGroup`**: она называется `formGroup`. И мы используем ее следующим образом:

{lang=html,crop-start-line=2,crop-end-line=3}
<<[code/forms/src/app/demo-form-sku-with-builder/demo-form-sku-with-builder.component.html](code/forms/src/app/demo-form-sku-with-builder/demo-form-sku-with-builder.component.html)

Здесь мы сообщаем Angular, что хотим использовать `myForm` в качестве `FormGroup` для этой формы.

T> Как говорилось ранее, при использовании `FormsModule`, `NgForm` будет автоматически применен к элементу `<form>`. Но есть исключение: `NgForm` не будет применен к `<form>`, у которой есть `formGroup`.
T>
T> Если вам интересно, то `selector` для `NgForm` будет:
T>
T> ```javascript
T> form:not([ngNoForm]):not([formGroup]),ngForm,[ngForm]
T> ```
T>
T> Это значит, что у вас может быть форма, к которой не применяется `NgForm`, благодаря атрибуту `ngNoForm`.

Мы также должны изменить `onSubmit`, чтобы использовать `myForm`, в отличие от `f`, потому что сейчас это `myForm` со своими настройками и значениями.

Еще одна вещь, которую нам нужно сделать: связать наш `FormControl` с тэгом `input`. Запомните, что **`ngControl` создает новый объект `FormControl`**, и связывает его с родительским `FormGroup`. Но в этом случае, мы использовали `FormBuilder`, чтобы создать наш `FormControl`.

Когда мы хотим связать **сушествующий `FormControl`** с `input` мы используем `formControl`:

{lang=html,crop-start-line=8,crop-end-line=12}
<<[code/forms/src/app/demo-form-sku-with-builder/demo-form-sku-with-builder.component.html](code/forms/src/app/demo-form-sku-with-builder/demo-form-sku-with-builder.component.html)

Здесь мы сообщаем директиве `formControl` смотреть на `myForm.controls` и использовать существующий `sku` `FormControl` для этого `input`.

### Давайте попробуем!

Вот как это будет выглядеть вместе:

{lang=javascript}
<<[code/forms/src/app/demo-form-sku-with-builder/demo-form-sku-with-builder.component.ts](code/forms/src/app/demo-form-sku-with-builder/demo-form-sku-with-builder.component.ts)

и шаблон:

{lang=html}
<<[code/forms/src/app/demo-form-sku-with-builder/demo-form-sku-with-builder.component.html](code/forms/src/app/demo-form-sku-with-builder/demo-form-sku-with-builder.component.html)

Запомните:

Чтобы создать новые `FormGroup` и `FormControl` неявно используют:

* `ngForm` и
* `ngModel`

Но, чтобы связать существующие `FormGroup` и `FormControl` используйте:

* `formGroup` и
* `formControl`

## Добавляем валидацию

Наши пользователи не всегда вводят данные в нужном формате. Если кто-то введет данные в неправильном формате, мы хотим сообщить ему об этом и не допустить отправку данных. Для этого мы используем _валидаторы_.

Валидаторы предоставляются модулем `Validators`. Самый простой валидотор `Validators.required`, который говорит нам о том, что указанное поле является обязательным, иначе `FormControl` будет считаться недействительным.

Чтобы использовать валидаторы, нам нужно сделать следующее:

1. Назначить валидатор для `FormControl` объекта
2. Проверить состояние валидатора в шаблоне и отреагировать соответствующим образом

Для назначения валидатора объекту `FormControl` достаточно просто передать его в качестве второго аргумента в конструктор `FormControl`:

{lang=javascript}
    let control = new FormControl('sku', Validators.required);

В нашем случае, поскольку мы используем `FormBuilder` мы будем использовать следующий синтаксис:

{lang=javascript,crop-query=.constructor}
<<[code/forms/src/app/demo-form-with-validations-explicit/demo-form-with-validations-explicit.component.ts](code/forms/src/app/demo-form-with-validations-explicit/demo-form-with-validations-explicit.component.ts)

Теперь нам нужно использовать нашу проверку в шаблоне. Существует два способы получить значение проверки в шаблоне:

1. Мы можем назначить `FormControl` `sku` переменной экземпляра класса - что требует дополнительных действий, но предоставляет легкий доступ к `FormControl` в шаблоне.
2. Мы можем получить `FormControl` `sku` из шаблона `myForm`. Для этого потребуется меньше работы в классе определения компонента, но несколько больше в шаблоне .

Чтобы понять разницу, рассмотрим следующие примеры:

### Присваиванием `sku` `FormControl` переменной экземпляра

Ниже представлено изображение того, как должна выглядеть наша форма с проверками:

![Форма с валидацией](images/forms/demo-form-with-validations-explicit-invalid.png)

Самый гибкий способ обработки отдельных `FormControls` в вашем шаблоне - установить каждый `FormControl` как переменную экземпляра в классе определения компонента. Вот как мы можем настроить `sku` в нашем классе:

{lang=javascript,crop-query=.DemoFormWithValidationsExplicitComponent}
<<[code/forms/src/app/demo-form-with-validations-explicit/demo-form-with-validations-explicit.component.ts](code/forms/src/app/demo-form-with-validations-explicit/demo-form-with-validations-explicit.component.ts)

Следует отметить, что:

1. Мы устанавливаем `sku: AbstractControl` в верхней части класса и
2. Мы назначаем `this.sku` после создания `myForm` с помощью `FormBuilder`

Это здорово, потому что мы можем ссылаться на `sku` в любом месте шаблона нашего компонентов. Недостатком является то, что нам нужно настроить переменную экземпляра **для каждого поля в нашей форме**. Для больших форм это может быть весьма затруднительно.

Теперь, когда мы проверили наш `sku`, я давайте обратим внимание на четыре различных способа отображения его значение в нашем шаблоне:

1. Проверка действительности всей формы и отображение сообщения
2. Проверка действительности нашего отдельного поля и отображение сообщения
3. Проверка правильности нашего отдельного поля и окраска поля красным цветом, если оно недействительно
4. Проверка действительности отдельного поля по конкретному условию и отображение сообщения

#### Сообщение поля

Мы можем проверить правильность всей нашей формы обратившись к `myForm.valid`:

{lang=html,crop-start-line=20,crop-end-line=21}
<<[code/forms/src/app/demo-form-with-validations-explicit/demo-form-with-validations-explicit.component.html](code/forms/src/app/demo-form-with-validations-explicit/demo-form-with-validations-explicit.component.html)

Помните, что `myForm` является `FormGroup`, а `FormGroup` действителен, если все дочерние элементы `FormControl` также действительны.

#### Сообщение поля

Мы также можем отобразить сообщение для конкретного поля `FormControl`, если это поле недействительно:

{lang=html,crop-start-line=14,crop-end-line=17}
<<[code/forms/src/app/demo-form-with-validations-explicit/demo-form-with-validations-explicit.component.html](code/forms/src/app/demo-form-with-validations-explicit/demo-form-with-validations-explicit.component.html)

#### Окраска поля

Я использую класс `.error` из Semantic UI CSS Framework's CSS, что означает, если я добавлю класс `error` к `<div class= "field">`, то он сделает красную обводку у тэга input.

Для этого можно использовать синтаксис для установки условных классов:

{lang=html,crop-start-line=7,crop-end-line=9}
<<[code/forms/src/app/demo-form-with-validations-explicit/demo-form-with-validations-explicit.component.html](code/forms/src/app/demo-form-with-validations-explicit/demo-form-with-validations-explicit.component.html)

Обратите внимание, что у нас есть два условия для установки класса `.error`:`!sku.valid` и `sku.touched`. Идея состоит в том, что мы хотим показать состояние ошибки только в том случае, если пользователь попытался редактировать форму, и теперь она недействительна.

Чтобы проверить, введите некоторые данные в тэг `input` и затем удалите содержимое поля.

#### Специфическая проверка

Поле формы может быть недействительным по многим причинам. Часто мы хотим показать другое сообщение в зависимости от причины неудачной проверки.

Для специфической проверки мы используем метод `hasError`:

{lang=html,crop-start-line=17,crop-end-line=18}
<<[code/forms/src/app/demo-form-with-validations-explicit/demo-form-with-validations-explicit.component.html](code/forms/src/app/demo-form-with-validations-explicit/demo-form-with-validations-explicit.component.html)

Обратите внимание, что `hasError` определяется как в `FormControl`, так и в `FormGroup`. Это означает, что вы можете передать второй аргумент для поиска конкретного поля из `FormGroup`. Например, мы можем переписать предыдущий пример следующим образом:

{lang=javascript}
         <div *ngIf="myForm.hasError('required', 'sku')"
           class="error">SKU is required</div>

#### Собираем все вместе

Ниже приведен полный листинг кода нашей формы с проверками с набором `FormControl` в виде переменной экземпляра:

{lang=javascript}
<<[code/forms/src/app/demo-form-with-validations-explicit/demo-form-with-validations-explicit.component.ts](code/forms/src/app/demo-form-with-validations-explicit/demo-form-with-validations-explicit.component.ts)

И шаблон:

{lang=html}
<<[code/forms/src/app/demo-form-with-validations-explicit/demo-form-with-validations-explicit.component.html](code/forms/src/app/demo-form-with-validations-explicit/demo-form-with-validations-explicit.component.html)

#### Удаление переменной экземпляра `sku`

В приведенном выше примере в качестве переменной экземпляра задается `SKU: AbstractControl`. Часто мы не хотим создавать переменную экземпляра для каждого `AbstractControl`. Так как мы будем ссылаться на этот `FormControl` в нашем шаблоне без переменной экземпляра?

Вместо этого мы можем использовать свойство `myForm.controls`:

{lang=html,crop-start-line=10,crop-end-line=17}
<<[code/forms/src/app/demo-form-with-validations-shorthand/demo-form-with-validations-shorthand.component.html](code/forms/src/app/demo-form-with-validations-shorthand/demo-form-with-validations-shorthand.component.html)

Таким образом, мы можем получить доступ к элементу управления `SKU`, не будучи вынуждены явно добавлять его в качестве переменной экземпляра класса компонента.

I> Мы использовали нотацию скобками, т.е. `myForm.controls['sku']`. Мы также можем использовать нотацию точкой, т.е. `myForm.controls.sku`. TypeScript может выдать предупреждение в случае использования нотации с точкой, если объект будет введен неверно.

### Нестандартные проверки

Нам часто требуются собственные проверки. Давайте разберем, как это сделать.

Чтобы понять, как реализованы валидаторы, давайте посмотрим на `Validators.required` из Angular core:

{lang=javascript}
    export class Validators {
      static required(c: FormControl): StringMap<string, boolean> {
        return isBlank(c.value) || c.value == "" ? {"required": true} : null;
      }

Валидатор:
- Принимает `FormControl` в качестве входных данных и
- Возвращает `StringMap<string, boolean>`, где ключ "error code" и значение `true` соответствует ошибке

#### Реализация валидатора

Предположим, у нас есть конкретные требования к значению `sku`. Например, значение `sku` должно начинаться с `123`. Для решения этой задачи мы можем написать следующий валидатор:

{lang=javascript,crop-query=.skuValidator}
<<[code/forms/src/app/demo-form-with-custom-validation/demo-form-with-custom-validation.component.ts](code/forms/src/app/demo-form-with-custom-validation/demo-form-with-custom-validation.component.ts)

Этот валидатор вернет код ошибки `invalidSku`, если значение `control.value` не будет начинаться с `123`.

#### Назначаем валидатор для `FormControl`

Теперь нам нужно добавить валидатор в наш `FormControl`. Однако есть одна небольшая проблема: у нас уже есть валидатор для `sku`. Как добавить несколько валидаторов для одного поля?

Для этого мы используем `Validators.compose`:

{lang=javascript,crop-query=context(.compose, 2, 2)}
<<[code/forms/src/app/demo-form-with-custom-validation/demo-form-with-custom-validation.component.ts](code/forms/src/app/demo-form-with-custom-validation/demo-form-with-custom-validation.component.ts)

`Validators.compose` обертывает два наших валидатора и позволяет назначить их к `FormControl`. `FormControl` является недействительным, если хотя бы одно из условий неверно.

Теперь мы можем использовать новый валидатор в нашем шаблоне:

{lang=html,crop-start-line=19,crop-end-line=20}
<<[code/forms/src/app/demo-form-with-custom-validation/demo-form-with-custom-validation.component.html](code/forms/src/app/demo-form-with-custom-validation/demo-form-with-custom-validation.component.html)

I> Обратите внимание, что в этом разделе я использую «явную» нотацию добавления переменной экземпляра для каждого `FormControl`. Это означает, что в шаблоне к этой главе, `sku` относится к `FormControl`.

Если вы запустите пример кода, то вы заметите, что, если вы введете что-то в поле, проверка `required` будет выполнена, но проверка `invalidSku` может и не быть. Здорово - это значит, что мы можем частично проверить наши поля и показать соответствующие сообщения об ошибке.

## Отслеживание изменений

До сих пор мы извлекали значение из нашей формы, вызывая `onSubmit`, когда отправляли форму. Но часто мы хотим отслеживать изменения значений в элементе.

У `FormGroup` и `FormControl` есть `EventEmitter`, которое позволяет нам наблюдать за изменениями.

I> `EventEmitter` является _Observable_, что означает, что он соответствует определенной спецификации для просмотра изменений. Если вы заинтересованы в спецификации Observable, [то вы можете найти ее тут](https://github.com/jhusain/observable-spec)

Чтобы следить за измененями, нам необходимо:

1. получить доступ к `EventEmitter` обратившись к `control.valueChanges`. Затем мы
2. добавляем _observer_ используя метод `.subscribe`

Пример:

{lang=javascript,crop-query=.constructor}
<<[code/forms/src/app/demo-form-with-events/demo-form-with-events.component.ts](code/forms/src/app/demo-form-with-events/demo-form-with-events.component.ts)

Здесь мы наблюдаем два отдельных события: изменения в поле sku и изменения в форме в целом.

Наблюдатель, в который мы передаем, - это объект с одним ключом: `next` (есть другие ключи, которые вы можете передать, но сейчас не об этом). `next` - это функция, которую мы хотим вызвать с новым значением, как только оно изменится.

Если мы напечатаем в текстовом поле '`kj`', то в консоли мы увидим:

{lang='text'}
    sku changed to:  k
    form changed to:  Object {sku: "k"}
    sku changed to:  kj
    form changed to:  Object {sku: "kj"}

Вы можете наблюдать, что каждое нажатие клавичи приводит к изменению элемента, что приводит к запуску наблюдателя. Когда мы наблюдаем за индивидуальной `FormControl` мы получаем значение (например, `kj`), но когда мы наблюдаем за целой формой, мы получаем объект, состоящий из пар ключ-значение (например, `{sku: "kj"}`).

## ngModel {#forms_ng_model}

`NgModel` это специальная директива: она связывает модель с формой. Особенность `ngModel` в том, что он реализует **двустороннюю связь**. Двусторонняя связь данных почти всегда сложнее и труднее для понимания в сравнении с односторонней связью. Angular устроен так, что в нем реализован односторонний поток данных: сверху-вниз. Однако, когда дело касается форм, бывают случаи, когда легче использовать двустороннюю связь.

W> Не спешите сразу использовать `ngModel`, только потому, что ранее в Angular 1 вы использовали `ng-model`. Есть веские причины [избегать двустороннюю передачу данных](https://www.quora.com/Why-is-the-two-way-data-binding-being-dropped-in-Angular-2). Конечно, `ngModel` может быть очень удобен, но знайте, что не следует полагаться на двустороннюю связь, как в Angular 1.

Давайте немного изменим нашу форму, например, мы хотим получить значение `productName`. Мы собираемся использовать `ngModel` для синхронизации экземпляра компонента с шаблоном.

Во-первых, вот наш класс определения компоненты:

{lang=javascript,crop-query=.DemoFormNgModelComponent}
<<[code/forms/src/app/demo-form-ng-model/demo-form-ng-model.component.ts](code/forms/src/app/demo-form-ng-model/demo-form-ng-model.component.ts)

Обратите внимание, что мы храним `productName: string` в качестве переменной экземпляра.

Дальше давайте используем `ngModel` в тэге `input`:

{lang=html,crop-start-line=13,crop-end-line=18}
<<[code/forms/src/app/demo-form-ng-model/demo-form-ng-model.component.html](code/forms/src/app/demo-form-ng-model/demo-form-ng-model.component.html)

Теперь заметьте, синтаксис для `ngModel` очень прост: мы используем квадратные и круглые скобки вокруг атрибута `ngModel`! Идея состоит в том, что мы используем квадратные скобки для _ввода_, а круглые для _вывода_. Это указывает на двустороннюю связь.

Обратите внимание на другое: мы все еще используем `formControl`, чтобы указать, что этот `input` должен быть привязан к `FormControl` в нашей форме. Мы делаем это потому, что `ngModel` только связывает `input` с переменной экземпляра - `FormControl` полностью отделен. Но поскольку мы все еще хотим проверить это значение и отправить его как часть формы, мы сохраняем директиву `formControl`.

И наконец, давайте отобразим значение `productName` в нашем шаблоне:

{lang=html,crop-start-line=4,crop-end-line=6}
<<[code/forms/src/app/demo-form-ng-model/demo-form-ng-model.component.html](code/forms/src/app/demo-form-ng-model/demo-form-ng-model.component.html)

Вот как это выглядит:

![Пример формы с ngModel](images/forms/demo-form-ng-model-submitted.png)

Все просто!

## Заключение

Формы состоят из большого числа изменяющихся элементов, но Angular все это значительно упрощает. Как только вы получите представление о том, как следует использовать `FormGroup`, `FormControl` и `Validation`, все станет очевидным!

%% BLOG{call_to_action}
