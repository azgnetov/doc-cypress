# Начало работы

## Настройка окружения

### Системное окружение

* Mac OS / Windows / Ubuntu, Debian, Fedora, etc.
* Последняя стабильная версия [Node.js][2]

## Инициализация проекта

Переходим в корень проекта:

```sh
cd path/to/root/of/the/project
```

Устанавливаем зависимости:

```sh
npm install cypress start-server-and-test @testing-library/cypress --save-dev
```

Запускаем [Cypress][1]:

```sh
npx cypress open
```

:::tip
Первый запуск необходим для автоматического создания директорий и файлов конфигураций.
:::

### Настройка проекта

Мы используем переменную `baseUrl` вместо переменных окружения для `origin` или `domain`,
которую нужно указать в корне проекта, в автоматически созданном файле `cypress.json`:

```json
{
  "baseUrl": "http://localhost:5555"
}
```

:::tip
Мы используем порт 5555 для тестирования, чтобы избежать конфликтов
со стандартными настройками разных фреймворков.
:::

Создаем тестовую конфигурацию для используемого фреймворка:

Для Angular добавляем в файл `angular.json` в корне проекта тестовую конфигурацию:

```json
{
  "build": {
    ...
    "configurations": {
      "testing": {
        ...
        "fileReplacements": [
          {
            "replace": "src/configurations/configuration.ts",
            "with": "src/configurations/configuration.testing.ts"
          }
        ]
      }
    }
    ...
  },
  ...
  "serve": {
    ...
    "configurations": {
      "testing": {
        "browserTarget": "pms-frontend:build:testing"
      }
    }
    ...
  },
  ...
}
```

:::tip
`PROJECT_NAME` - необходимо заменить на имя проекта
:::

Создаем файл `/src/configurations/configuration.testing.ts`,
копируя содержимое из `configuration.ts`.

Добавляем команды в `package.json`:

```json
{
  "scripts": {
    ...
    "cypress:open": "cypress open",
    "cypress:frontend:serve": "ng serve --configuration=testing --port=5555",
    "cy:serve:local": "cypress:open cypress:frontend:serve",
    "cypress:tests:run": "cypress run",
    "cypress": "start-server-and-test cypress:frontend:serve http-get://127.0.0.1:5555 cypress:tests:run"
    ...
    },
  ...
}
```

`cypress:frontend:serve` - запуск фронтенда с указанной конфигурацией
и номером порта, в зависимости от фреймворка:

* Angular - команда `ng serve`.
* React - команда `PORT=5555 react-scripts start`.
* Vue - команда `npm run build`.

`cy:serve:local` - локальный запуск приложения и [Cypress][1] в режиме GUI;

`cypress:tests:run` - запуск тестов в `headless` режиме;

`cypress` - упорядоченный запуск приложения и тестов в `headless` режиме.

Теперь, для запуска [Cypress][1] можно использовать команду:

```sh
npm run cypress:open
```

Добавляем в файл `gitignore`:

```sh
*.code-workspace
cypress/screenshots
cypress/videos
cypress/logs
cypress/fixtures
```

[1]:https://cypress.io
[2]:https://nodejs.org
