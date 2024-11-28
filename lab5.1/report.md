# Лабораторная работа № 5 

## Задание
Сделать мониторинг сервиса, поднятого в кубере (использовать, например, prometheus и grafana). Показать хотя бы два рабочих графика, которые будут отражать состояние системы. Приложить скриншоты всего процесса настройки.

## Ход выполнения работы

По классике первое, что нужно сделать - настроить окружение.

Установку буду делать с помощью helm по вот [такой](https://grafana.com/docs/grafana/latest/setup-grafana/installation/helm/) инструкции из официальной доки (для Prometheus аналогично).

Добавляем хелм репозиторий для Grafana:

    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

И для Prometheus:

    helm repo add grafana https://grafana.github.io/helm-charts

Обновляем репозиторий: 

    helm repo update

Теперь устанавливаем Prometheus + сразу создаем пространство имен:

    helm install prometheus prometheus-community/prometheus --namespace monitoring --create-namespace

![photo_2024-11-28_14-20-17](https://github.com/user-attachments/assets/1738fc33-5a28-4ac2-913b-c42cb11f747c)

Устанавливаем Grafana:

    helm install grafana grafana/grafana --namespace monitoring

![photo_2024-11-28_14-20-16](https://github.com/user-attachments/assets/a26d86fb-41de-46c5-b617-9ce5670e5c94)

Теперь получаем пароль от Grafana:

![photo_2024-11-28_14-20-15](https://github.com/user-attachments/assets/9909ba35-f5e8-4275-99b8-4cba679f68af)

Пробрасываем порт: 

![photo_2024-11-28_14-20-14](https://github.com/user-attachments/assets/1faeb19f-e292-4af5-b498-a146552f2c79)

И переходим по ссылочке. Тут мы уже получаем интерфейс grafana:

![photo_2024-11-28_14-20-12](https://github.com/user-attachments/assets/73525f23-afa6-432e-b148-f0357f3d57e2)

Теперь вводим тот самый пароль, который мы получали несколько шагов назад и ура, мы на главной странице графаны!

![photo_2024-11-28_14-20-11](https://github.com/user-attachments/assets/a97f7792-9df6-4b5e-b896-8c41a38fd910)

Подключаться к Prometheus будем по ссылке. чтобы достать рабочий сервис в нашем пространстве имен нужно выполнить команду `kubectl get svc -n monitoring` и оттуда забрать имя сервиса, в моем случае я буду использовать prometheus-server)

![photo_2024-11-28_14-20-09](https://github.com/user-attachments/assets/bc2d05e5-4536-4111-9d98-b0e6d60455a5)

Теперь в Grafana добавляем в качестве data source Prometheus (большое внимание уделяем правильности ссылки на сервер!!!):

![photo_2024-11-28_14-20-10](https://github.com/user-attachments/assets/b6d543ef-1598-422f-92db-f368573250f8)

Теперь можем строить графики. Делается это в разделе dasboards(неожиданность).

Для того, чтобы построить гарфик надо выбрать метрику. После выбора метрики нужно запустить запрос. Также графана предоставляет возможность делать кучу разных манипуляций с графиками, но я особо ничего не меняла. 

Первая метрика - количество http запросов.

![photo_2024-11-28_14-20-08](https://github.com/user-attachments/assets/dd8ce4b0-a573-423d-9021-15cac5286344)

Вторая - кол-во кэша

![photo_2024-11-28_14-20-06](https://github.com/user-attachments/assets/8983a3c9-94b3-4afb-a07a-3fc177991a97)

Потом я добавила еще 2 графика и сделала красивый дашборд, по которому можно видеть состояние моего сервиса

![photo_2024-11-28_14-20-03](https://github.com/user-attachments/assets/cf5728f1-b901-4b23-ac82-9b6ab695b483)


## Вывод

В ходе выполнения данной ЛР я познакомилась с такими инстурментами как Grafana и Prometheus. Мне понравилось с ними работать, тк интерфейс реально понятный и интересный. Попробую помониторить еще что-нибудь. Кстати забавный факт, это первая лаба где у меня НИЧЕГО НИ РАЗУ не упало и все получилось достаточно быстро, хотя с интсрументом впервые работаю. Забавно)
