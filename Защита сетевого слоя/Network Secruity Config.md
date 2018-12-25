# Network Security Config

Начиная с Android 7, появилась возможность управлять настройками сетевой безопасности без изменения исходного кода приложения. Представленный механизм получил название [**Network security configuration**](https://developer.android.com/training/articles/security-config).

Основные возможности:

* Использование сторонних, в том числе самоподписанных сертификатов для сетевых соединений
* Отдельная сетевая конфигуация для debug-варианта прилиложения
* Полный запрет незашифрованного (cleartext) трафика для приложения
* Прикрепление (pinning) сертификатов

## Зачем это все нужно

Приходит как-то тестировщик к разработчику и говорит:

\- Не могу тестировать! Трафик на Charles не показывается!  
\- Как это не показывается?  
\- Откуда мне знать, на этом телефоне все норм, а на этом нет. Сертификаты проверил.  

---

Именно в этом момент появляется необходимость добавить в проект конфигуационный файл для NSС.

## Базовая настройка

Для начала необходимо создать файл `network_security_config.xml` в каталоге `res/xml` с таким содержимым:

```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <base-config cleartextTrafficPermitted="false">
        <trust-anchors>
            <certificates src="system" />
        </trust-anchors>
    </base-config>

    <debug-overrides cleartextTrafficPermitted="false">
        <trust-anchors>
            <certificates src="system" />
            <certificates src="user" />
        </trust-anchors>
    </debug-overrides>

    <domain-config cleartextTrafficPermitted="false">
        <domain includeSubdomains="true">example.com</domain>
        <pin-set>
            <pin digest="SHA-256">fwza0LRMXouZHRC8Ei+4PyuldPDcf3UKgO/04cDM1oE=</pin>
        </pin-set>
    </domain-config>
</network-security-config>
```

Теперь необходимо добавить ссылку на этот файл в `AndroidManifest.xml` следующим образом:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest ... >
    <application android:networkSecurityConfig="@xml/network_security_config"
                    ... >
        ...
    </application>
</manifest>
```

Формат NSC файла достаточно интуитивный и может быть использован в приведенном виде в большинстве проектов. Для этого нужно будет менять только доменное имя хэш ключа. Для реализации каких-то специфических сценариев потребуется изучить [полный формат файла](https://developer.android.com/training/articles/security-config#FileFormat).