# Стиль кода

## Основные правила

### Новый Тест - Новый Spec

Для создания нового теста всегда создаём новый файл, который
содержит в себе тестовый сценарий либо набор сценариев,
проверяющий какую-то функцию или действие пользователя.
Не стоит писать множество тестов в одном файле, это усложнняет логику
и увеличивает потребление памяти.

### Независимые тесты

При создании тестов важно, чтобы было как можно меньше зависимых и
повторяющихся проверок. Тесты не должны зависеть друг от друга,
таким образом при сбое какого-либо компонента все остальные тесты
продолжат выполнение.
Повторяющийся код нужно выносить в блоки `before` или `beforeEach`.
Атомарные проверки в отдельных блоках `it` нужны только в тех случаях,
когда они являются мягкими, а для жестких проверок допустимо
объединение в одном блоке `it`.

:::tip Рекомендуется

```js
context('Admin on the new user page', () => {

  describe('When page loaded', () => {

    before(() => {
      cy.login('admin');
      cy.visit('/users/new');
    });

    it('sees new user  form fields', () => {
      cy.findByTestId('form-fields').should('be.displayed');
    });
  });

  describe('When clicks submit button', () => {

    before(() => {
      cy.findByTestId('submit-button').click();
    });

    it('sees displayed error alerts', () => {
      cy.findByTestId('first-name-error').should('contain', 'First name is required');
      cy.findByTestId('last-name-error').should('contain', 'Last name is required');
      cy.findByTestId('phone-error').should('contain', 'Phone is required');
    });
  });

  describe('When fills all mandatory fields and clicks submit', () => {

    before(() => {
      cy.findByTestId('first-input').clear();
      cy.findByTestId('submit-button').click();
    });

    it('seees displayed success message', () => {
      cy.findByTestId('success-message').should('contain', 'User created');
    });
  });

});
```

:::

:::danger Не рекомендуется

```js
context('User on the start page', () => {

  describe('When submits account data', () => {

    it('visits the form page', () => {
      cy.visit('/users/new');
    });

    it('requires first name', () => {

      cy.findByTestId('first-input').type('Johnny');
    });

    it('requires last name', () => {
      cy.findByTestId('last-input').type('Appleseed');
    });

    it('can submit a valid form', () => {
      cy.findByTestId('form').submit();
    });
  });
});
```

:::

### Возвращаемые значения

Поскольку [Cypress][1] выполняет команды асинхронно,
для возвращаемых значений нельзя использовать присвоение
через `var`, `const` или `let`.
Нужно использовать замкнутые выражения через `then()`.

:::tip Рекомендуется

```js
cy.findByTestId('button').then(($btn) => {
  const txt = $btn.text();
  ...
  cy.findByTestId('form').submit();
})
```

:::

:::danger Не рекомендуется

```js
const button = cy.get('button');
const form = cy.findByTestId('form');
...
button.click()
```

:::

### Сторонние ресурсы

Не нужно тестировать сторонние ресурсы на уровне UI.
Если необходимо, остается использовать `cy.request()` на уровне
API. Еще одним способом предотвращения тестирования стороннего
ресурса является применение [Application Actions][2].
Например, в случае если вам необходимо проверить оплату, то не нужно проверять
сторонние сервисы (yandex money, stripe, paypal и т.д.). Можно просто вернуть
результат успешной или не успешной операции.

### Излишние ожидания

Избегайте использование фиксированных ожиданий через `cy.wait()`,
поскольку это излишне замедляет тест.
Например, команды `cy.visit()` и `cy.request()` заканчиваются
только после получения ответа от
сервера, поэтому не нуждаются в дополнительных ожиданиях.
Выполнение `cy.get()` так же происходит синхронно, поэтому,
команды типа assertion заканчиваются только после проверки,
соответственно их можно использовать вместо ожиданий появления элементов.

## Выбор элементов

Необходимо использовать устойчивые к изменениям селекторы.

[Cypress][1] использует [CSS локаторы][3] для выбора элемента из DOM дерева.
С помощью комманды `cy.get()` можно найти любой элемент на странице.
Поведение запроса команды `cy.get()` точно соответствует тому,
как `$(…)` работает в jQuery.

```js
cy.get(selector);
cy.get(selector, options);
cy.get(alias, options);
```

:::tip Рекомендуется
Использовать атрибуты `data-testid`, чтобы предоставить контекст своим
селекторам и изолировать их от изменений CSS или JS.
:::

:::danger Не рекомендуется
Использовать хрупкие селекторы, которые могут меняться,
например `class`, `id`, `tag`, `textContent`.
:::

В остальных случаях, например когда нет доступа к коду проекта,
можно использовать любые
имеющиеся селекторы. Например, при тестировании локализации,
можно выбрать элементы по тексту, используя метод `cy.contains()`.

### Наименование селекторов

Селектор элемента необходимо называть в стиле `kebab-case`
с маской `название-тип_элемента`.

