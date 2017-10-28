# Формы в Angular {#forms}

## Такие важные и сложные формы

Вероятнее всего, формы являются наиболее важной частью веб-приложения. Несмотря на 

Несмотря на обилие нажатий по ссылкам и перемещений курсора, все же самую большую и важную часть данных от пользователей мы получаем через _формы_.

На первый взгляд, формы кажутся очень простыми: вы создаете тэг `input`, пользователь заполняет форму и нажимает submit. Неужели все так сложно?

Оказывается, формы могут быть очень сложными. И вот почему:

* Формы могут использоваться для изменения данных как на странице, так и на сервере
* Очень часто изменения должны быть отображены в другой части страницы
* Пользователи не ограничены в том, что они вводят. Следовательно, у нас должна быть возможность проверять эти данные
* Пользовательский интерфейс должен: быть предсказуемым и четко обрабатывать ошибки, в случае их появления
* Зависимые поля могут иметь сложную логику
* У нас должна быть возможность тестировать наши формы, не прибегая к DOM селекторам

К счастью, в Angular есть инструменты, которые помогут нам решить эти проблемы.

* **`FormControl`** позволяет нам инкапсулировать данные формы и предоставляет объекты для работы с ними
* **`Validator`** позволяет нам проверять данные любым способом, которым мы захотим
* **Observers** позволяет нам отслеживать изменения в формах и реагировать на них соответствующим образом

В этой главе, шаг за шагом, мы рассмотрим, как создаются формы. Мы начнем с простых формы и дальше будем усложнять их логику.

## `FormControl` и `FormGroup`

Два основных объекта в Angular формах - это `FormControl` и `FormGroup`.

### `FormControl`

`FormControl` представляет собой одно поле ввода - это наименьшая часть формы в Angular.

`FormControl` инкапсулирует данные формы и состояния, сообщающие о валидности, изменениях или начилии ошибок.

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

Для создания формы мы создаем `FormControl`ы (или группу `FormControl`), а затем присоединяем к ним метаданные и логику.

Как и многое другое в Angular, мы добавляем класс (в данном случае `FormControl`) в DOM с помощью атрибута (в данном случае `formControl`). В итоге, это может выглядеть так:

```html
<!-- часть большой формы -->
<input type="text" [formControl]="name" />
```

Это создаст новый объект `FormControl` в контексте нашей формы. Далее мы подробно разберем, как это работает.

### `FormGroup`

Многие формы состоят из нескольких полей, поэтому нам нужен способ управления несколькими `FormControl`. Если мы хотим проверить правильность нашей формы, то перебор в массиве всех `FormControl` с валидацией каждого элемента будет выглядеть достаточно громоздким. `FormGroup` решает эту проблему путем создания интерфейса обертки вокруг коллекции `FormControl`.

Создадим `FormGroup`:

```javascript
let personInfo = new FormGroup({
    firstName: new FormControl("Nate"),
    lastName: new FormControl("Murray"),
    zip: new FormControl("90210")
})
```

