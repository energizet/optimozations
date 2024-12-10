# README

## Пайплайн с использованием HashiCorp Vault

```yaml
stages:
   - test
   - build
   - deploy

variables:
   DOCKER_DRIVER: overlay2
   VAULT_ADDR: "http://vault.energizet.ru:8200"
   VAULT_TOKEN: "$CI_VAULT_TOKEN"
   VAULT_SECRET_PATH: "secret/data/deploy/password"
   IMAGE_NAME: "energizet/history"
   IMAGE_TAG: "latest"

test:
   stage: test
   image: mcr.microsoft.com/dotnet/sdk:8.0
   only:
      - merge_requests
      - master
   script:
      - dotnet test Histoty.Storublioner.Tests/Histoty.Storublioner.Tests.csproj --logger:trx
      - exit $CI_PIPELINE_STATUS

build:
   stage: build
   image: docker:latest
   services:
      - docker:dind
   only:
      - merge_requests
   script:
      - docker build -t $IMAGE_NAME:$IMAGE_TAG .

build_master:
   stage: build
   extends: build
   only:
      - master
   after_script:
      - docker save $IMAGE_NAME:$IMAGE_TAG -o history_image.tar
   artifacts:
      paths:
         - history_image.tar
      expire_in: 1 hour

deploy:
   stage: deploy
   image: docker:latest
   services:
      - docker:dind
   only:
      - master
   when: manual
   before_script:
      - apk update && apk add curl jq
      - >
         export DEPLOY_PASSWORD=$(curl -s --header "X-Vault-Token: $VAULT_TOKEN" \
           $VAULT_ADDR/v1/$VAULT_SECRET_PATH | jq -r '.data.password')
      - echo $DEPLOY_PASSWORD | docker login -u energizet --password-stdin
      - docker load -i history_image.tar
   script:
      - docker push $IMAGE_NAME:$IMAGE_TAG
```

## Почему хранение секретов в CI/CD переменных репозитория — плохая практика?

1. **Уязвимость к утечкам**: 
   CI/CD переменные репозитория (например, в GitLab) могут быть доступны на разных этапах пайплайна, особенно если доступ к репозиторию имеют различные пользователи и сервисы. Это увеличивает шанс утечки данных, особенно если кто-то из разработчиков случайно использует их в логах или коммитах.

2. **Нет централизованного управления доступом**: 
   Когда секреты хранятся непосредственно в репозитории или CI/CD, управление доступом становится трудным. Это особенно важно, если в проекте работают несколько команд или сервисов, которым требуется различный уровень доступа к секретам.

3. **Отсутствие аудита и журналирования**:
   Хранение секретов в репозитории не предоставляет возможности для централизованного мониторинга и аудита. Вы не можете отслеживать, кто и когда получил доступ к секрету, что снижает уровень безопасности.

4. **Проблемы с изменением секретов**:
   При изменении секретов вам необходимо вручную обновлять их в каждом репозитории или конфигурации, что увеличивает вероятность ошибок и утечек.

---

## Преимущества использования HashiCorp Vault:

1. **Централизованное управление секретами**:
   Секреты хранятся в одном месте (HashiCorp Vault), и доступ к ним осуществляется через безопасные каналы с использованием API.

2. **Политики и контроль доступа**:
   С помощью Vault можно настроить строгие политики доступа, определяя, кто и как может получить доступ к конкретным секретам. Это значительно повышает безопасность.

3. **Шифрование данных**:
   Все данные в HashiCorp Vault шифруются, а доступ к ним осуществляется через зашифрованные каналы, что защищает секреты даже при компрометации сети.

4. **Временные и динамические секреты**:
   Vault поддерживает генерацию временных или динамических секретов, которые имеют ограниченный срок действия, что добавляет дополнительный уровень безопасности.

5. **Аудит и мониторинг**:
   Vault предоставляет встроенные возможности для аудита всех запросов и операций, что позволяет отслеживать доступ к секретам.
