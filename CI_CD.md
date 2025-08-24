### CI/CD пайплайн зроблю простий, буде базуватись на GitHub Actions. secrets: буду зберігати в GitHub secrets (AWS keys via OIDC). 

## CI частина

Трігер: Коли пуш в feature/* або PR в staging/main
Кроки:
- Витягую код.
- Ставлю Node.js v20.
- npm install і запускаю тести
- Будую докер імейджі - docker build -t $ECR_REPO/frontend:$TAG . і так для кожного докерфайлу
- Пушу в ECR - логін через aws ecr get-login-password , потім docker push

- Для мікросервісів - Якщо зміни тільки в одному, будую тільки його, щоб не витрачати час та ресурси.

## CD частина

- Дев - Автоматично після успішного CI. kubectl apply -f k8s/dev/ --namespace dev. З роллінг апдейтом.
- Stage: Деплой після мерджу PR в staging. kubectl apply -f k8s/stage/ --namespace stage.
- Prod: Треба ручне підтвердження тім лідом або сенйор девом (в GitHub environments). kubectl apply -f k8s/prod/.
- Зв'язок з кубером - Використаю kubectl прямо в аctions (є вже готовий action setup-kubectl, можна взяти його)
- Rollback: Якщо виникла якась помилка, повернусь до попереднього тегу в ECR і зроблю деплой знову

В цілях економії часу буду також кешувати npm і докер слої.
Так система виходить достатньо гнучкою - в дев все буде швидко, в проді надійно
