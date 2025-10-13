# Домашнее задание №32
> Цель: научиться автоматизировать процесс разработки и сборки программного обеспечения, использовать инструменты для разработки
pipeline CI/CD и интеграции с редактором кода VSC, а также оптимизировать процесс сборки с помощью параллельного исполнения.

## TASK 
Gitlab CI practice
**Написать gitlab-ci pipeline для приложения** https://github.com/AnastasiyaGapochkina01/dos-29-cicd_04
Требования:
Pipeliene должен состоять из этапов
code-quality - запуск линтеров
tests - запуск тестов
build - сборка и пуш docker image
deploy - запуск контейнера на удаленной машине
notify - уведомление о начале pipleine и о его завершении (в любой мессенджер)

Этап code-quality должен включать в себя запуск линтеров как для кода приложения так и для файлов docker
Этап tests должен состоять из двух jobs:
первая должна прогонять unit тесты, запускаться всегда и сохранять в артефактах файл с результатами
вторая должна запускаться после удачного деплоя и проверять что приложение доступно
Этап build должен собирать и пушить docker image для приложения (registry можно использовать любой, хоть container registry от самого gitlab, хоть docker hub, хоть свой собственный)
Этап deploy должен запускать новую версию приложения на удаленной машине (стратегию продумайте самостоятельно);
Этап notify должен запускаться дважды:
при старте пайплайна
по окончанию пайплайна (включая статус завершения)
опционально в уведомлении об окончании пайплайна можно добавить файл с результатами тестов

1. Склонируем проект на локальную машину и настроим там git для пуша в gitlab CI.
   <img width="1399" height="588" alt="image" src="https://github.com/user-attachments/assets/433c0899-23a7-41cc-952f-e482cc70eb51" />
