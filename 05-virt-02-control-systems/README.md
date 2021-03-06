### 5.2. Системы управления виртуализацией
---
Задача 1
Выберите подходящую систему управления виртуализацией для предложенного сценария. Детально опишите ваш выбор.

Сценарии:

- 100 виртуальных машин на базе Linux и Windows, общие задачи, нет особых требований
Преимущественно Windows based инфраструктура, требуется реализация программных балансировщиков нагрузки, репликации данных и автоматизированного механизма
создания резервных копий - подойдут все системы виртуализации VmWare vSphere, Hyper-v, KVM, Xen с NetBSD модулем, OpenStack, а так же облачные системы AWS.
- Требуется наиболее производительное бесплатное opensource решение для виртуализации небольшой (20 серверов) инфраструктуры Linux и Windows виртуальных машин - KVM, Xen, OpenStack.
- Необходимо бесплатное, максимально совместимое и производительное решение для виртуализации Windows инфраструктуры - KVM, Xen с NetBSD модулем.
- Необходимо рабочее окружение для тестирования программного продукта на нескольких дистрибутивах Linux - VmWare vSphere, Hyper-v, KVM, Xen, OpenStack, AWS.

---
Задача 2
Опишите сценарий миграции с VMware vSphere на Hyper-V для Linux и Windows виртуальных машин. Детально опишите необходимые шаги для использования всех преимуществ
Hyper-V для Windows.

В базе знаний VmWare указано, что надо использовать VMware vCenter Converter - 
https://docs.vmware.com/en/vCenter-Converter-Standalone/6.2/com.vmware.convsa.guide/GUID-9D2DE0FE-678C-4A30-90E6-060DF4217586.html

Попытался сравнить VmWare и Hyper-v, особых преимущетсв последней системы, за исключением плюсов наличия большего количества драйверов на хостовой машине при использовании паравиртуализации (и то не уверен в этом преимущетсве), не вижу.

---
Задача 3
Опишите возможные проблемы и недостатки гетерогенной среды виртуализации (использования нескольких систем управления виртуализацией одновременно) и
что необходимо сделать для минимизации этих рисков и проблем. Если бы у вас был бы выбор, то создавали ли вы бы гетерогенную среду или нет? Мотивируйте ваш ответ примерами.

Очевидный минус это трудозатраты, необходимо иметь несколько команд, каждая из которых будет специализироваться на обслуживании одной конкретной системы. Это повлечет за собой
финансовые затраты.
Плюс - гибкость и универсальность, к примеру не все аппратные контроллеры поддреживаются VmWare, имея альтернативу мы можем на этих серверах развернуть другую систему.
Думаю создавал бы, на мой взгляд матчится на одной системе в большой инфраструктуре нельзя.
