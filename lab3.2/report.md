# Лабораторная работа №3*
## Задание

1. Создать helm chart на основе обычной 3 лабы
2. Задеплоить его в кластер
3. Поменять что-то в сервисе, задеплоить новую версию при помощи апгрейда релиза
4. В отчете приложить скрины всего процесса, все использованные файлы, а также привести три причины, по которым использовать хелм удобнее чем классический деплой через кубернетес манифесты

## Ход выполнения работы
Как и в любой работе, первое, что нам нужно сделать - подготовить окружение. 

### Установка Helm
Скачиваем Helm по официальной инструкции (первая ссылка в гугле =) )

    curl -fsSL https://get.helm.sh/helm-v3.10.0-linux-amd64.tar.gz -o helm-v3.10.0-linux-amd64.tar.gz

Распаковываем то, что только что скачали: 

    tar -zxvf helm-v3.10.0-linux-amd64.tar.gz

Ищем бинарник и перемещаем его 

    mv linux-amd64/helm /usr/local/bin/helm

Запускаем helm! Тут у меня возникла проблемка с правами для одной директории, поэтому выполняем еще 

    chmod 600 /home/inksrgv/.kube/config

Проверяем успешность установки с помощью `helm version`. Должно вывести что-то типа

    version.BuildInfo{Version:"v3.10.0", GitCommit:"ce66412a723e4d89555dc67217607c6579ffcb21", GitTreeState:"clean", GoVersion:"go1.18.6"}

### Создание helm chart на основе 3 лабы

Для того, чтобы создать helm chart нужно выполнить инструкцию create: `helm create test-app-chart`

![photo_2024-11-28_11-47-28](https://github.com/user-attachments/assets/da30c1b2-b1cd-471b-bf48-e56eb2f66aa8)

После этого у нас появится новая папка нашего chart, переходим в нее. Здесь нас интересуют 3 файла, в которые мы будем вносить изменения (в остальные тоже можно, но я не рискнула). Важно следить за табуляцией, из-за неправильной табуляции тут может быть много ошибок.

/templates/deployment.yaml

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      annotations:
        helm.sh/revision: "{{ .Release.Revision }}"
      name: {{ .Release.Name }}
      labels:
        app.kubernetes.io/name: {{ .Release.Name }}
        app.kubernetes.io/instance: {{ .Release.Name }}
    spec:
      replicas: {{ .Values.replicaCount }}  
      selector:
        matchLabels:
          app.kubernetes.io/name: {{ .Release.Name }}
      template:
        metadata:
          labels:
            app.kubernetes.io/name: {{ .Release.Name }}
            app.kubernetes.io/instance: {{ .Release.Name }}  
        spec:
          containers:
            - name: {{ .Release.Name }}
              image: {{ .Values.image.repository }}:{{ .Values.image.tag }}  
              imagePullPolicy: Never 
              ports:
                - containerPort: {{ .Values.service.port }}

/templates/service.yaml

    apiVersion: v1
    kind: Service
    metadata:
      name: {{ .Release.Name }}
      labels:
        app.kubernetes.io/name: {{ .Release.Name }}
        app.kubernetes.io/instance: {{ .Release.Name }}
    spec:
      ports:
        - port: {{ .Values.service.port }}
          targetPort: 5000
      selector:
          app.kubernetes.io/name: {{ .Release.Name }}

values.yaml (изменения были только в этих полях)

    replicaCount: 1

    image:
      repository: test-app  
      tag: latest 
      pullPolicy: Never  

    service:
      type: ClusterIP
      port: 5000        

Вот тут важно, чтобы имя образа соответствовало собранному докер образу. Также важно указывать тип сервиса (хоть дока и говорит что в случае, если я не укажу тип сервиса ничего не произойдет, его все же следует указать, тк без него не соберется ничего))

После выполнения всех изменений переходим к следующему пункту.

### Деплой хелма в кластер

Для того, чтобы задеплоить helm, нужно выполнить `helm install test-app-release ./test-app-chart`

![photo_2024-11-28_12-12-43](https://github.com/user-attachments/assets/3e00a4ac-d07a-4dae-8474-b1c7cc545ce3)

Теперь проверим статусы пода и сервиса: 

![photo_2024-11-28_11-46-02](https://github.com/user-attachments/assets/3ad2ed1b-f53e-49f4-842e-063d0c08df60)

Не повезло, но мы не расстраиваемся. Из официальной доки: 

`The status ImagePullBackOff means that a container could not start because Kubernetes could not pull a container image (for reasons such as invalid image name, or pulling from a private registry without imagePullSecret).`

В моем случае ошибка была в имени контейнера (не то что бы я искала ее 40 минут ;) ). Исправили, перезапустили, смотрим результат:

Под: 

![photo_2024-11-28_11-46-25](https://github.com/user-attachments/assets/01801121-3802-4b5f-9926-380151e4863e)

Сервис: 

![photo_2024-11-28_11-46-28](https://github.com/user-attachments/assets/c17c7be3-fb4d-4264-b0b8-052d44fe486b)

Все круто! ура

Теперь попробуем перейти на нашу страничку. 

![photo_2024-11-28_11-46-31](https://github.com/user-attachments/assets/9da59b21-e34f-4376-a787-880a4e5d02ae)

АААААААААААа

Ладно, все нормально) Гугл вот что говорит: из-за типа сервиса ClusterIP мой сервис доступен долько внутри кластера, а чтобы сделать его доступным извне, необходимо пробросить порты

![photo_2024-11-28_11-46-33](https://github.com/user-attachments/assets/812b99aa-8e5c-491d-b2ea-d2cab83e1105)

Переходим по ссылке и ... ура! все сработало

![photo_2024-11-28_11-46-35](https://github.com/user-attachments/assets/43bb8a78-cf36-41fc-8cea-ac32afa2136a)

### Апгрейд релиза

Поменять можно много чего, я добавила еще несколько подов (изменила параметр replicaCount в файле values.yaml). Далее нужно выполнить апгрейд релиза с помощью команды `helm upgrade test-release ./test-chart`

Посмотрим теперь на поды:

![photo_2024-11-28_11-46-38](https://github.com/user-attachments/assets/8a6be882-391f-4f4a-ab1d-6c2ac9f3d60a)

Как можно заметить количество подов увеличилось, значит релиз успешен.

Также можно поменять тип сервиса, например на NodePort, чтоб порты каждый раз не пробрасывать

Теперь поглядим историю моего релиза: 

![photo_2024-11-28_11-46-39](https://github.com/user-attachments/assets/2ffa30ae-f217-4d98-8346-9422fc344577)

Полезно и приятно

## Три причины, по которым использовать хелм удобнее чем классический деплой через кубернетес манифесты

1. Все важные параметры хранятся в одном файле values. При запуске хелма значения, прописанные в values применяются одномременно ко всему проекту, благодаря чему ошибки ищутся быстрее =)
2. В случае чего можно быстро откатить релиз, без танцев с бубном. Управление релизом в целом здесь достаточно простое.
3. Еще можно написать шаблон и использовать его в разных проектах, прикольно.

## Вывод

В ходе выполнения лабораторной работы я познакомилась с таким инструментом как Helm. Пока что не могу понять своего отношения к этому инструменту. Попробую поднять что-нибудь посложнее с помощью хелма и сравню с кубером, тк пока мне кажется что кубер манифесты удобнее (или понятнее..) 🤔