`У FormGroup` и `FormControl` общий предок ([`AbstractControl`](https://angular.io/docs/ts/latest/api/forms/index/AbstractControl-class.html)). Это означает, что мы можем проверить `status` или `value` у `personInfo` так же легко. как и у одного `FormControl`:

```javascript
personInfo.value; // -> {
//   firstName: "Nate",
//   lastName: "Murray",
//   zip: "90210"
// }

// теперь мы можем запросить информацию о состоянии группы,
// значение котороц зависит от значений дочерних 'FormControl':
personInfo.errors // -> StringMap<string, any> of errors
personInfo.dirty  // -> false
personInfo.valid  // -> true
// и т.д.
```

Обратите внимание, когда мы пытаемя получить `value` из `FormGroup` мы получаем **объект** с парами ключ-значение. Очень удобно получать весь список значений нашей формы без перебора всех `FormControl`.

## Наша первая форма

Существует множество ньюансов при создании формы, и некоторые важные их них мы еще не затрагивали. Поэтому, давайте перейдем к примеру и разберем его по частям.

%% BEGIN_BOOK

I> Все примеры кода для данной главы вы можете найти в папке `forms/`

%% END_BOOK

%% BLOG{full_code_listing}

Вот изображение формы, которую мы собираемся создать:

![Пример простой формы Sku](images/forms/demo_form_sku_simple.png)

В нашем воображаемом приложении мы создаем сайт электронной коммерции, на котором мы публикуем продаваемые продукты. В этом приложении нам необходимо сохранять SKU продукта. Поэтому, давайте создадим простую форму, в которой SKU будет единственным полем для ввода данных.

I> SKU - это артикул товара. Это уникальный id продукта, который мы будем отслеживать. В будущем, когда мы говорим о SKU, мы подразумеваем ID продукта.

У нас очень простая форма: всего одно поле ввода для `sku` (с меткой label) и кнопкой submit.

Давайте превратим эту форму в компонент. Если вы помните, для создания компонента необходимо выполнить три шага:

* Сконфигурировать декоратор `@Component()`
* Создать шаблон
* Реализовать пользовательские функции в классе определения компонента

### Загрузка `FormsModule`

Чтобы начать использовать библиотеку создания форм, нам нужно сначала убедиться в том, что мы ее импортировали в наш `NgModule`.

Существует два способа работы с формами в Angular. В этой главе мы поговорим о каждом из них: с использованием `FormsModule` и `ReactiveFormsModule`. Поскольку мы будем использовать оба варианта, мы импортируем их в наш модуль. Для этого в `app.ts` мы делаем следующее:

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

... и еще. Мы не говорили о том, как использовать эти директивы, и что они делают. Но мы скоро это рассмотрим. На данный момент, нам просто нужно запомнить, что импортируя `FormsModule` и `ReactiveFormsModule` в наш `NgModule`, мы можем использовать любую директиву из этого списка в нашем шаблоне или внедрить в наши компоненты.

### Простая форма SKU: декоратор @Component 

Теперь создадим компонент:

{lang=javascript,crop-query=1-.templateUrl}
<<[code/forms/src/app/demo-form-sku/demo-form-sku.component.ts](code/forms/src/app/demo-form-sku/demo-form-sku.component.ts)

Определим `selector`  `app-demo-form-sku`. Если вы помните, `selector` сообщает Angular, что элементы связаны с этим компонентом. В этом случае мы можем использовать этот компонент с помощью тега `app-demo-form-sku`. Например:

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

Вы можете заметить, что мы используем оба в теге `<form>` в нашем шаблоне:

{lang=html,crop-start-line=3,crop-end-line=4}
<<[code/forms/src/app/demo-form-sku/demo-form-sku.component.html](code/forms/src/app/demo-form-sku/demo-form-sku.component.html)

Изначально у нас есть `#f="ngForm"`. Синтаксис `#v=thing` говорит, что мы хотим создать локальную переменную для этого шаблоа.

Создадим алиас для `ngForm` относящийся к переменной `#f` для этого шаблона. Откуда появился `ngForm`? Из директивы `NgForm`.

К какому типу объекта относится `ngForm`? Он относится к `FormGroup`. Значит мы можем использовать `f` как `FormGroup` в нашем пшаблоне. Это как раз то, что мы далем при `(ngSubmit)`.

W> Внимательные читатели могут заметить, что мы говорили, что `NgForm` автоматически привязывается к тэгам `<form>` (из-за селектора `NgForm`). Это означает, что нам не нужно добавлять атрибут `ngForm`, чтобы использовать `NgForm`. Но мы все же помещаем `ngForm` в качестве атрибута тэга. Неужели опечатка?
W>
W> Нет, это не опечатка. Если бы `ngForm` был _значением_ атрибута, то тем самым мы бы сообщили Angular, что мы хотим использовать `NgForm` для этого атрибута. В этом случае мы используем `ngForm` как _атрибут_ когда мы назначем _ссылку_. То есть, мы говорим, что значение выражения `ngForm` должно быть присвоено локальной переменной шаблона `f`.
W>
W> `ngForm` уже находится в этом элементе и вы можете думать, как будто мы "экспортируем" `FormGroup`, чтобы мы имели возможность сослаться на нее в другом месте нашего шаблона.

Мы связываем действие `ngSubmit` нашей формы с помощью синтаксиса: `(ngSubmit)="onSubmit(f.value)"`.

* `(ngSubmit)` - поступает из `NgForm`
* `onSubmit()` - будет реализован в нашем классе определения компонента (см. ниже)
* `f.value` - `f` - это `FormGroup`, как было указано выше. И `.value` вернет пары ключ/значение этой `FormGroup`

Если обобщить, то все это означает: "когда я отправляю форму, вызов `onSubmit` в экземпляре компонента передает значение формы в качестве аргументов".

#### `input` и `NgModel`

В тэге `input` есть некоторые вещи, на которые мы должны обратить внимание, прежде чем заговорим о `NgModel`:

{lang=html,crop-start-line=3,crop-end-line=14}
<<[code/forms/src/app/demo-form-sku/demo-form-sku.component.html](code/forms/src/app/demo-form-sku/demo-form-sku.component.html)

* `class="ui form"` и `class="field"` -  эти классы необязательны. Они из [CSS framework Semantic UI](http://semantic-ui.com/). Мы добавили их в некоторые из наших примеров для придания хорошего вида. Они не являются частью Angular.
* Атрибут `label` "`for`" и атрибут `input` "`id`" должны совпадать [согласно стандарту W3C](http://www.w3.org/TR/WCAG20-TECHS/H44.html)
* Мы устанавливаем в `placeholder` значение "SKU" только в качестве подсказки для пользователя, когда это поле ввода пустое.

Директива `NgModel` определяет `selector` `ngModel`. Это значит, что мы можем прикрепить его к тэгу `input`, добавив этот атрибут: `ngModel="whatever"`. В этом случае мы определяем `ngModel` без значения атрибута.

TСуществует несколько способов указать `ngModel` в ваших шаблонах, и это один из них. Когда мы используем `ngModel` без значения атрибута, мы определяем:

1. _одностороннюю_ связь данных
2. мы хотим создать `FormControl` в этой форме с именем `sku`

`NgModel` **создает новый `FormControl`**, который **автоматически добавляется** в родительский `FormGroup` (в данном случае, в форму) и затем привязывает элемент DOM к новому `FormControl`. То есть, он устанавливает связь между тэгом `input` в нашем шаблоне и `FormControl` и асспоциация совпала с именем, в данном случае `"sku"`.

I> **`NgModel` против `ngModel`**: в чем разница? Обычно, когда мы используем PascalCase, например, `NgModel`, мы указываем _class_ и ссылаеммя на объект, который определен в коде. В случае (CamelCase), например, `ngModel`, поступающий из `selector` директивы, он используется только в DOM / шаблоне.
I>
I> Также стоит отметить, что `NgModel` и `FormControl` являются отдельными объектами. `NgModel` - это _директива_, которую вы используете в вашем шаблоне, в то время как `FormControl` - объект, используемый для представления данных и проверок в форме.

T> Иногда мы хотим сделать _двустороннюю_ связь с помощью `ngModel`, как это реализовано в Angular 1. Мы рассмотрим, как это сделать в конце этой главы.

### Простая форма SKU: класс, определяющий компонент

Теперь давайте посмотрим на наше определение класса:

{lang=javascript,crop-query=.DemoFormSkuComponent}
<<[code/forms/src/app/demo-form-sku/demo-form-sku.component.ts](code/forms/src/app/demo-form-sku/demo-form-sku.component.ts)

Здесь наш класс определяет одну функцию: `onSubmit`. Это функция, которую мы вызываем, когда отправляем нашу форму. Сейчас мы просто выводим в консоль значение, которые передаем.

### Давайте попробуем!

Соединим все вместе. Вот как будет выглядеть наш код:

{lang=javascript}
<<[code/forms/src/app/demo-form-sku/demo-form-sku.component.ts](code/forms/src/app/demo-form-sku/demo-form-sku.component.ts)

и шаблон:

{lang=html}
<<[code/forms/src/app/demo-form-sku/demo-form-sku.component.html](code/forms/src/app/demo-form-sku/demo-form-sku.component.html)

Если мы откроем это в браузере, то это будет выглядеть так:

![Пример простой формы Sku: статус отправлено](images/forms/demo_form_sku_simple_submitted.png)


## Используем `FormBuilder`

Создание `FormControl` и `FormGroup` с неявным использованием `ngForm` и `ngControl` является удобным способом, но не предоставляет нам вариаций для настройки. Более гибким и распространенным способом cоздания форм является использование `FormBuilder`.

`FormBuilder` - это удобный класс, который помогает нам создавать формы. Как вы помните, формы состоят из `FormControl` и `FormGroup`, и `FormBuilder` помогает нам их создавать (вы можете думать об этом, как о "фабричном" объекте).

Давайте добавим `FormBuilder` к нашему предыдущему примеру. Обратим наше внимание на:

* как использовать `FormBuilder` в нашем классе, определяющем компонент
* как использовать нашу пользовательскую `FormGroup` в `form` в шаблоне

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

T> Как говорилось ранее, при использовании `FormsModule`, `NgForm` будет автоматически применен к элементу `<form>`? Есть исключение: `NgForm` не будет применен к `<form>`, у которой есть `formGroup`.
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

Когда мы хотим связать **сушествующий `FormControl`** к `input` мы используем `formControl`:

{lang=html,crop-start-line=8,crop-end-line=12}
<<[code/forms/src/app/demo-form-sku-with-builder/demo-form-sku-with-builder.component.html](code/forms/src/app/demo-form-sku-with-builder/demo-form-sku-with-builder.component.html)

Здесь мы сообщаем директиве `formControl` смотреть на `myForm.controls` и использовать существующий `sku` `FormControl` для этого `input`.

### Давайте попробуем!

Вот как это смотрится вместе:

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

Теперь, когда мы проверили наш `sku`, я хочу обратить ваше внимание на четыре различных способа отобразить его значение в нашем шаблоне::

1. Проверка действительности всей формы и отображение сообщения
2. Проверка действительности нашего отдельного поля и отображение сообщения
3. Проверка правильности нашего отдельного поля и окраска поля красным цветом, если оно недействительно
4. Проверка действительности нашего отдельного поля по конкретному требованию и отображение сообщения

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

Обратите внимание, что у нас есть два условия для установки класса `.error`:`!sku.valid` и `sku.touched`. Идея заключается в том, что мы хотим показать состояние ошибки только в том случае, если пользователь попытался редактировать форму, и теперь она недействительна.

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
