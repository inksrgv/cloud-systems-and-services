# Лабораторная работа №3

## Задание
Поднять kubernetes кластер локально (например minikube), в нём развернуть свой сервис, используя 2-3 ресурса kubernetes. В идеале разворачивать кодом из yaml файлов одной командой запуска. Показать работоспособность сервиса.
(сервис любой из своих не опенсорсных, вывод “hello world” в браузер тоже подойдёт)

## Ход выполнения работы

### Настройка окружения

Миникуб у меня был установлен, но вообще установить можно по [инструкции](https://minikube.sigs.k8s.io/docs/start/?arch=%2Fwindows%2Fx86-64%2Fstable%2F.exe+download).

Чтобы проверить, что миникуб установлен и работает, запустим локальный кластер:

![Снимок экрана 2024-11-07 185159](https://github.com/user-attachments/assets/b20578fa-09a7-4acc-b6dd-3ee46a9b4214)

Кластер запустился успешно. Проверим его статус:


![Пример изображения](pics/docker-images.png)

Тоже все ок, можем идти дальше.

<details>
<summary> Устанавливаем lens </summary>
  
Идем на [официальный сайт](https://k8slens.dev/) и смотрим инструкцию по установке.


Получаем ключик: 
      
      curl -fsSL https://downloads.k8slens.dev/keys/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/lens-archive-keyring.gpg > /dev/null

Указываем stable канал:

    echo "deb [arch=amd64 signed-by=/usr/share/keyrings/lens-archive-keyring.gpg] https://downloads.k8slens.dev/apt/debian stable main" | sudo tee /etc/apt/sources.list.d/lens.list > /dev/null

Устанавливаем lens:

    sudo apt install lens

Дальше регистрируемся и ура! приложение нам доступно:

![Снимок экрана 2024-11-07 185749](https://github.com/user-attachments/assets/825375b5-68ae-4ed0-b217-9979f347150a)

</details>

### Разворачиваем сервис!

Для примера будем разворачивать простенькое приложение, которое открывает html-файл. Я такое уже писала для второй лабы, поэтому за основу возьмем его же. 

Первое, что нам нужно сделать - пересобрать наш Docker-образ:

![docker-build](https://github.com/user-attachments/assets/a5a0e3ad-db3d-4e55-9c25-ed4c0e3f51c9)

Посмотрим теперь что наш образ виден:

![docker-images](https://github.com/user-attachments/assets/65abc35d-f554-499f-ba65-56b99ff5fb44)

Загружаем этот образ в minikube:

    minikube image load test-app

И идем писать манифесты! Для этой лабы их будет два: deployment и service.

Файл `deployment.yaml`


    apiVersion: apps/v1
    kind: Deployment
    metadata:
    name: test-app
    spec:
    replicas: 1
    selector:
    matchLabels:
      app: test-app
    template:
    metadata:
      labels:
        app: test-app
    spec:
      containers:
      - name: test-app
        image: test-app
        ports:
        - containerPort: 5000

Файл `service.yaml`

    apiVersion: v1
    kind: Service
    metadata:
    name: test-app-service
    spec:
    selector:
    app: test-app
    ports:
    - protocol: TCP
      port: 80
      targetPort: 5000
    type: NodePort

Запускаем!(как и просили, одной командой)

![kubectl-apply](https://github.com/user-attachments/assets/5138c2d4-ea4c-4cb5-ad94-83fe44663ba3)

Глядим что получилось:

![Снимок экрана 2024-11-07 192457](https://github.com/user-attachments/assets/90f91fcd-75f1-4225-a60f-c631058753eb)

Видим ошибку ImagePullBackOff. Увидела в комментариях к лабам, что не только я столкнулась с этой проблемой, поэтому подглядела решение у ребят :) Спасибо Вам коллеги🤝

Итак, в файл `service.yaml` была добавлена строчка 

    imagePullPolicy: Never

Перезапускаем наш сервис иииии..... ловим новую ошибку

![Снимок экрана 2024-11-07 191737](https://github.com/user-attachments/assets/e607c440-b5ce-4f3f-b689-a157a500e467)

Ошибка говорит мне лишь о том, что под валит тесты и не работает, поэтому бежим смотреть логи:

![Снимок экрана 2024-11-07 191936](https://github.com/user-attachments/assets/102edcbe-a518-4c30-a9b1-27761c0ea2e3)

Мда, не этого я ожиидала. Меняем рабочую директорию в Dockerfile, потому что я забыла ее поменять)

Пересобираем образ, перезапускаем сервис, и получаем новую ошибку!

![Снимок экрана 2024-11-07 192012](https://github.com/user-attachments/assets/b096b036-ff9a-488f-964a-039597b030e9)

Я забыла загрузить образ :). 

Загружаем образ с помощью команды

    minikube image load test-app

Запускаем сервис и ... это победа!

![Снимок экрана 2024-11-07 192146](https://github.com/user-attachments/assets/47f60f14-9b5b-4f8b-8fc0-3ae972a54626)

Посмотрим как это выглядит в lens: 

![Снимок экрана 2024-11-07 192837](https://github.com/user-attachments/assets/04ea9dc9-3c4d-46e9-b135-4b1892b4cc05)

Эстетично

Теперь чекнем работающие сервисы:

![Снимок экрана 2024-11-07 192828](https://github.com/user-attachments/assets/38c0f62c-5f94-46a9-b8be-ffa8606e4c0e)

Перейдем по ссылке и..

![Снимок экрана 2024-11-07 192822](https://github.com/user-attachments/assets/8a391934-4bb2-4cff-9d98-13aa9963c2c3)

Ура! Работает)

Поглядим на мой кластер:

![Снимок экрана 2024-11-07 192858](https://github.com/user-attachments/assets/3b351d5c-149a-4a03-92ed-63349c069a44)

Ну красота!

### Вывод

В ходе выполнения лабораторной работы я подняла сервис в minikube. Это не первый мой опыт работы с minikube, однако сложности, как можно заметить, возникали) В целом, работой я довольна, узнала о новом инструменте и вспомнила как работать с minikube.

