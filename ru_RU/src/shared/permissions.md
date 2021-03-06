# Permissions
Блок отвечающий за работу с системными разрешениями на использование ресурсов, компонентов или оборудования, в этой главе документации будут описаны принципы инициализации и работы с разрешениями.

Блоки разрешений действуют по принципу прохождения каждого компонента по шлюзовой системе, что подразуемевает что на каждый вызов функции компонентабудет производится верификация через middleware permissions manager-а, что гарантирует нам то что вызываемый нами публичный код отрабатывает функционально каждый слой вызовов. Как уточнение для последующих разраотчиков предпологается в информационном гайде обозначить способы при которых перед каждым запросом пользователь должен провести проверку на вхождение по причине того что все компоненты по умолчанию являются отзываемыми.

*Устарело, оставить для описания совместимости и различных вариаций*
### Sinopsis
Блок системных зависимостей относительно использования в системе в первую очередь касается компонентов, и их областей работы и предоставляемого ими функционала. Запрос на использование защищенных компонентов делится на несколько разных видов:

- Статические
- Динамические (Ленивые)
- Отзываемые статические

Способы работы с инициализацией влияют на поведение и принципы работы с блоком, и работают в разных областях, далее будут перечислены особенности и способы инициализации.

#### Статические
Статические зависимости инициализируются на этапе заполнения манифеста, с набором гарантированных разрешений, предупреждение о использовании которых производится единожды во время установки приложения, например к таким можно причислить компоненты:

- Storage
- IPFS
- P2P

По причине того что предупреждение о их использовании отражено в описании приложения, и устанавливая приложение пользователь соглашается на то что приложение может иметь постоянный доступ к ним без дополнительных запросов в сервис безопасности.

#### Динамические
Динамические системные зависимости работают через сервис безопасности и дополнительных ленивых запросах, которые дополнительным образом вне зоны DApp будут уточнять у пользователя разрешение на еденичное использование компонента, включая параметры с которыми ему разрешено работать и какой промежуток времени.

Для примера по такому принципу работает компонент Keychain, который спрашивает о использовании и ключах у пользователя через еденичные запросы которые он отправляет для получения экземпляра ключа которым он может вызвать функцию подписи. 

#### Отзываемые статические
Вид разрешений которые пользователь может отозвать в процессе использования приложения, к таким компонентам можно отнести например компонент `Notification` доступ к которому можно запросить на этапе установки приложения добавив его в манифест, при этом в последующем пользователь по собственному желанию может выключить отозвав его доступ.