2. Подготовим в Yandex Cloud машину куда будем деплоить и 2 раннера с тегами "docker" и "shell" для выполняние разных джобов и привяжем их к gitlab через url и token.
<img width="1610" height="434" alt="image" src="https://github.com/user-attachments/assets/497c4652-e893-46df-bb57-29003a9e815b" />
<img width="1134" height="366" alt="image" src="https://github.com/user-attachments/assets/a7691908-1cbf-4ff4-b994-444c5e50f376" />
3. Подготовим бота в telegram через BotFather c именем NotifyBot для уведомлений о старте пайплайна и о его завершении. Token и chat-id разместим в переменные окружения.
4. Подготовим переменное окружение в GitLab 
<img width="1337" height="519" alt="image" src="https://github.com/user-attachments/assets/f672871b-c012-4cbd-8dbf-26bab5e3f6ef" />
5. Выведем текст самого конфигурационного файла CI/CD .gitlab-ci.yml (не получилось сделать test и smoke в одном stage :(   )

   ```
   ---
# Этапы пайплайна
stages:
  - code-quality
  - build
  - test
  - deploy
  - smoke
  - notify

variables:
  IMAGE: "backend-diary"
  TAG: "v01"
  APP_NAME: "dos-29-backend-diary"
  SSH_HOST: "158.160.172.160"
  SSH_USER: "user"
  REGISTRY: "${CI_REGISTRY_IMAGE}"
  TELEGRAM_MESSAGE_START: " 🚀 Пайплайн с номером ($CI_COMMIT_SHORT_SHA) запущен ${CI_PIPELINE_CREATED_AT}. "
  TELEGRAM_MESSAGE_END: "✅ Пайплайн ($CI_COMMIT_SHORT_SHA) завершился . Status: $CI_JOB_STATUS. Commit message: $CI_COMMIT_MESSAGE"

notify-start:
  stage: notify
  image: curlimages/curl:latest
  script:
    - echo "Отправляем уведомление о начале пайплайна..."
    - curl -s -X POST "https://api.telegram.org/bot$TELEGRAM_TOKEN/sendMessage" -d chat_id=$TELEGRAM_CHAT_ID -d text="$TELEGRAM_MESSAGE_START"
  rules:
    - when: always
  needs: []
  allow_failure: true
  tags:
    - shell

code-quality:
  stage: code-quality
  image: node:18
  script:
    - echo "Запускаем линтеры...."
    - npm install eslint --save-dev
    - npx eslint server.js __tests__
  tags:
    - docker

build-job:
  stage: build
  script:
    - echo "🔧 Логинимся в GitLab Container Registry..."
    - echo "${CI_REGISTRY_PASSWORD}" | docker login -u "${CI_REGISTRY_USER}" "${CI_REGISTRY}" --password-stdin
    - echo "🐳 Собираем Docker image..."
    - docker build --cache-from ${REGISTRY}:${TAG} -t ${REGISTRY}:${TAG} .
    - echo "📦 Пушим image в registry..."
    - docker push ${REGISTRY}:${TAG}
    - echo "🧹 Очищаем учетные данные Docker..."
    - docker logout ${CI_REGISTRY}
  tags:
    - shell

test-jsonapp:
  stage: test
  script:
    - echo "🔧 Логинимся в GitLab Container Registry..."
    - echo "${CI_REGISTRY_PASSWORD}" | docker login -u "${CI_REGISTRY_USER}" "${CI_REGISTRY}" --password-stdin
    - echo "📥 Загружаем image из registry..."
    - docker pull ${REGISTRY}:${TAG}
    - echo "🧹 Проверяем, нет ли старого контейнера..."
    - docker rm -f ${APP_NAME}-test 2>/dev/null || true
    - echo "🛠 Запускаем тест..."
    - docker run -d --name ${APP_NAME}-test ${REGISTRY}:${TAG} sh -c "echo '=== NPM INFO ===' && npm --version && echo '=== NPM TEST START ===' && npm test || { echo 'NPM TEST FAILED'; exit 1; } && echo '=== NPM TEST SUCCESS ===' && ls -la /app" || { echo "Ошибка выполнения тестов"; exit 1; }
    - echo "⏳ Ожидаем завершения тестов..."
    - timeout 30 docker wait ${APP_NAME}-test
    - mkdir -p reports
    - echo "📥 Копируем результат теста..."
    - docker cp ${APP_NAME}-test:/app/tests-result.json reports/tests-result.json 2>/dev/null || echo "Файл tests-result.json не найден"
    - docker rm -f ${APP_NAME}-test
  artifacts:
    paths:
      - reports/tests-result.json
    when: always
    expire_in: 1 week
  needs: [build-job]
  tags:
    - shell

deploy-backend:
  stage: deploy
  before_script:
    - echo "🔐 Настраиваем SSH-доступ..."
    - mkdir -p ~/.ssh
    - echo "$SSH_PRIVATE_KEY" > ~/.ssh/yandex_key
    - chmod 600 ~/.ssh/yandex_key
    - ssh-keyscan -H ${SSH_HOST} >> ~/.ssh/known_hosts
  script:
    - echo "🚀 Деплой на ${SSH_HOST}..."
    - ssh -i ~/.ssh/yandex_key ${SSH_USER}@${SSH_HOST} "
        echo '${CI_REGISTRY_PASSWORD}' | docker login -u '${CI_REGISTRY_USER}' ${CI_REGISTRY} --password-stdin &&
        docker pull ${REGISTRY}:${TAG} &&
        docker rm -f ${APP_NAME} || true &&
        docker run -d --name ${APP_NAME} -p 3000:3000 ${REGISTRY}:${TAG}
      "
  needs: [test-jsonapp]
  tags:
    - docker

smoke-test:
  stage: smoke
  image: curlimages/curl:latest
  script:
    - echo "🛠 Проверяем доступность приложения после деплоя..."
    - curl -f -I http://${SSH_HOST}:3000 || echo "Smoke test failed"
  needs: [deploy-backend]
  tags:
    - docker   

notify-end:
  stage: notify
  image: curlimages/curl:latest
  script:
    - |
      set -x 
      echo "Отправляем уведомление о завершении пайплайна..."
      echo "Начальное значение STATUS: $STATUS"
      echo "Статус пайплайна: $CI_JOB_STATUS"
      if [ "$CI_JOB_STATUS" = "running" ]; then
        STATUS="Успешно"
        echo "Обновлённое значение STATUS: $STATUS"
        echo "Статус пайплайна: $CI_JOB_STATUS"
        MESSAGE="✅Пайплайн завершён успешно! Все джобы выполнены."
      else
        STATUS="Неудачно"
        MESSAGE="❌Пайплайн завершён с ошибкой! Проверьте логи джоб."
      fi
      
      echo "Отправляем сообщение в Telegram..."
      curl -s -X POST "https://api.telegram.org/bot$TELEGRAM_TOKEN/sendMessage" \
        -d chat_id="$TELEGRAM_CHAT_ID" \
        -d text="Статус пайплайна= $STATUS%0A$MESSAGE%0AДетали= https://gitlab.com/skurat.aleksey/hw_lesson32/-/pipelines/$CI_PIPELINE_ID"

      if [ -f reports/tests-result.json ]; then
        echo "Отправляем файл результатов теста..."
        curl -s -X POST "https://api.telegram.org/bot$TELEGRAM_TOKEN/sendDocument" \
          -F chat_id="$TELEGRAM_CHAT_ID" \
          -F document=@reports/tests-result.json \
          -F caption="Результаты теста из джобы test-jsonapp (пайплайн #$CI_PIPELINE_ID)"
      else
        echo "Файл tests-result.json не найден, отправка документа скипаем."
      fi
  allow_failure: true
  tags:
    - shell 
```
> Как мы видим джоб notify-start входит в stage notify, который идет самым последним, но за счет needs: [] запускаем его самым первым.
На этапе build собираем образ и закидываем его в gitlab-registry
<img width="1180" height="266" alt="image" src="https://github.com/user-attachments/assets/67adf50d-fad5-4d5d-bef7-5e904eede93f" />
На этапе теста мы вновь подгружаем наш собранный образ и проводим тестирование а результат теста ложим в виде json файла в артефакты.
После этого проводим деплой на виртуальынй сервер в ya облаке и проверям его доступность с помощью smoke получаем
<img width="791" height="181" alt="image" src="https://github.com/user-attachments/assets/bd7f632b-abdd-4302-8481-5d71bf58870e" />


