# Бекап система
Зроблю просту бекап систему яка базується на AWS. Фокус на ключових речах: конфіги K8s, images в ECR, логи. Використаю aws бекап для central management. Щоденні бекапи для прода, та раз на тиждень для dev та stage. Retention - 7 днів для dev та stage і 30 днів для прода — автоматичне видалення старих, щоб не платити зайві кошти за зберігання. Відновлення буде відбуватись через aws консоль, швидко і просто

## Бекапи K8s кластера

- Control plane - EKS автоматично бекапить etcd
- Конфіги  -зберігатиму все в git (deployments, services тощо) 


## ECR images

Бекапи: 
- Lifecycle policy вже є (зберігаємо 30 останніх) , додатково replicate в інший region для DR (просто enable в ECR console)
- Retention - Авто - як в AWS розділі

## Логи 

- Cloud Watch логи - експорт в S3 через subscription filter
- Retention: 7 днів в CW , 30 днів в S3 для прода
