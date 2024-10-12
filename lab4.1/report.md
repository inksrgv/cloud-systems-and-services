# CI/CD Default

<details>
<summary> Техническое задание </summary>

Написать “плохой” CI/CD файл, который работает, но в нем есть не менее пяти “bad practices” по написанию CI/CD
Написать “хороший” CI/CD, в котором эти плохие практики исправлены
В Readme описать каждую из плохих практик в плохом файле, почему она плохая и как в хорошем она была исправлена, как исправление повлияло на результат
</details>

## Это база

Была написана роль для раскатки grafana + плейбук. 


```yaml
---

stages:
  - setup
  - lint
  - test

variables:
  GRAFANA_PORT: 3000
  GRAFANA_VERSION: 9.0.0

setup_grafana:
  stage: setup
  image: ubuntu:latest
  services:
    - name: grafana/grafana:$GRAFANA_VERSION
      alias: grafana
  script:
    - echo "Installing curl..."
    - apt-get update && apt-get install -y curl 
    - echo "Grafana service is being set up"
    - sleep 10  
    - curl -I http://grafana:$GRAFANA_PORT || { echo "Grafana did not respond"; exit 1; }
  only:
    - main

lint:
  stage: lint
  image: python:3.9
  script:
    - echo "Running linters..."
    - pip install flake8
    - flake8 .
  only:
    - main

check_grafana:
  stage: test
  image: curlimages/curl:latest
  script:
    - echo "Checking if Grafana is running on localhost:$GRAFANA_PORT..."
    - sleep 10
    - curl -I http://grafana:$GRAFANA_PORT || { echo "Grafana is not responding!"; exit 1; }
  services:
    - name: grafana/grafana:$GRAFANA_VERSION
      alias: grafana
  only:
    - main
```

