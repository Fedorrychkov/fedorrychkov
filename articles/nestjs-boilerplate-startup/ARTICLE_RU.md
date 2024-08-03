# Nestjs, Firebase, GCloud. Как быстро поднять API backend на TypeScript.


![Main preview](./images/BoilerplatePreview.png?raw=true "Main preview")

Очень здорово, что Вы решили открыть эту статью. Меня зовут Федор, я фуллстечу с конца 2021 года на постоянной основе. На всякий случай, мой профиль на [github](https://github.com/Fedorrychkov/Fedorrychkov). Этой небольшой статьей я хочу:
- Дать старт небольшой серии туториалов на тему запуска backend API
- Предоставить собранный пример nestjs проекта с интеграцией firebase
- Помочь разработчикам, выходцам из Frontend, быстро подготовить окружение для разработки бэка

Конкретно эта статья - описание и step by step инструкция по интеграции firebase с нюансами.

Еще хочу заранее предупредить читателя: Эта и будущие статьи в целом подойдут новичкам, но все же нужно быть знакомым с JavaScript/TypeScript в целом, либо не боятся гуглить то, что здесь будет разобрано не достаточно детально.


Нюансы работы с Firebase:
- На определенном этапе вам потребуется завести платежный аккаунт в Google Cloud, в моем случае, я взял отдельную карту Казахстанского банка.
Если у вас нет возможности привязать свою карту к firebase, есть несколько путей решения этой проблемы:
  - Завести карту в стране не находящейся под санкциями
  - Найти сервисы по генерации платежных карт
  - Заказать карту онлайн

Интеграция IaaS/PaaS решений на подобии Firebase не сильно отличаются друг от друга. Firebase условно бесплатная платформа, если укладываться в лимиты тарифного плана. Так же использование Firebase накладывает ограничения на плохо спроектированый код. На моем опыте, моя ошибка в проектировании частотности запросов к Firestore, повлекла за собой ущерб компании в 800 долларов в день при кратном росте трафика. Спустя некоторое время дебага удалось сократить расходы в 30 раз.

*Примечание*: предложенный стек хорошо подойдет для пет проектов и небольших production проектов. Итоговая бойлерплейт репа будет включать в себя необходимый минимум конфигурации для старта разработки. Если вы ранее не работали с Nodejs и Nest.js в частности, у вас могут возникнуть трудности с некоторыми концепциями и конструкциями кода, заранее рекомендую подготовить, например, [эту статью](https://habr.com/ru/companies/timeweb/articles/663234/).

## Личный опыт
На самом деле, хоть я и начал заниматься fullstack лишь несколько лет назад, мне уже доводилось пробовать себя в кросс функицональных проектах, на разных стеках и платформах, в основном это js/ts. С Nestjs познакомился попав в компанию, которая выбрала путь изоморфной Fullstack разработки, с тех пор этот путь меня не отпускает. Так вот, за несколько лет работы над разными проектами, мне удалось собрать простой и минимальный боейлерплейт для очередного MVP стартапа или пет проекта. Я очень надеюсь, что эта статья и итоговая репа, упростит Вам жизнь.

## Перед инициализацией
Я буду писать этот бойлерплейт со своей точки зрения и окружения, поэтому первым делом: *я работаю на платформе MacOS, все предлагаемые действия должны быть кроссплатформенными, но не исключаю трудностей*. Буду рад вопросам и рекомендациям в комментариях под этим постом.

### Мой сетап

В терминале я использую ZSH оболочку, вместо обычного BASH, поэтому первый линк - [ohmyzsh](https://github.com/ohmyzsh/ohmyzsh/wiki/Installing-ZSH).

Еще у меня есть необходимость держать разные версии nodejs, связано это с тем, что за годы работы накапливаются проекты разной свежести + привычки никто не отменял. Для удобной работы, предлагаю установить Вам [NVM](https://github.com/nvm-sh/nvm), он же менеджер версий для nodejs.

И напоследок, так как проектов у разработчика может быть много, использую [pnpm](https://pnpm.io/installation)

## Непосредственно инициализация

Перейдем к основному набору команд и конфигурации проекта. Для начала, давайте установим nodejs 20 версии при помощи nvm. ```nvm i 20 && nvm use 20```, а после поставим pnpm. ```npm i -g pnpm```

Предварительно, проверим, что все пакеты доступны в терминале.

```
➜  my pnpm -v
9.6.0
➜  my nvm -v
0.39.2
➜  my npm -v
10.8.1
➜  my nvm ls
->     v20.16.0
```

Двигаемся дальше и устанавливаем nest cli глобально: ```npm i -g @nestjs/cli```, после успешной установки cli можем перейти к шагу создания проекта. Делается это при помощи команды ```nest new nestjs-startup-boilerplate```, где после слова new вы можете написать название своего проекта. Далее будет предоставлен выбор конфигурации инициализации проекта. 

- 1 шаг, выбор пакетного менеджера, я выберу **pnpm**.
![Package manager to use](./images/nestjs_init.png?raw=true "Package Manager Step")

- 2 шаг, а на этом пока все, главное выбрать пакетный менеджер.

На данный момент, у вас должен получится [вот такой набор изменений](https://github.com/Fedorrychkov/nestjs-startup-boilerplate/commit/33355292acb7111adca3afbd769fd2fc81694efd) файлов проекта.

## Конфигурация
Линтинг, tsconfig, env файлы. Давайте законфигурим это все.

В любом проекте будет полезно иметь path aliases, чтобы не часто видеть импорты кода аля ```import something from '../../../../modules/something'```. Мне больше нравится что-то такое: ```import something from '~/modules/something'```. Читать и поддерживать это намного приятнее. В tsconfig файл внесем несколько изменений, объяснять подробно я не буду, [подробнее о настройках tsconfig](https://www.typescriptlang.org/tsconfig/).

![tsconfig changes](./images/tsconfig.png?raw=true "tsconfig changes")

Следом, я хочу внести изменения в eslint.js и добавить туда привычной мне конфиг.

![eslint changes](./images/eslintjs.png?raw=true "eslint changes")

Давайте обновим package.json, в scripts добавим следующие команды:
```
...,
scripts: {
  ...,
  "lint": "eslint \"{src,apps,libs,test}/**/*.ts\"",
  "lint:fix": "eslint \"{src,apps,libs,test}/**/*.ts\" --fix"
},
...
```
Обязательно установим новую зависимость (это плагин для eslint конфига):
```
pnpm add -D eslint-plugin-simple-import-sort
```

Далее займемся prettier файлом.

![Prettier changes](./images/prettierrc.png?raw=true "Prettier changes")

### ENV файлы
Для начала, добавим наш .env.example. В этом файле, мы примерно подскажем разработчикам, какие переменные можно конфигурить в проекте.
На данный момент, он выглядит у нас, вот так:
```
SA_KEY=path_to_service_file

MINI_APP_URL=domain_to_mini_app # оставим это, как задел на будущее
```

Обязательно настроим .gitignore файл, добавим в него следующие дополнения:
```
# all env files except example
.env*
!.env.example

# google service account file (by firebase)
service-account*.json

# firebase config file
.firebaserc

# tg library local cache file
sessions.json
```

[Очередной коммит с изменениями](https://github.com/Fedorrychkov/nestjs-startup-boilerplate/commit/0e3f830d2b55137a9e72e33b32673210cf273642). А если, мы еще и запустим команду ```pnpm run lint:fix```, то получим исправленные [по линтеру файлы](https://github.com/Fedorrychkov/nestjs-startup-boilerplate/commit/95a73d71132db09df39cef8a8734339893235cde).

И так, на данном этапе, базовый конфиг проекта готов. Далее нас ждет настройка firebase проекта через кабинет.

## Подготовка к работе с Firebase

Обычно в своих проекта я использую Firebase, как провайдер: Google Cloud Storage, Firestore Database, Firebase Auth. Давайте настроим первые две услуги.

Сперва открываем [консоль firebase проектов](https://console.firebase.google.com/u/2/). Если вы еще ни разу не работали с Firebase, у вас будет полупустой дашборд

![Empty Firebase Console](./images/firebase_main.png?raw=true "Empty Firebase Console")

Жмакайте на Get Started, приступим к созданию проекта. На 1 экране создания у вас попросят ввести название, я назову свой так: nestjs-boilerplate-example. Это название будет участвовать в будущих настройках env файлов проекта. Далее на 2 экране мне предлагают включить аналитику, мне она не нужна, я отказываюсь и создаю проект. После успешного создания у вас появится проект на дашборде + вы можете перейти к нему нажав на Continue в окне ожидания создания проекта.

В дашборде созданного проекта Вы увидите что-то наподобие этого.

![Firebase Project Console](./images/firebase_project_console.png?raw=true "Firebase Project Console")

В целом, пока все просто. Далее, нам нужно включить услуги: Firestore, Authentication, Storage. Эти разделы можно увидеть слева, в сайдбаре консоли, они хранятся в разделе Build.

![Firebase Project Build](./images/firebase_build_sidebar.png?raw=true "Firebase Project Build")

При открытии каждого раздела, первым делом вы увидите предложение о включении функциональности. Get Started, Create Database и так далее. До определенных тарифных лимитов использования, эти услуги бесплатные. [Подробнее о тарифах и лимитах](https://firebase.google.com/pricing)

Создание базы данных потребует выбора сервера базирования, в целом, можете выбрать любой удобный Вам, все зависит от распределения ваших пользователей, я выбираю обычно Европу (Eur3). + Мод запуска, можете оставить спокойно production режим для БД. *Примечание: я пробовал us-central1 и eur3, особой разницы скорости работы в рантайме не заметил.*

Ну вот, теперь у вас полностью готовый firebase проект. К слову, таким же образом вы можете спокойно генерировать N firebase проектов под необходимые контуры (Production/Stage/Local/Test, в целом для жизни моих проектов вполне хватает).

Давайте перейдем к следующему этапу. Нам нужно получить необходимые данные для запуска проекта. Для этого перейдите в ваш [Project Settings (Эта ссылка на мой проект, открыть вы его не сможете, но ссылку в качестве примера можете посмотреть)](https://console.firebase.google.com/u/2/project/nestjs-boilerplate-example/settings/general) (находится в поповере по клику на шестеренку).

Попав в настройки проекта, выберите вкладку, Service Accounts. И нажмите на кнопку, **generate new private key**. После этого Вы сможете скачаеть файл .json формата. Он нам понадобится для шага подключения к firebase.

## Конфигурация проекта для подключения к Firebase.

И так, давайте разберемся с переменными окружения.
Скаченный ранее service json файл из firebase, перенесите в корень проекта, и, например, назовите его service-account.json.
Название сервис файла занесите в ваш .env.dev файл, в поле ```SA_KEY=service-account.json```. Env.dev файл можно сделать на основе примера в ```.env.example```. И обновим команды в package.json, раздел scripts.
```
...
"scripts": {
  ...
  "build:prod": "NODE_ENV=production nest build",
  "start": "NODE_ENV=production nest start",
  "start:dev": "NODE_ENV=development nest start --watch"
  "start:prod": "NODE_ENV=production node dist/main"
  ...
}
...
```
По сути, мы лишь добавили NODE_ENV переменную. Далее, это значение можно будет достать из process.env.NODE_ENV. Остальные переменные окружений лучше пробрасывать через конкретные .env файлы. Так же для примера добавим сразу же .env файл (можете создать копию из .env.dev, он у нас будет для production режима в будущем)

Теперь подключим наших env переменные в коде проекта.
Нам понадобится 2 файла. app.module.ts и main.ts.
Начнем с src/env.ts файла, заполним его небольшой логикой и флагами из process.env.NODE_ENV:

```
const ENV = process.env.NODE_ENV

export const isDevelop = ENV === 'development'
export const isProduction = ENV === 'production'

export const getEnvFile = () => {
  if (isDevelop) {
    return '.env.dev'
  }

  return '.env'
}
```

*Примечание: переданный при запуске NODE_ENV можно сразу же достать, остальные значения будем подгружать при помощи ConfigService в AppModule.*

Еще нам потребуется библиотека для чтения значений и загрузки  env. ```pnpm add @nestjs/config```.
Теперь можем немного изменить app.module.ts:

```
import { Module } from '@nestjs/common'

import { AppController } from './app.controller'
import { AppService } from './app.service'
import { ConfigModule } from '@nestjs/config'
import { getEnvFile } from './env'

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      envFilePath: getEnvFile(),
    }),
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```
Таким образом env можно будет использовать по всему проекту.

Теперь можем добавить шаг подключения к firebase. Для этого нам понадобится установить пакет ```pnpm add firebase-admin```. Открываем main.ts файл и добавляем инициализацию firebase-admin.
```
import { ConfigService } from '@nestjs/config'
import { NestFactory } from '@nestjs/core'
import * as admin from 'firebase-admin'
import * as fs from 'fs'

import { AppModule } from './app.module'

async function bootstrap() {
  const app = await NestFactory.create(AppModule)
  const configService: ConfigService = app.get(ConfigService)
  // Получаем значение названия сервис файла из env файла
  const accountPath = configService.get<string>('SA_KEY')
  // Типизацию можно подтюнить под сервисный файл, как видите, с этим я пока не заморачивался
  const serviceAccount: any = JSON.parse(fs.readFileSync(accountPath, 'utf8'))

  const adminConfig: admin.ServiceAccount = {
    projectId: serviceAccount.project_id,
    privateKey: serviceAccount.private_key,
    clientEmail: serviceAccount.client_email,
  }

  admin.initializeApp({
    credential: admin.credential.cert(adminConfig),
    databaseURL: `https://${adminConfig.projectId}.firebaseio.com`,
  })

  // Значение порта так же можно вынести в env по необходимости, этот шаг я опустил
  await app.listen(8080)
}

bootstrap()
```

Итак, у нас готово [подключение к firebase](https://github.com/Fedorrychkov/nestjs-startup-boilerplate/commit/ba9bf254910bd62c3b8e73c795d1830677e4971b) проекту.

Однако это еще не все! Далее мы займемся:
- Модулем Firestore
- Примерами модели данных с контроллерами и сервисами
- Добавим gcloud bucket модуль, для работа с файлами

## Firestore Модуль
Начнем с самого модуля. Создадим папку src/providers/firestore. В этой папке у нас будет организован необходимый набор кода для подключения к Firestore коллекциям.

Сразу же можем установить пакет для firestore ```pnpm add @google-cloud/firestore```.

В папке firestore добавляем 4 файла:
- firestore.module.ts - для подключения в AppModule проекта
- firestore.providers.ts - для списка фаерстор entity документов
- types.ts - для типизации модуля
- index.ts чисто для красивого реэкспорта firestore модуля.

```
// firestore.providers.ts
export const FirestoreDatabaseProvider = 'firestoredb'
export const FirestoreOptionsProvider = 'firestoreOptions'
export const FirestoreCollectionProviders: string[] = [/* Далее будет необходимо добавлять классы документов коллекций firestore */]
```

```
// types.ts
// Добавляем тип, для типизации аргументов модуля
import { Settings } from '@google-cloud/firestore'

export type FirestoreModuleOptions = {
  imports: any[]
  useFactory: (...args: any[]) => Settings
  inject: any[]
}
```

```
// firestore.module.ts
// Здесь мы создаем свой модуль провайдер для firestore коллекций
import { Firestore } from '@google-cloud/firestore'
import { DynamicModule, Module } from '@nestjs/common'

import {
  FirestoreCollectionProviders,
  FirestoreDatabaseProvider,
  FirestoreOptionsProvider,
} from './firestore.providers'
import { FirestoreModuleOptions } from './types'

@Module({})
export class FirestoreModule {
  static forRoot(options: FirestoreModuleOptions): DynamicModule {
    const collectionProviders = FirestoreCollectionProviders.map((providerName) => ({
      provide: providerName,
      useFactory: (db) => db.collection(providerName),
      inject: [FirestoreDatabaseProvider],
    }))

    const optionsProvider = {
      provide: FirestoreOptionsProvider,
      useFactory: options.useFactory,
      inject: options.inject,
    }

    const dbProvider = {
      provide: FirestoreDatabaseProvider,
      useFactory: (config) => new Firestore(config),
      inject: [FirestoreOptionsProvider],
    }

    return {
      global: true,
      module: FirestoreModule,
      imports: options.imports,
      providers: [optionsProvider, dbProvider, ...collectionProviders],
      exports: [dbProvider, ...collectionProviders],
    }
  }
}
```

Осталось подключить модуль в app.module.ts

```
...
import { ConfigModule, ConfigService } from '@nestjs/config'
...
import { FirestoreModule } from './providers'
...

@Module({
  imports: [
    FirestoreModule.forRoot({
      imports: [ConfigModule],
      useFactory: (configService: ConfigService) => ({
        keyFilename: configService.get<string>('SA_KEY'),
      }),
      inject: [ConfigService],
    }),
  ],
  ...
})
```

На этом этапе, у нас в целом все готово для создания модулей с контроллерами, репозиториями и так далее. Вот [текущий набор правок](https://github.com/Fedorrychkov/nestjs-startup-boilerplate/commit/2163b22cb1270ecc254e603a332278a28fc93086).

## Example Module
Наконец то мы дошли до создания примера модуля. Мы реализуем базовый CRUD на примере абстрактной сущности example. Конкретно метод delete реализовывать я не стану, его реализация ничем не отличается от других вызовов, разве что вызывать в методах репозитория будем .delete(documentId) и все. Вместо удаления документа можно реализовать мягкое удаление, которыое под капотом будет менять поле status у документа (например status: ACTIVE/ARCHIVED). В нашем примере модуля этого не будет. В общем, начнем мы с того, что создадим src/modules, в котором и заведем example модуль. Эта сущность нужна только для примера реализации, как можно организовать код проекта. Само собой, Вы вольны делать что угодно и как угодно.

Предлагаю, например, такую структуру модулей.

```
src/modules
  - example
    - repositories
      - example.repository.ts
      - index.ts
    - controllers
      - example.controller.ts
      - index.ts
    - services
      - example.service.ts
      - index.ts
    - entities
      - example.document.ts
      - index.ts
    - dto
      - example.request.ts
      - example.response.ts
      - index.ts
    example.module.ts
helpers
  - time.ts
  - id.ts
  - index.ts
```

Приступим, сперва добавим helpers, ```pnpm add dayjs uuid```

```
// time.ts
import * as dayjs from 'dayjs'
import * as duration from 'dayjs/plugin/duration'
import * as isToday from 'dayjs/plugin/isToday'
import * as timezone from 'dayjs/plugin/timezone'
import * as utc from 'dayjs/plugin/utc'

// В целом можно обойтись без всего этого, тут уже зависит от вашего проекта, оставил в качестве примера
dayjs.extend(utc)
dayjs.extend(timezone)
dayjs.extend(isToday)
dayjs.extend(duration)

export const time = dayjs
```

```
// id.ts
import { v4 } from 'uuid'

const getUniqueId = (): string => v4()

export { getUniqueId }
```

Теперь займемся непосредственно файлами модуля Example. Менять название модуля и таблицы я не буду, но предлагаю использовать example, как некую абстракцию поста, включающую в себя название, текст, флаг опубликовано или нет и доп свойства, id документа и даты создания и последнего обновления.
Для дат, в firestore есть подходящая модель Timestamp, с ней достаточно легко работать и фильтровать по ней документы. В качестве id документа будем использовать uuid/v4 утилиту.

```
// example/entities/example.document.ts
import { Timestamp } from '@google-cloud/firestore'

export class ExampleDocument {
  static collectionName = 'example'

  id: string
  title: string
  text?: string | null
  isPublished: boolean
  createdAt?: Timestamp | null
  updatedAt?: Timestamp | null
}

// example/dto/example.filter.ts
export class ExampleFilter {
  public isPublished?: boolean
}

// example/dto/example.request.ts
export class ExampleRequestBody {
  public title: string
  public text?: string
}

// example/dto/example.response.ts
export class ExampleResponseItem {
  public id: string
  public title: string
  public text?: string | null
  public isPublished: boolean
  public createdAt?: string | null
  public updatedAt?: string | null
}
```

Добавим наш контроллер с методами:
- Получение списка с query параметрами для фильтрации examples по флагу isPublished и без
- Получение одного example документа по ID
- Создание example
- Редактирование документа
- Обновление флага isPublished (будет один метод, который переключает флаг без параметров).

```
// example/controllers/example.controller.ts
import { Body, Controller, Get, NotFoundException, Param, Patch, Post, Query } from '@nestjs/common'

import { ExampleFilter, ExampleRequestBody } from '../dto'
import { ExampleDocument } from '../entities'
import { ExampleService } from '../services'

@Controller('v1/example')
export class ExampleController {
  constructor(private readonly exampleService: ExampleService) {}

  @Get('/')
  async getList(@Query() query?: ExampleFilter): Promise<ExampleDocument[]> {
    const response = await this.exampleService.getList(query)

    if (!response?.length) {
      throw new NotFoundException('Examples are not exist')
    }

    return response
  }

  @Get('/:id')
  async get(@Param('id') id: string): Promise<ExampleDocument> {
    const response = await this.exampleService.getItem(id)

    if (!response) {
      throw new NotFoundException('Example does not exist')
    }

    return response
  }

  @Post('/')
  async create(@Body() body: ExampleRequestBody): Promise<ExampleDocument> {
    return this.exampleService.create(body)
  }

  @Patch('/:id')
  async update(@Param('id') id: string, @Body() body: ExampleRequestBody): Promise<ExampleDocument> {
    return this.exampleService.update(id, body)
  }

  @Patch('/publish/:id')
  async togglePublish(@Param('id') id: string): Promise<ExampleDocument> {
    return this.exampleService.togglePublish(id)
  }
}
```

Большая часть реализации делегирована сервису ExampleService, давайте добавим и его.

```
// example/services/example.service.ts

import { Injectable, NotFoundException } from '@nestjs/common'

import { ExampleFilter, ExampleRequestBody } from '../dto'
import { ExampleRepository } from '../repositories'

@Injectable()
export class ExampleService {
  constructor(private readonly exampleRepository: ExampleRepository) {}

  public async getList(filter: ExampleFilter) {
    return this.exampleRepository.find(filter)
  }

  public async getItem(id: string) {
    return this.exampleRepository.getDataByDocumentId(id)
  }

  public async create(body: ExampleRequestBody) {
    return this.exampleRepository.create(body)
  }

  public async update(id: string, body: ExampleRequestBody) {
    const { doc, data } = await this.exampleRepository.getUpdate(id)

    if (!doc || !data) {
      throw new NotFoundException('Example document does not exist')
    }

    const response = this.exampleRepository.getValidProperties({ ...data, ...body }, true)
    const changedKeys = Object.keys(body)
    const valuesToUpdate: Partial<ExampleRequestBody> = {}

    for (const key of changedKeys) {
      const newValue = response?.[key]
      const currentValue = doc?.[key]

      if (newValue !== currentValue) {
        valuesToUpdate[key] = newValue
      }
    }

    if (Object.keys(valuesToUpdate).length > 0) {
      await doc.update({ ...valuesToUpdate, updatedAt: response?.updatedAt })
    }

    return response
  }

  public async togglePublish(id: string) {
    const { doc, data } = await this.exampleRepository.getUpdate(id)

    if (!doc || !data) {
      throw new NotFoundException('Example document does not exist')
    }

    const newPublishedState = !data?.isPublished

    const response = this.exampleRepository.getValidProperties({ ...data, isPublished: newPublishedState }, true)

    await doc.update({ isPublished: newPublishedState, updatedAt: response?.updatedAt })

    return response
  }
}
```

Пока что, покрыт тайной лишь файл example.repository.ts, именно он отвечает за обращения в базу данных Firestore коллекции Example. Давайте добавим и его реализацию.

```
// example/repositories/example.repository.ts

import { CollectionReference, Query, Timestamp } from '@google-cloud/firestore'
import { Inject, Injectable, Logger } from '@nestjs/common'
import { getUniqueId, time } from 'src/helpers'

import { ExampleFilter } from '../dto'
import { ExampleDocument } from '../entities'

@Injectable()
export class ExampleRepository {
  private logger: Logger = new Logger(ExampleRepository.name)

  constructor(
    @Inject(ExampleDocument.collectionName)
    private collection: CollectionReference<ExampleDocument>,
  ) {}

  async getDataByDocumentId(id: string): Promise<ExampleDocument | null> {
    const snapshot = await this.collection.doc(id).get()

    if (!snapshot.exists) {
      return null
    } else {
      return snapshot.data()
    }
  }

  /**
  * Метод возвращающий значение документа и доп методы по работе с документом коллекции, нас будет интересовать метод .update(...props) 
  **/
  async getUpdate(id: string) {
    const doc = await this.collection.doc(id)
    const snapshot = await doc.get()

    if (!snapshot.exists) {
      return { doc: null, data: null }
    } else {
      return { doc, data: snapshot.data() }
    }
  }

  /**
  * По сути, именно этот метод у нас будет отвечать за добавление новых фильтров в запрос за коллекцией
  * На данный момент, использую только флаг isPublished
  **/
  private findGenerator(filter: ExampleFilter) {
    const collectionRef = this.collection
    let query: Query<ExampleDocument> = collectionRef

    if (typeof filter?.isPublished === 'boolean') {
      query = query.where('isPublished', '==', filter.isPublished)
    }

    return query
  }

  async find(filter: ExampleFilter): Promise<ExampleDocument[]> {
    const list: ExampleDocument[] = []
    const query = this.findGenerator(filter)

    const snapshot = await query.get()

    snapshot.forEach((doc) => list.push(doc.data()))

    return list
  }

  async create(payload: Pick<ExampleDocument, 'title'> & Partial<ExampleDocument>) {
    const validPayload = this.getValidProperties(payload)
    const document = await this.collection.doc(validPayload.id)
    await document.set(validPayload)

    return validPayload
  }

  /**
  * Этот метод нужен нам для подготовки данных перед записью в firestore
  * Документ на входе может быть без id и других полей, поэтому организуем фолбэк значения и установку свйоств времени работы с документом
  * Флаг newUpdatedAt используем для установки текущей даты в поле updatedAt
  **/
  public getValidProperties(
    document: Omit<ExampleDocument, 'id' | 'isPublished'> & { id?: string; isPublished?: boolean | null },
    newUpdatedAt = false,
  ) {
    const dueDateMillis = time().valueOf()
    const createdAt = Timestamp.fromMillis(dueDateMillis)

    return {
      id: document.id || getUniqueId(),
      title: document.title,
      text: document.text ?? null,
      isPublished: document.isPublished ?? false,
      createdAt: document.createdAt ?? createdAt,
      updatedAt: newUpdatedAt ? createdAt : (document.updatedAt ?? null),
    }
  }
}
```

Само собой в файле example/index.ts организуем реэкспорт модуля и документа.
```
// example/index.ts
export * from './entities'
export * from './example.module'
```

Теперь можем обновить файл firebase.providers.ts

```
// firebase.provider.ts
import { ExampleDocument } from 'src/modules/example'

...

// Таким образом, мы подключаем документ в список коллекций Firestore для работы с ExampleDocument
// В будущем сюда потребуется добавлять другие, новые коллекции firestore
export const FirestoreCollectionProviders: string[] = [ExampleDocument.collectionName]

```

Чтобы example модуль заработал, добавим его в app.module.ts файл.

```
...
import { ExampleModule } from './modules/example'
...

@Module({
  imports: [
    ...,
    ExampleModule,
  ],
  ...
})
export class AppModule {}
```

Кажется мы забыли еще кое-что. Работа с firebase без конфиг файлов firebase не совсем удобна, особенно если речь идет о различных контурах. Нужно будет добавить глобально firebase cli ```npm install -g firebase-tools```. После этого, скорее всего вам потребуется авторизоваться командой ```firebase login```. Как только вы пройдете все требуемые шаги, можем приступить к конфигурации работы с firebase из терминала.

В терминале, в корне вашего проекта, потребуется ввести команду ```firebase init```. Эта команда запустит процесс инициализации нового или созданного ранее проекта Firebase. Выберите, в моем случае, Firestore из предлагаемых пунктов. Далее, если вы как и я, уже создали firebse проект, выберите пункт ```Use an existing project```, найдите в предложенном списке ваш проект и выберите его. Продолжайте настройку, обычно далее ничего лишнего не нужно менять, дефолтные названия файлов можем оставить как есть.

*Примечание: Выбор осуществляется нажатием на пробел, а продолжение на enter/return (в зависимости от вашей клавиатуры). Опции на выбор представляются в виде полого круга (по сути, radio button).*

*Примечание 2: Иногда может быть такое, что требуемый проект не подтягивается, в таком случае, готовые файлы конфига можете взять из коммита, который будет в конце этой части настроек*.
___ 
Так же может возникнуть проблема, как в моем случае: *Error: It looks like you haven't used Cloud Firestore in this project before. Go to https://console.firebase.google.com/project/nestjs-boilerplate-example/firestore to create your Cloud Firestore database.* Она решается путем добавление платежного аккаунта. Если набрать команду ```firebase init --debug```, то можно увидеть конкретную ошибку.

Мой лог ошибки инициализации подключения к проекту без активного платежного аккаунта:

```
":{"code":403,"message":"Read access to project 'nestjs-boilerplate-example' was denied: please check billing account associated and retry","status":"PERMISSION_DENIED"}}
[2024-07-28T18:40:19.841Z] error getting database typeHTTP Error: 403, Read access to project 'nestjs-boilerplate-example' was denied: please check billing account associated and retry {"name":"FirebaseError","children":[],"context":{"body":{"error":{"code":403,"message":"Read access to project 'nestjs-boilerplate-example' was denied: please check billing account associated and retry","status":"PERMISSION_DENIED"}},"response":{"statusCode":403}},"exit":1,"message":"HTTP Error: 403, Read access to project 'nestjs-boilerplate-example' was denied: please check billing account associated and retry","status":403}
[2024-07-28T18:40:19.842Z] database_type: undefined
```

Установить платежный аккаунт можно на странице: [GCP Billing](https://console.cloud.google.com/billing?authuser=2)

После создания платежного аккаунта в GCP и привязки проекта к этому аккаунту, завершить конфигурацию удасться без лишних проблем.

![Firebase init selector](./images/firebase_init.png?raw=true "Firebase init selector")

После того, как вы пройдете все шаги, в проект добавятся такие файлы:
- .firebaserc
- firebase.json
- firestore.indexes.json - список актуальных индексов редактируем в этом файле
- firestore.rules - здесь будет код правила работы с firestore. Его значение мы не меняем в дашборде проекта

*Примечание: Вместо файла .firebaserc в гите мы оставляем его пример, .firebaserc.example. То, как организовать CI/CD для всего этого мы разберем в следующей статье про деплой бэкенда на VPS.*

В будущем нас будут интересовать только .firebaserc (в нем будет название firebase проекта) и firestore.indexes.json (в нем можно будет конфигурить ваши индексы и деплоить их в проект командой ```firebase deploy --only firestore:indexes```)

Итоговый набор изменений на данном этапе в [этом коммите](https://github.com/Fedorrychkov/nestjs-startup-boilerplate/commit/b4b656287f3df64e30e7d3638f9e9d1dd001665d).
___

На текущем этапе мы уже можем спокойно работать с проектом и добавлять необходимые нам индексы в Firestore и так далее. Однак это не все, например я, не могу жить без husky, дополнительные прекоммит хуки и многое другое можно настроить через него. Особенно это полезно в командной разработке, для дополнительного битья по рукам разработчиков. Вы смело можете пропустить следующий шаг, если вам это не нужно. В итоговом бойлерплейт репозитории уже будут добавлены все необходимые настройки для работы через husky. После настройки прекоммитов, мы обязательно дополнительно разберем работу текущего api/example и добавим необходимые индексы для коллекции example. Надеюсь вам еще не надоело, давайте продолжать!

# Продолжаем добавлять
Пройдя все шаги выше у нас есть:
- Рабочий бэкенд API, его уже можно поднять локально и потыкать api/example через curl или, например, Postman.
- Рабочий коннект к Firebase проекту и конфиг firebase/firestore

Впереди еще хочется обсудить:
- husky прекоммит хук для запуска линтера кода
- разберем текущий CRUD и добавим индексы
- добавим API для загрузки файлов в Google Cloud Bucket (он же Storage)

## Husky

Дока [Husky Get Started](https://typicode.github.io/husky/get-started.html)

В терминале запускаем команду ```pnpm add -D husky```, далее запускаем инициализацию командой ```npx husky init```, это добавит в проект .husky папку, + pre-commit файл, который будет запускаться на стадии коммита изменений. Давайте немного мутируем package.json новой командой в scripts.
```
...
"scripts": {
  ...,
  "prepare": "node .husky/install.mjs"
}
```
И доабавим файл [.husky/install.mjs](https://typicode.github.io/husky/how-to.html)

```
// Skip Husky install in production and CI
if (process.env.NODE_ENV === 'production' || process.env.CI === 'true') {
  process.exit(0)
}
const husky = (await import('husky')).default
console.log(husky())
```

Этот скрипт нужен для избежания ошибки установки husky, после команды установки зависимостей ```npm i/pnpm i```

Еще, нам нужно будет отредактировать файл .husky/pre-commit
```
pnpm run lint-staged && pnpm run lint:fix
```

Так как подразумевается запуск хука, который будет проверять файлы, нужно дополнительно установить пакет ```pnpm add -D lint-staged``` и добавить в package.json дополнительный конфиг.

```
...
"scripts": {
  ...,
  "lint-staged": "lint-staged --allow-empty",
},
"lint-staged": {
  "*.{js,jsx,ts,tsx}": [
    "pnpm lint --fix"
  ]
}
...
```

По итогу у нас соберется [такой коммит](https://github.com/Fedorrychkov/nestjs-startup-boilerplate/commit/534ed4df51f77229ae7b93b35c45dee507be743e).
Теперь, все будущие коммиты будут валидироватьcя по js(x)/ts(x) файлам на основе eslint/prettier и добавлять правки для будущего коммита при помощи команды, добавленной ранее ```lint:fix```. Можно расценивать данный пример прекоммит хука, как основу для ваших личных конфигураций.
Полный набор доступных гит хуков можно посмотреть в [githooks](https://git-scm.com/docs/githooks)

## Протестируем и соберем индексы для api/example

В app.module уже подключен пример рабочего контроллера в app.controller.ts, его можно вызвать запросом на урл http://localhost:8080 из браузера или Postman. Я буду использовать Postman для более удобной демонстрации.

![Postman Main Get Route](./images/postman_main_route.png?raw=true "Postman Main Get Route")

Давайте сделаем еще несколько запросов.

### Список example документов.

![Postman Example List Get Route](./images/postman_example_list.png?raw=true "Postman Example List Get Route")

В результате мы получим ожидаемую ошибку 404, об отсутствующих документах. (Это поведение изначально заложено в коде контроллера, конфигурировать можно как вам вздумается)

А теперь, давайте запросим список example с фильтром в query параметрах ?isPublished=<false или true>.

![Postman Example List Get Route With Wrong Filter](./images/postman_wrong_boolean_filter_example.png?raw=true "Postman Example List Get Route With Wrong Filter")

Так, мы послали запрос с параметром, однако получаем ту же ошибку. На самом деле это нормально, но есть проблема. На данный момент, наше апи не умеет читать и работает с boolean значениями, они отображаются в контроллерах и сервисах, как строковые значения 'true' | 'false'.
Если залогировать входящий аргумент query в контроллере GET v1/example, то мы увидим следующую картину
```
{ isPublished: 'false' }
```
Есть несколько способов, как решить эту проблему.
1. На уровне repository, в методе findGenerator в ручную приводить boolean строку к Boolean типу.
```
// Вместо
if (typeof filter?.isPublished === 'boolean') {
  query = query.where('isPublished', '==', filter.isPublished)
}

// Что то вроде такого
if (filter?.isPublished) {
  const isPublished = filter?.isPublished === 'true' ? true : false
  query = query.where('isPublished', '==', isPublished)
}
```
2. Добавить дополнительный шаг по их трансформации

Давайте попробуем пойти по 2 шагу. Отредактируем метод контроллера.

```
// example/controllers/example.controller.ts
import {
  ...,
  ParseBoolPipe,
  ...,
} from '@nestjs/common'
...
  @Get('/')
  async getList(@Query('isPublished', ParseBoolPipe) isPublished?: boolean): Promise<ExampleDocument[]> {
    const response = await this.exampleService.getList({ isPublished })
    // ...
  }
...
```

Таким образом, трансформируем входящий параметр isPublished в boolean значение.

### Создание нового example документа

![Postman Example Create](./images/postman_example_create.png?raw=true "Postman Example Create")

Я создал несколько документов, для примера. Давайте еще раз запросим наш список, с фильтром по isPublished=false


![Postman Example Unpublished list](./images/postman_unpublished_example_list.png?raw=true "Postman Example Unpublished list")

Если присмотреться, можно увидеть, что список подтягивается корректно. Однако для многих коллекций требуется менять направления или гарантировать возвращение списка, например, с учетом даты создания по descending/ascending значениям.

```
// repositories/example.repository.ts

...
  async find(filter: ExampleFilter): Promise<ExampleDocument[]> {
    ...
    let query = this.findGenerator(filter)

    query = query.orderBy('createdAt', 'desc')
    ...
  }
...
```

После попытки запросить метод списка еще раз, мы увидим, что ответ изменился на ошибку 500. Если пойти в консоль, то мы увидим, что firestore выкинул ошибку отстутствия индекса на такой запрос спика.

```
9 FAILED_PRECONDITION: The query requires an index. You can create it here: <url>
```

Смело переходите по этому URL в консоль проекта, там вы увидите предложение добавления нового индекса. Не спешите добавлять его от туда (вы можете запускать создание индексов и из консоли, но я бы хотел делать это через файл firestore.indexes.json)

![Firebase index creation modal](./images/firebase_index_creation.png?raw=true "Firebase index creation modal")

Нам нужно взять данные из этой модалки и вручную завести новый индекс, будет это выглядеть так:

```
// firestore.indexes.json
{
  "indexes": [
    {
      "collectionGroup": "example",
      "queryScope": "COLLECTION",
      "fields": [
        {
          "fieldPath": "isPublished",
          "order": "ASCENDING"
        },
        {
          "fieldPath": "createdAt",
          "order": "DESCENDING"
        }
      ]
    }
  ],
  "fieldOverrides": []
}
```

Поле ```__name__``` не нужно указывать в этом конфиге, при деплое, он добавляется автоматически. Еще важно соблюдать очередность полей (fields).

После редактирования файла, смело запускайте команду деплоя ```firebase deploy --only firestore:indexes```

После запуска, в той же консоле фаербейза, во вкладке indexes вы увидите свой индекс, со статусом Building..., необходимо дождаться его сборки и вновь сделать запрос за списком.

### Получение Example по ID
Теперь мы можем проверить метод получения example документа по id, скрин вставлять не буду, так как этот запрос уже должен работать без проблем. В моем случае, это ```http://localhost:8080/v1/example/7c8a5d30-beca-409a-8509-873616c80f5a```, Ваш ID может отличаться от моего.

### Редактирование Example документа по ID
Проверим метод редактирования example документа, я отредактирую title.

![Postman edit example](./images/postman_edit_title_example.png?raw=true "Postman edit example")

Если снова запросить список или документ по id, мы так же увидим измененные данные.

### Смена флага isPublished
Помимо этого, давайте поменяем значение isPublished в документе, сделав запрос на еще один endpoint.

![Postman toggle publis example](./images/postman_toggle_publis.png?raw=true "Postman toggle publis example")

Если вы проверите состояние списка isPublished=false или true, то увидите изменения в возвращаемых данных.

Так же прикрепляю ссылку на текущий набор api вызовов в Postman [json файлик](./common/postman_example.json). Можете скачать его и импортировать в свой Postman workspace для быстрой развертки окружения запросов.

Очередной [коммит](https://github.com/Fedorrychkov/nestjs-startup-boilerplate/commit/81e8040835da547f5f6a2367f51f5104f1ee64f4) с изменениями. На данном этапе я мог бы остановиться, но мы еще не разобрали момент с публикацией файлов в GCloud Storage...

## Storage Bucket

Давайте определим заранее моменты работы со Storage, которые мне известны.
- Nest.js предоставляет [документацию по загрузке файлов](https://docs-nestjs.netlify.app/techniques/file-upload)
- Бесплатный Storage предоставляет 5GB хранилища, сверх этого объема придется платить ежемесячно за каждый байт данных.
- Бесплатный Spark не дает возможности создавать отдельные бакеты, но такой кейс мы учтем в коде. Однако рекомендую использовать дефолтный бакет проекта, а сами файлы резолвить по папкам и подпапкам в коде.

Кейсы использования Storage
- Чтение и запись файла во временную папку в корне проекта, /uploads в нашем случае
- Запись и удаление файла в Storage
- Генерация публичной ссылки на файл, если такая необходимость нужна (по дефолту мы всегда будем отдавать публичный линк, вы можете переписать или дополнить необходимый кусок кода относительно ваших кейсов, я предоставлю рабочий пример)

Давайте приступим, нас ждет еще N новых файлов.
В папке privders, рядом с firebase, добавим новую папку - bucket.
В ней мы заведем, само собой файл bucket.module.ts и кучу вспомогательных. Кстати, чуть не забыл, установим пакеты ```pnpm add @google-cloud/storage multer lodash```, и обязательно ```pnpm add -D @types/multer```.

Предлагаемая структура:

```
src/providers
  - bucket
    - providers
      default.bucket.ts
      index.ts
    bucket.constants.ts
    bucket.module.ts
    bucket.providers.ts
    bucket.shared.service.ts
    bucket.types.ts
    index.ts
    utils.ts
```

Давайте опишем каждый файл, кода будет относительно немного. И добавим загрузку и удаление в отдельный ednpoint работы с изображениями через api/example/:id/image. Начнем со вспомогательных файлов.

```
// default.bucket.ts
export class DefaultBucketProvider {
  static bucketName = 'default'
}

// bucket.constants.ts
export const getDefaultOptions = (role: string) => ({
  entity: 'allUsers',
  role: role,
})
```

bucket.providers.ts представляет из себя ту же концепцию, что и файл firestore.providers.ts.
```
// bucket.providers.ts
import { DefaultBucketProvider } from './providers'

export const StorageBucketsProvider = 'StorageBucketsProvider'
export const StorageOptionsProvider = 'StorageOptionsProvider'

export const StorageBucketProviders: string[] = [DefaultBucketProvider.bucketName]

// bucket.types.ts
import { Bucket, Storage } from '@google-cloud/storage'

export type StorageProps = {
  keyFilename: string
}

export type FirestoreModuleOptions = {
  imports: any[]
  useFactory: (...args: any[]) => StorageProps
  inject: any[]
}

export type BucketProvider = {
  bucket: Bucket
  storage: Storage
}
```

Наш сервис для работы с Bucket.
```
// bucket.shared.service.ts
import { Bucket } from '@google-cloud/storage'
import { Logger } from '@nestjs/common'
import { extname } from 'path'

export class BucketSharedService {
  private bucket: Bucket
  private logger: Logger

  constructor(bucket: Bucket, logName?: string) {
    this.bucket = bucket
    this.logger = new Logger(`${BucketSharedService.name}_${logName}`)
  }

  public async isFileExists(name: string) {
    try {
      const fileName = name
      const file = this.bucket.file(fileName)

      const [isExists] = await file.exists()

      return isExists
    } catch (error) {
      throw error
    }
  }

  public async deleteFileByName(path: string, folderPath: string) {
    return new Promise(async (resolve, reject) => {
      const pathes = path?.includes('%2F') ? path?.split('%2F') : path?.split('/')
      const fileName = pathes?.[pathes?.length - 1]
      const file = this.bucket.file(`${folderPath ? `${folderPath}/` : ''}${fileName}`)

      const [isExists] = await file.exists()

      if (!isExists) {
        reject(new Error('File does not exist'))
      }

      /**
       * Нет гарантий, что он наверняка удаляет, но вроде бы файлы становятся не доступными к поиску по их ссылкам
       */
      file
        .delete()
        .then((res) => {
          resolve(res)
        })
        .catch((err) => {
          this.logger.error('Error with file bucket removing', err?.message)
          reject(err)
        })
    })
  }

  /**
   * Using with internal upload folders
   */
  public async saveFileByUploadsFolder(definedFile: Express.Multer.File, folderPath?: string): Promise<string> {
    const uniqueSuffix = `${folderPath || 'main'}/${Date.now()}-${Math.round(Math.random() * 1e9)}`
    const fileName = `${uniqueSuffix}${extname(definedFile.path)}`

    return new Promise((resolve, reject) => {
      this.bucket
        .upload(definedFile.path, {
          destination: fileName,
        })
        .then((response) => {
          const [uploadedFile] = response || []
          const file = this.bucket.file(uploadedFile?.metadata?.name)

          file.makePublic(async (err) => {
            if (err) {
              this.logger.error(`Error making file public: ${err}`)
              reject(err)
            } else {
              this.logger.log(`File ${file.name} is now public.`)
              const publicUrl = file.publicUrl()
              this.logger.log(`Public URL for ${file.name}: ${publicUrl}`)
              resolve(publicUrl)
            }
          })

          return true
        })
        .catch((err) => {
          reject(err)
        })
    })
  }

  /**
   * Using without multer storage option, only memory buffer
   */
  public async saveFileByUrlAndBuffer(path: string, folderPath: string, buffer: Buffer): Promise<string> {
    return new Promise(async (resolve, reject) => {
      const uniqueSuffix = `${folderPath || 'main'}/${Date.now()}-${Math.round(Math.random() * 1e9)}`
      const fileName = `${uniqueSuffix}${extname(path)}`
      const file = this.bucket.file(fileName)
      await file.save(buffer)

      file.makePublic(async (err) => {
        if (err) {
          this.logger.error(`Error making file public: ${err}`)
          reject(err)
        } else {
          this.logger.log(`File ${file.name} is now public.`)
          const publicUrl = file.publicUrl()
          this.logger.log(`Public URL for ${file.name}: ${publicUrl}`)
          resolve(publicUrl)
        }
      })
    })
  }
}
```
Добавляем модуль с логикой подключения к Storage.
```
// bucket.module.ts
import { Storage } from '@google-cloud/storage'
import { DynamicModule, Module } from '@nestjs/common'
import * as fs from 'fs'

import { getDefaultOptions } from './bucket.constants'
import { StorageBucketProviders, StorageBucketsProvider, StorageOptionsProvider } from './bucket.providers'
import { FirestoreModuleOptions, StorageProps } from './bucket.types'

@Module({})
export class BucketModule {
  static forRoot(options: FirestoreModuleOptions): DynamicModule {
    const bucketProviders = StorageBucketProviders.map((providerName) => ({
      provide: providerName,
      useFactory: async (storage: Storage) => {
        console.log(storage, 'storage')
        /**
         * Use default bucket name
         */
        const bucket = storage.bucket(providerName === 'default' ? `${storage.projectId}.appspot.com` : providerName)

        const [isExist] = await bucket.exists()

        /**
         * Basic steps to create public bucket with writer rules, available only for BLAZE price
         */
        if (!isExist) {
          const options = getDefaultOptions(storage.acl.WRITER_ROLE)

          await bucket.create().catch((err) => console.error(`bucket ${providerName} creation get error`, err))

          console.info(`bucket ${providerName} created successfully`)

          bucket.acl.add(options, (err) => {
            if (!err) {
              console.info(`acl added successfully to ${providerName} bucket`)
            } else {
              console.error(`bucket ${providerName} error`, err)
            }
          })
        }

        return { bucket, storage }
      },
      inject: [StorageBucketsProvider],
    }))

    const optionsProvider = {
      provide: StorageOptionsProvider,
      useFactory: options.useFactory,
      inject: options.inject,
    }

    const provider = {
      provide: StorageBucketsProvider,
      useFactory: (config: StorageProps) => {
        const serviceAccount: { project_id?: string } = JSON.parse(fs.readFileSync(config.keyFilename, 'utf8'))

        return new Storage({ ...config, projectId: serviceAccount.project_id ?? '' })
      },
      inject: [StorageOptionsProvider],
    }

    return {
      global: true,
      module: BucketModule,
      imports: options.imports,
      providers: [optionsProvider, provider, ...bucketProviders],
      exports: [provider, ...bucketProviders],
    }
  }
}
```
Далее нам нужна конфигурация для выгрузки файлов из API во временную папку /uploads.
```
// utils.ts
import { diskStorage } from 'multer'
import { extname } from 'path'

export const storage = diskStorage({
  destination: './uploads',
  filename: (_, file, cb) => {
    // Предлагаемая регенерация имени файла
    const uniqueSuffix = `${Date.now()}-${Math.round(Math.random() * 1e9)}`
    cb(null, `${uniqueSuffix}${extname(file.originalname)}`)
  },
})
```

Константа ```storage``` понадобится нам, чтобы не перегружать memory при работе с загружаемым изображением. Если не использовать diskStorage, то мы можем столкнуться с нехваткой оперативной памяти при росте трафика.

```
// index.ts - для коротких импортов
export * from './bucket.module'
export * from './bucket.shared.service'
export * from './bucket.types'
export * from './providers'
export * from './utils'
```

Чтобы bucket модуль успешно запустился, нужно импортировать его в app.module.ts.

```
// app.module.ts
...

@Module({
  imports: [
    ...
    BucketModule.forRoot({
      imports: [ConfigModule],
      useFactory: (configService: ConfigService) => ({
        keyFilename: configService.get<string>('SA_KEY'),
      }),
      inject: [ConfigService],
    }),
  ],
  ...
})
export class AppModule {}
```

Добавим helper для валидации файлов. В нем мы опишем разрешенные расширения файлов и размеры.

```
- helpers
  ...
  - fileValidation
    constants.ts
    utils.ts
    index.ts
```

```
// constants.ts
export const listDefaultImageExt = 'image/png,image/jpeg,image/webp'

export const listPngAndJpegImageExt = 'image/png,image/jpeg'

export const IMG_MAX_SIZE_IN_BYTE = 716800 // 700kb

export const IMG_MAX_1MB_SIZE_IN_BYTE = 1048576 // 1mb

export const IMG_MAX_5MB_SIZE_IN_BYTE = 1048576 * 5 // 1mb

// utils.ts
import { reduce } from 'lodash'

export const getFileTypesRegexp = (ext: string): string => reduce(ext.split(','), (acc, key) => `${acc}|${key}`)
```

Теперь займемся контроллером и методом работы с изображением, по пути, добавим новый параметр в example.document.ts и example.repository.ts, детальные изменения можно будет посмотреть в коммите, в конце этой части.

```
// example/controllers/example.controller.ts
...

@Post('/:id/image')
@UseInterceptors(FileInterceptor('file', { storage, limits: { files: 1 } }))
async updateExampleImage(
  @UploadedFile(
    new ParseFilePipe({
      validators: [
        new MaxFileSizeValidator({ maxSize: IMG_MAX_1MB_SIZE_IN_BYTE }),
        new FileTypeValidator({ fileType: getFileTypesRegexp(listPngAndJpegImageExt) }),
      ],
    }),
  )
  file: Express.Multer.File,
  @Param('id') id: string,
): Promise<ExampleDocument> {
  return this.exampleService.updateImage(id, file)
}
...
```

```
// example/services/example.service.ts
...
export class ExampleService {
  private bucketService: BucketSharedService

  constructor(
    private readonly exampleRepository: ExampleRepository,
    @Inject(DefaultBucketProvider.bucketName)
    private readonly bucketProvider: BucketProvider,
  ) {
    this.bucketService = new BucketSharedService(this.bucketProvider.bucket, ExampleService.name)
  }
...
  public async updateImage(id: string, file: Express.Multer.File) {
    try {
      const { doc, data } = await this.exampleRepository.getUpdate(id)

      if (!doc || !data) {
        throw new NotFoundException('Example document does not exist')
      }

      const imageUrl = await this.bucketService.saveFileByUploadsFolder(file, `example/${data?.id}`)

      // Можно добавить удаление через доп проверку if (data?.imageUrl, но в целом это не обязательно, для кейса, когда файла нет, мы обязательно оборачиваем вызов метода удаления в trycatch блок)
      try {
        /**
         * Try to remove previously file
         */
        await this.bucketService.deleteFileByName(data?.imageUrl, `example/${data?.id}`)
      } catch {}

      const response = this.exampleRepository.getValidProperties({ ...data, imageUrl }, true)

      await doc.update({ imageUrl, updatedAt: response?.updatedAt })

      /**
       * Need for deletion uploads file by file.path
       */
      fs.unlinkSync(file.path)

      return response
    } catch (error) {
      /**
       * Need for deletion uploads/ file path
       */
      fs.unlinkSync(file.path)

      throw error
    }
  }
...
}
```

В целом, по загрузке файлов все. Мы добавили простой пример загрузки одного изображения в документ example. Давайте совершим запрос и проверим, что все работает корректно.

![Postman upload image example](./images/postman_example_upload_image.png?raw=true "Postman upload image example")

Можно заметить, что в модель добавилось поле imageUrl с установленной ссылкой из GCloud Storage. Полный набор изменений можно найти в [этом коммите](https://github.com/Fedorrychkov/nestjs-startup-boilerplate/commit/e0e0f789b9459a55729dde26a7a06f3dbe1cd513).

# Заключение
Надеюсь, что в данной статьей мы достигли намеченной цели, описать минимальный конфиг проекта с примером структуры кода, работы с Firestore и GCloud Bucket. Полный пример nestjs проекта можно найти у меня на [GitHub](https://github.com/Fedorrychkov/nestjs-firebase-startup-boilerplate). Надеюсь, что я смог доступно и с достаточным количеством кода описать основные шаги по генерации CRUD приложения. Я бы не хотел останавливаться на этой статье. Буду рад получить обратную связь и критику. Итоговая репа может послужить не самым плохим примером для Вашего быстрого старта разработки MVP или пет проекта на Nest.js в связке с Firebase или любым другим PaaS решением.

В следующих статьях мы вернемся к этому бойлерплейту и попробуем сделать больше:
- Реализуем методы работы с авторизацией и аутентификацией в Firebase в Nest.js API (На базе этой статьи и текущего example проекта).
- Добавим Swagger, для удобного просмотра контрактов и тестирования API.
- Немного разберем деплой получившегося backend приложения и настроим нотификации в командный чат Telegram.
- Попробуем написать Telegram Bot на основе nestjs-startup-boilerplate.
- Создадим Mini-App в связке с получившимся Telegram ботом.
