<img width="441" height="322" alt="image" src="https://github.com/user-attachments/assets/9bfd1ba9-5dd3-4d5d-b8b8-7318382c3127" />

## Структура кластера та namespaces

У кластері EKS буду використовувати неймспейс для ізоляції. Для dev і stage- окремі неймспейси в спільному кластері ( kubectl create namespace dev і kubectl create namespace stage ) Для прода все в дефолт неймспейс , оскільки це окремий кластер

Додам ResourceQuota на кожен неймспейс. Обмежу CPU на 2 ядра і пам'ять на 4гб для dev , щоб тести не з"їли всі ресурси.

## Deployments та Pods

Для кожного сервісу створюю окремий деплоймент (*у файлі tradeoffs.md опишу декілька варіантів як можна булоб зробити це по іншому, тут юзаю стандартну ідею для мікросервісів*)

Фронтенд (Next.js): Деплоймент з 1-2 репліками на старт.
Бекенд (Rest api): Аналогічно, з 1-2 репліками, і з probes для перевірки хелсі.
Мікросервіси: Окремий Deployment для кожного , з однією реплікою спочатку , але готові до скейлу.
У всіх подах пропишу resources requests і limits (щось типу - cpu: 100m requests і 500m limits, memory: 256Mi requests і 1Gi limits), щоб Kubernetes міг правильно планувати і HPA працював без проблем.
Probes: додам livenessProbe і readinessProbe, HTTP GET на /health, щоб кубер знав коли под живий і готовий приймати трафік. Без цього можуть бути помилки з невідправленими запитами або з рестартами 

## Services та Networking

Щоб поди могли спілкуватися візьму Services. Для фронту і API -  ClusterIP бо зовнішній доступ через Ingress.
Для мікросервісів теж ClusterIP 
Нетворкінг - внутрішній трафік йде через DNS кубера. Не буде статичних айпі, все динамічно що робить систему гнучкою

## Configs та Secrets

Для конфігів (URL, порти) — ConfigMap, монтую як environment variables в подсах.

Для секретів (паролі, ключі) — Secret, створюю через kubectl і монтую аналогічно.

## Scaling та Reliability

Для мікросервісів візьмемо HPA на базі CPU з target 50% , minReplicas 1, maxReplicas 10. Це автоматично скейлить поди при навантаженні.
Для reliability — PodDisruptionBudget з minаvailable 1, щоб при оновленнях не всі поди вимкнулися одразу.
Встановлю Metrics Server у самому кластері , бо без нього HPA не зможе працювати.
