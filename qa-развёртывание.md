# Развёртывание

## Автоматический запуск тестов в GitLab

Добавляем в файл `.gitlab-ci.yml`:

```yml
stages:
  - prepare
  - test
```

:::tip
Порядок этапов важен, `prepare` должен находится в перед
`test`.
:::

Добавляем этап `prepare`:

```yml
npm_dependencies:
  stage: prepare
  image: node:12
  artifacts:
    untracked: true
    expire_in: 1 hour
  script:
    - npm i
  tags:
    - docker
    - linux
```

Добавляем переменные:

```yml
variables:
  CI_PROJECT_NAME: "MY_PROJECT"
  CYPRESS_CACHE_FOLDER: "${CI_PROJECT_NAME}/cache/Cypress"
```

:::tip
`CI_PROJECT_NAME`- имя вашего проекта в гитлабе.

`CYPRESS_CACHE_FOLDER` - директория для кеша [cypress][1].
:::

Добавляем этап запуска тестов:

```yml
cypress_tests:
  stage: test
  image: cypress/base:10
  script:
    - npm run cypress
  artifacts:
    when: always
    paths:
      - cypress/videos/
      - cypress/screenshots/
    expire_in: 24 hours
  tags:
    - docker
    - linux
```

:::tip
`expire_in: 24 hours` указывает на время хранения, сгенерированных во время
прохождения тестов, скриншотов и видео. Можно менять для разных типов
тестирования. Время указывается в формате:
`N yrs N mos N days N hours N min N sec`.
:::

Обновляем файл `package.json`:

```json
"scripts": {
    "cypress:open": "cypress open",
    "cypress:frontend:serve": "ng serve --configuration=testing --port=5555",
    "cy:serve:local": "cypress:open cypress:frontend:serve",
    "cypress:tests:run": "cypress run",
    "cypress": "start-server-and-test cypress:frontend:serve http-get://127.0.0.1:5555 cypress:tests:run"
}
```

:::tip
В `cypress:frontend:serve` указывается команда для запуска фронтенд-проекта
:::

Пример готового файла `.gitlab-ci.yml`:

```yml
stages:
  - prepare
  - test

variables:
  CI_PROJECT_NAME: "MY-PROJECT"
  CYPRESS_CACHE_FOLDER: "${CI_PROJECT_NAME}/cache/Cypress"

npm_dependencies:
  stage: prepare
  image: node:12
  artifacts:
    untracked: true
    expire_in: 1 hour
  script:
    - npm i
  tags:
    - docker
    - linux

cypress_tests:
  stage: test
  image: cypress/base:10

  script:
    - npm run cypress
  artifacts:
    when: always
    paths:
      - cypress/videos/
      - cypress/screenshots/
    expire_in: 24 hours
  tags:
    - docker
    - linux
```

## Управление окружениями

Для запуска тестов на разных окружениях необходимо создать отдельные
конфигурации [Cypress][1], которые можно указывать в параметрах запуска.

Добавляем в `./cypress/plugins/index.js`:

```js
const fs = require('fs-extra');
const path = require('path');

function getConfigurationByFile (file) {
  const pathToConfigFile = path.resolve('.', 'cyconfig', `${file}.json`);
  return fs.readJson(pathToConfigFile);
}

module.exports = (on, config) => {
  const file = config.env.configFile || 'dev';
  return getConfigurationByFile(file);
}
```

Создаем, основанные на файле `cypress.json`, новые файлы конфигурации
`dev.json`, `stg.json`, `prod.json` в директории `./cyconfig`.

Запускаем [Cypress][1] с нужной конфигурацией:

```yml
cypress run --env configFile=dev
```

## Взаимодействие с Docker

По умолчанию [Cypress][1] в CI запускает тесты в браузере Electron, который по
поведению отличается от Chrome.
При нехватке памяти или задержке загрузки iframe [Cypress][1] в начале тестов
рекомендуется использованию опцию `--disable-dev-shm-usage` с Google Chrome.

Добавляем в `./cypress/plugins/index.js`:

```js
module.exports = (on, config) => {
  on('before:browser:launch', (browser = {}, launchOptions) => {
    if (browser.family === 'chrome') {
      console.log('Adding --disable-dev-shm-usage...')
      launchOptions.args.push('--disable-dev-shm-usage')
    }
    return launchOptions
  })
}
```

Меняем образ в `.gitlab-ci.yml` на актуальную версию Chrome, например:

```yml
image: cypress/browsers:node12.18.0-chrome83-ff77
```

Запуск:

```yml
cypress run --browser chrome
```

[1]:https://cypress.io