хорошего тут мало конечно...
![image](https://github.com/user-attachments/assets/d88c6d0e-bc00-40cf-9900-dab0e41dae14)

## чем дальше в лес

#### Использование нестабильного образа 
Используется `ubuntu:latest`, что не очень хорошо, так как `latest`  может привести к неожидаемому поведению, при обновлении базового образа.  
Лучше завязываться на конкретные стабильные версии, например на `ubuntu:20.04`. Это гарантирует, что изменения в образе не приведут к непредсказуемым ошибкам.

#### Оожидание сервиса через "sleep"
Использование фиксированного времени ожидания с `sleep` — неэффективно, так как сервис может подняться быстрее или медленнее указанного времени и мы либо теряем наше время, либо валимся на стабильной сборке :(
Чтобы было супер-гуд используется цикл с попытками подключения через `for i in {1..10}; do curl -I && break || sleep 3; done`, что делает процесс ожидания более гибким. 

#### Тяжелые образы для линтинга
Линтинг штука полезная, но не заморчная. Использование полного образа `python:3.9` увеличивает время сборки, а мы это не уважаем.
Нужно использовать легковесный образ `python:3.9-alpine` и быть как Молния МакКвин.

#### Линтеры запускаются на каждой ветке
Повторимся, линтеры штука полезная, но прогонять их на каждом коммите - не наш вариант, это избыточно и прошлый век.
Меням на запуск при `merge requests`и уменьшаем количество ненужных проверок.


```yaml
stages:
  - setup
  - lint
  - test

variables:
  GRAFANA_PORT: 3000
  GRAFANA_VERSION: 9.0.0

setup_grafana:
  stage: setup
  image: ubuntu:latest
  services:
    - name: grafana/grafana:$GRAFANA_VERSION
      alias: grafana
  script:
    - echo "Installing curl..."
    - apt-get update && apt-get install -y curl 
    - echo "Grafana service is being set up"
    - sleep 10  
    - curl -I http://grafana:$GRAFANA_PORT || { echo "Grafana did not respond"; exit 1; }
  only:
    - main

lint:
  stage: lint
  image: python:3.9
  script:
    - echo "Running linters..."
    - pip install flake8
    - flake8 .
  only:
    - main

check_grafana:
  stage: test
  image: curlimages/curl:latest
  script:
    - echo "Checking if Grafana is running on localhost:$GRAFANA_PORT..."
    - sleep 10
    - curl -I http://grafana:$GRAFANA_PORT || { echo "Grafana is not responding!"; exit 1; }
  services:
    - name: grafana/grafana:$GRAFANA_VERSION
      alias: grafana
  only:
    - main
```
![image](https://github.com/user-attachments/assets/79ca12e0-1724-45d4-8c6d-ba8f2899cf2f)


## что-то ещё

```yaml
stages:
  - setup
  - lint
  - test

variables:
  GRAFANA_PORT: 3000
  GRAFANA_VERSION: 9.0.0

setup_grafana:
  stage: setup
  image: ubuntu:20.04
  services:
    - name: grafana/grafana:$GRAFANA_VERSION
      alias: grafana
  script:
    - apt-get update && apt-get install -y curl
    - echo "Waiting for Grafana to start..."
    - for i in {1..10}; do curl -I http://grafana:$GRAFANA_PORT && break || sleep 3; done
  only:
    - main

lint:
  stage: lint
  image: python:3.9-alpine
  script:
    - pip install flake8
    - flake8 .
  only:
    - merge_requests 

check_grafana:
  stage: test
  image: curlimages/curl:latest
  script:
    - echo "Checking if Grafana is running on localhost:$GRAFANA_PORT..."
    - for i in {1..10}; do curl -I http://grafana:$GRAFANA_PORT && break || sleep 3; done
  services:
    - name: grafana/grafana:$GRAFANA_VERSION
      alias: grafana
  only:
    - main
```
![image](https://github.com/user-attachments/assets/5f066bad-9cf7-4528-8cff-8eb00abcbb3c)

## конфетка

```yaml
stages:
  - setup
  - lint
  - test

cache:
  paths:
    - .cache/pip

variables:
  GRAFANA_PORT: 3000
  GRAFANA_VERSION: 9.0.0
  TIMEOUT: 30 

setup_grafana:
  stage: setup
  image: ubuntu:20.04
  services:
    - name: grafana/grafana:$GRAFANA_VERSION
      alias: grafana
  script:
    - apt-get update && apt-get install -y curl
    - echo "Waiting for Grafana to start..."
    - for i in $(seq 1 $TIMEOUT); do curl -I http://grafana:$GRAFANA_PORT && break || sleep 1; done || (echo "Grafana not started in $TIMEOUT seconds" && exit 1)
  only:
    - main

lint:
  stage: lint
  image: python:3.9-alpine
  script:
    - pip install --cache-dir .cache/pip flake8
    - flake8 .
  only:
    - merge_requests
  artifacts:
    paths:
      - logs/lint/
    expire_in: 1 week  

check_grafana:
  stage: test
  image: curlimages/curl:latest
  services:
    - name: grafana/grafana:$GRAFANA_VERSION
      alias: grafana
  script:
    - echo "Checking if Grafana is running on localhost:$GRAFANA_PORT..."
    - for i in $(seq 1 $TIMEOUT); do curl -I http://grafana:$GRAFANA_PORT && break || sleep 1; done || (echo "Grafana not responding in $TIMEOUT seconds" && exit 1)
  only:
    - main
  artifacts:
    paths:
      - logs/test/
    expire_in: 1 week

lint_python_and_shell:
  stage: lint
  image: python:3.9-alpine
  parallel: 2 
  script:
    - echo "Running multiple lint jobs in parallel"
    - pip install --cache-dir .cache/pip flake8
    - flake8 .
  only:
    - merge_requests
  artifacts:
    paths:
      - logs/lint_parallel/
    expire_in: 1 week

```
![image](https://github.com/user-attachments/assets/ea9eb262-d1d8-4d35-8d6c-9cb5eed43288)