:::tip Рекомендуется

```html
<a href="#" data-testid="facebook-button">Login via Facebook</a>
```

:::

:::danger Не рекомендуется

```html
<a href="#" data-testid="fbButtonSOCIAL">Login via Facebook</a>
```

:::

:::tip
В случаях, когда элемент неуникален, используются дополнительные атрибуты.
Например, в случае когда у вас есть две кнопки с одинаковым названием
и типом, но располагаются они в разных местах страницы или обладают разными состояниями,
мы добавляем атрибуты "расположение" и/или "состояние".

`расположение-название-тип_элемента-состояние`

```html
<img class="header-user-avatar" data-testid="topbar-profile-img-enabled">
```

Чтобы добавить `data-testid` в общий компонент с уникальным идентификатором
какой-либо сущности, нужно добавить в код:

```html
<a href="#" [attr.data-testId]="'facebook-button-' + button.id">Login via Facebook</a>
```

:::

## Переменные

Для удобства восприятия, переменные селекторов выносятся в начало теста.
Имена переменных селекторов формируются по маске `НАЗВАНИЕ_ТИП` по умолчанию и,
если элемент может принимать более сложные состояния -
`РАСПОЛОЖЕНИЕ_НАЗВАНИЕ_ТИП_СОСТОЯНИЕ`.

:::tip Рекомендуется

```js
const PROFILE_ICON = 'img.profile';
```

:::

:::danger Не рекомендуется

```js
const icon = 'img.profile';
```

:::

Например селектор активной кнопки на верхней панели инструментов
может быть записан в формате:

```js
const TOPBAR_SEARCH_BUTTON_ACTIVE = 'a[class="top-button"]';
```

## Фикстуры

Фикстуры должны быть расположены в директории `cypress/fixtures` и
названы в формате `kebab-case`, по маске `страница-действие.json`. При
необходимости, маска может быть дополнена дополнительными
аттрибутами `тип`, `статус`, `состояние`, `свойство` и т.д.

Константы и переменные окружения могут быть расположены как в фикстурах,
так и в файле `cypress.json`, например:

```json
"env": {
  "email": "user@test.com",
  "password": "123456"
}
```

## Пример оформления кода

```js
// переменная, хранящая css селектор.
const INPUT_EMAIL_ADDRESS = 'input[formcontrolname="email"]';

// Описание сценария (страницы) - Given
context('User on login page', () => {

  // Описание действий сценария - When
  describe('Guest user opens login page', () => {

    // Повторяющееся действие для каждого 'it' (если необходимо).
    before(() => { // Action
      cy.visit('/');
    });

    // Описание ожидаемого результата и выполняемых проверок - Then
    it('title should be correct', () => {
      // Проверяем корректность title страницы.
      cy.title().should('include', 'Login page');
    });

    // Описание ожидаемого результата и выполняемых проверок - Then
    it('email address input field should exist', () => {
      // Проверяем наличие поля email address.
      cy.get(INPUT_EMAIL_ADDRESS).should('exist');
    });

  });

});
```

```js
// Описание сценария (страницы) - Given
context('Admin on the new user page', () => {

  // Описание действий сценария - When
  describe('When page loaded', () => {

    before(() => {
      //Вызов кастомной команды логина
      cy.login('admin');
      //Переход на страницу localhost:port/user/new
      cy.visit('/users/new');
    });

    // Описание ожидаемого результата и выполняемых проверок - Then
    it('Displayed new user form fields', () => {
      //Проверка формы на видимость
      cy.findByTestId('form-fields').should('be.displayed');
    });
  });
  // Описание действий сценария - When
  describe('When admin click submit button', () => {
    before(() => {
      //Жмём кнопку Submit
      cy.findByTestId('submit-button').click();
    });

    // Описание ожидаемого результата и выполняемых проверок - Then
    it('displayed error alerts', () => {
      //Проверяем что отображаються ошибки
      cy.findByTestId('first-name-error').should('contain', 'First name is required');
      cy.findByTestId('last-name-error').should('contain', 'Last name is required');
      cy.findByTestId('phone-error').should('contain', 'Phone is required');
    });
  });

  // Описание действий сценария - When
  describe('When admin fill all mandatory fields and click submit', () => {

    before(() => {
      //Чистим поле First name
      cy.findByTestId('first-input').clear();
      //Жмём кнопку Submit
      cy.findByTestId('submit-button').click();
    });

    // Описание ожидаемого результата и выполняемых проверок - Then
    it('displayed success message', () => {
      //Проверяем содерживое формы об успешном создании юзера
      cy.findByTestId('success-message').should('contain', 'User created');
    });

  });

});
```

[1]:https://cypress.io
[2]:https://www.cypress.io/blog/2019/01/03/stop-using-page-objects-and-start-using-app-actions/
[3]:https://comaqa.gitbook.io/selenium-webdriver-lectures/selenium-webdriver.-vvedenie/tipy-lokatorov
