# Лабораторная работа №2

## Задание
1. Написать “плохой” Docker compose файл, в котором есть не менее трех “bad practices” по их написанию.
2. Написать “хороший” Docker compose файл, в котором эти плохие практики исправлены.
3. В Readme описать каждую из плохих практик в плохом файле, почему она плохая и как в хорошем она была исправлена, как исправление повлияло на результат.
4. После предыдущих пунктов в хорошем файле настроить сервисы так, чтобы контейнеры в рамках этого compose-проекта так же поднимались вместе, но не "видели" друг друга по сети. В отчете описать, как этого добились и кратко объяснить принцип такой изоляции.

## Ход выполнения работы 

### "Плохой" Docker compose файл

Плохой Docker compose будет выглядеть так:
    
    version: '3.8'

    services:
      web:
        build: .
        ports:
          - "5000:5000"
        environment:
          - FLASK_APP=/app/main.py
          - FLASK_RUN_HOST=0.0.0.0
          - DATABASE_URL=postgresql://postgres:password@db/postgres

      db:
        image: postgres:latest
        environment:
          POSTGRES_USER: postgres
          POSTGRES_PASSWORD: password
          POSTGRES_DB: postgres
        volumes:
          - pgdata:/var/lib/postgresql/data

      pgadmin:
        image: dpage/pgadmin4
        environment:
          PGADMIN_DEFAULT_EMAIL: vragozina393@gmail.com
          PGADMIN_DEFAULT_PASSWORD: admin
        ports:
          - "8080:80"

        volumes:
          pgdata:

Перейдем к описанию "плохих" практик:
1. Явно не указывать версию образа. Ну тут все аналогично обычному Dockerfile: указание тега latest может привести к непредсказуемому поведению сборки и работы контейнера в целом. Если в обычном докерфайле это не супер страшно (хотя нежелательно), то в docker compose файле может вылезти несовместимость версий (например БД и веба).
2. Явное указание переменных окружения. Такие данные легко потерять да и вообще это небезопасно, опять же аналогично обычному Dockerfile.
3. Если продолжать говорить о безопасности данных, то плохой практикой является отсутствие системы секретов. Вообще очень крутая практика, которая обеспечивает изоляцию данных.
4. Отсутствие зависимостей. Вот эта практика относится конкретно к docker compose. Почему это плохо: Представим, что у нас есть какой-то сервис и какая-то база данных. В структуре .yml файла они идут друг за другом и зависят друг от друга, а зависимости нигде не указаны. Веб сервис запустится, однако он не сможет работать без БД, но и подключиться он к ней тоже не может, она ведь еще не запущена… кринж 🙁
4.1. Тут я узнала, что есть “альтернативы” depends_on, такие как sleep, использование скриптов для создания “задержки” и тд. Это конечно может сработать, но правильнее будет использовать depends_on
5. Нет изоляции сервисов друг от друга. Это кстати к 4 пункту задания. Это плохая практика, тк это может повлиять на безопасность (например, кто-то получил доступ к нашему веб-сервису, при этом наши сервисы не изолированы. С веба злоумышленник легко пройдет к бд и заберет оттуда все данные, неприятненько получится). Также при возникновении каких-то ошибок работа одного неизолированного сервиса может повлиять на работу другого.🙁

### "Хороший" Docker compose файл

Будет выглядеть так:

    version: '3.8'

    services:
      db:
        image: postgres:13.3
        environment:
          POSTGRES_USER: postgres
          POSTGRES_PASSWORD: /run/secrets/postgres_password
          POSTGRES_DB: postgres
        volumes:
          - pgdata:/var/lib/postgresql/data
        networks:
          - db-network
          - app-network
        secrets:
          - postgres_password

      web:
        build: .
        ports:
          - "5000:5000"
        env_file:
          - .env
        networks:
          - app-network
        depends_on:
          - db

      pgadmin:
        image: dpage/pgadmin4
        environment:
          PGADMIN_DEFAULT_EMAIL: vragozina393@gmail.com
          PGADMIN_DEFAULT_PASSWORD: /run/secrets/pgadmin_password
        ports:
          - "8080:80"
        depends_on:
          - db
        networks:
          - pg-network
        secrets:
          - pgadmin_password

    networks:
      app-network:
      db-network:
      pg-network:

    secrets:
      postgres_password:
        file: postgres_password.txt
      pgadmin_password:
        file: pgadmin_password.txt

    volumes:
      pgdata:

Что изменилось:
1. Добавила версию образа для postgres. Теперь поведение более предсказуемое. Кстати, конкретно в этом примере версия сильно роляет, у меня были проблемы с совместимостью из-за latest.
2. Переменные окрыжения спрятаны в файл .env
3. Добавлены секреты для важных данных (у меня это пароли)
4. Добавлены зависимости сервисов друг от друга: в моем случае и web и pgadmin зависимы от базы данных.
5. Сервисы изолированы с помощью networks. Каждый сервис работает в собственной сети.

Запустим сборку с помощью команды `docker-compose up --build`

![Снимок экрана 2024-11-07 200704](https://github.com/user-attachments/assets/8e82237d-2a6c-4c4f-9341-210f92fb3446)

Ну и посмотрим как оно работает. Первое окно - окно регистрации:

![Снимок экрана 2024-11-07 200734](https://github.com/user-attachments/assets/053ace03-b88b-4ada-b128-e84a20b84ac7)

После нажатия на кнопку "Зарегистрироваться" данные отправляются в БД.

![Снимок экрана 2024-11-07 200742](https://github.com/user-attachments/assets/b92f63a8-e9bf-4da9-bfee-204b337e5536)

Проверим успешность записи:

![Снимок экрана 2024-11-07 200748](https://github.com/user-attachments/assets/99b7404e-286f-45d0-b692-14d144799b9f)

Итого мы имеем простую форму, данные с которой записываются и хранятся в БД.

