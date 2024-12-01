# Лабораторная работа №5* 

## Задание
Настроить алерт кодом IaaC (например через конфиг алертменеджера, главное - не в интерфейсе графаны:), показать пример его срабатывания. Попробовать сделать так, чтобы он приходил, например, на почту или в телеграм. Если не получится - показать имеющийся результат и аргументировать, почему дальше невозможно реализовать.

## Ход выполнения работы 
Как и любую другую работу, эту я начинаю с подготовки окружения. Нужно установить Алертменеджера, а сделать это можно с помощью:

    helm install alertmanager prometheus-community/alertmanager

Поглядим, что все установилось: 

![photo_2024-12-01_14-59-14](https://github.com/user-attachments/assets/6c1d82b3-f266-4bd7-9189-7c55830b6037)

Пробросим порты с помощью 

    kubectl port-forward service/prometheus-alertmanager 9093:9093 -n monitoring

Ну и посмотрим на интерфейс Алертменеджера

![photo_2024-12-01_14-59-22](https://github.com/user-attachments/assets/cd63c5ec-801b-4746-916f-524c733ffca3)

Я попробовала настроить алерты на почту, но они не приходили(( 

Но мы не расстраиваемся, а идем делать через тг! Подготавливаем бота. Использовать буду базу - BotFather. Создаю бота парой команд и получаю токен. 

![photo_2024-12-01_14-59-31](https://github.com/user-attachments/assets/a049d409-db29-4c4f-8d49-9d74b1ccd0f7)

По токену иду смотреть ChatID. Изначально я хотела сделать сообщения в ЛС, но бот со мной дружить отказался, поэтому все сообщения будут в чатике, его id и забираем.

![Снимок экрана 2024-12-01 150815](https://github.com/user-attachments/assets/c09add18-1242-4dfb-89b8-fd3b545478f1)

Теперь идем писать конфиг для нашего алертменеджера. Так как приложение (если можно так сказать) у меня самое банальное, то и метрик я подобрать показательных не смогла, поэтому будет использование памяти) Итак, конфиг получился таким:

    alertmanager:
      config:
        global:
          resolve_timeout: 1m
          telegram_api_url: "https://api.telegram.org"

        route:
          receiver: telegram

        receivers:
          - name: telegram
            telegram_configs:
              - chat_id: *айди полученный в прошлом шаге*
                bot_token: *токен*
                send_resolved: true
                parse_mode: Markdown
                message: |-
                  {{ range .Alerts }}
                    *{{ .Annotations.summary }}*
                    {{ .Annotations.description }}
                  {{ end }}

                
    serverFiles:
      alerting_rules.yml:
        groups:
          - name: alerts
            rules:
              - alert: HighMemoryCache
                expr: container_memory_cache > 150000000
                for: 1m
                labels:
                  severity: critical
                annotations:
                  summary: "High Memory Cache"
                  description: "Memory Cache is more than 150000000"

В нем есть две логический части: настройка самого алертменеджера, в которой мы указываем куда мы будем слать алерты, и сама настройка алерта.

Далее нужно выполнить следующую команду для того, чтобы конфиг запустить:

    helm upgrade -f alertmanager.yml -n monitoring prometheus prometheus-community/prometheus

На этом шаге у меня все запустилось: 

![photo_2024-12-01_14-59-43](https://github.com/user-attachments/assets/39f8040d-dabd-4095-afc8-3bea49c47dbc)

Потом я проверила под, он упал( 

![photo_2024-12-01_14-59-41](https://github.com/user-attachments/assets/2a8872ad-dd79-4ac0-9089-a1f4b2c4c97c)

Ошибка ни о чем не говорит кроме как о том, что он падает. После просмотра логов я поискала ошибку, как оказалось у меня был неправильно указан чат айди, поэтому с ним надо внимательно!

Исправила ошибку, запустила заново 

![photo_2024-12-01_14-59-42](https://github.com/user-attachments/assets/f2fe1a93-eff6-4736-a191-f204a71aee89)

Все гуд.

Теперь запускаем Alertmanager и глядим на наш алерт. 

![photo_2024-12-01_14-59-50](https://github.com/user-attachments/assets/b7ea9626-9965-4882-9223-dcb01cbbcd6c)

Открываем телеграм и урааааа, бот прислал вот что:

![photo_2024-12-01_14-59-53](https://github.com/user-attachments/assets/6a9d31bb-c923-4534-b606-d38404977684)

До слез

## Вывод

Лаба получилась классная! Очень простая на понимание, но не совсем простая в реализации, есть некоторые тонкости. Не обошлось без ошибок конечно, но в целом все сделалось достаточно быстро, ставлю 10/10
