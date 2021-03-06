# Зависимости
Блок документации отвечающий за управление зависимостями DApp относительно использования сервисов предоставляемых прочими DApp.

### Sinopsis
Блок управления зависимостями необходим для управления жизненным циклом работы приложения, того как он прогружает блоки зависимостей и производит инициалищзацию билдер объекта для создания асинхронного прокси канала.

Виды зависимостей, различаемые по методам загрузки и инициализации:
- Обязательные (required)
- Необязательные (optional)

С учетом нашего текущего рантайма, мы не можем использовать сервисы как самодостаточные процессы демоны, которые будут иметь свой собственный жизненный цикл обработки событий не использующий при этом инициализацию нового WebView. В следствии с этим сервисы являющиеся частью системы должны быть компонентами привязанными с "прожиганием" предварительным стороннего DApp в фоновом режиме перед запуском целевого приложения. Далее будут приведены подходы к инициализации зависимости и поведения основного рантайма.

#### Обязательные зависимости
Обязательные зависимости обрабатываются по принципу полной предзагрузки относительно приложения, в которой проверяется отработка и откликается ли приложение на пинг сообщения.

_Lifecycle:_
- Проверка зависимостей
- Запуск в фоновом режиме всех необходимых зависимостей
  - Если сервис уже существует как реальная задача (например приложение уже открыто и просто свернуто), то задачу считать запущенной и продолжить инициализации зависимостей.
  - Если задача не существует, создать и запустить новый экземпляр процесса, с работой в фоновом режиме.
  - Отправить в запущенный экземпляр ping сигнал для проверки работоспособности зависимости
- После запуска всех зависимостей запустить основное приложение.

_Примечание_
В случае если приложение зависимость уже запущено, и оно прошло проверку на работоспособность, нужно учитывать что даже после "закрытия" этого приложения, полностью закрывать его экземпляр нельзя потому что на него все еще действует активная "ссылка" из нашего основного запущенного приложения, поэтому это приложение должно висеть до тех пор пока мы используем основное приложение которое зависит от процесса.

В таком раскладе также следует учитывать что если мы хотим статичное время жизни процесса зависимости, например - создать еще один экземпляр этого приложения который мы не показываем мы добавляем очень большой расход ресурсов, потому что одна вкладка blink в среднем, в запущеном состоянии использует около 200мб оперативной памяти и добавляет в шедулер минимум 3 дополнительных потока. К тому же в таком случае нам будет куда сложнее управлять межпроцессным взаимодействием за счет того что нам потребуются дополнительные обработчики которые должны отслеживать работу статичных сервисов.

### Необязательные зависимости
В случае необязательных зависимостей, цикл инициализации работает по стандартному принципу - будет запущено основное приложение с его базовым контекстом, в случае необязательных зависимостей мы обрабатываем наши зависимости лениво в случаях когда они нам действительно нужны. Таким образом мы можем оптимизировать используемые ресурсы для работы с сервисами которые по умолчанию не нужны нашему DApp. Ленивая инициализация запускает "прожиг" зависимости в момент когда она ему потребуется.

Например:
```js
Dependencies
  // Запускаем "прожиг"
  .LazyInit("dapp_name.service_name")
  
  // Отрабатывает промис после инициализации сервиса
  .then((InitializedService) => {
    // Работа с ленивым сервисом после его инициализации
  })
```


## Версионирование
При загрузке приложения зависимости, мы имеем возможность подгружать несколько типо зависимостей, которые декларируются в манифесте приложения:

- Dependencies - прочие dapp от которых зависит наше приложение, с их версией, которую можно указать как абсолютно точную, так и относительную по спецификации [gemfile](https://guides.rubygems.org/patterns/#declaring_dependencies). При расхождении версии с уже установленным DApp с тем же названием, выводить предупреждение до установки - если пользователь подтверждает установку, мы устанавливаем dapp зависимость указанной версии и замещаем ей текущий dapp.
- Libraries - зависимости которые мы загружаем из IPFS, файлы JS которые мы инициализирует и проводим инъекцию из IPFS после предварительной загрузки. 

### Дополнительные примечания (Marketplace)
В магазине приложений, при установке приложения с зависимостью обязательной и необязательной, показывается уведомление о зависимостях и том что они будут установлены вместе с приложением для его работоспособности.