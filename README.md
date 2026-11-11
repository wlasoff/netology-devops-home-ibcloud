# Домашнее задание к занятию  «Безопасность в облачных провайдерах» - Студент группы FOPS-42 Власов Владимир

Используя конфигурации, выполненные в рамках предыдущих домашних заданий, нужно добавить возможность шифрования бакета.

---
## Задание 1. Yandex Cloud   

1. С помощью ключа в KMS необходимо зашифровать содержимое бакета:

 - создать ключ в KMS;
 - с помощью ключа зашифровать содержимое бакета, созданного ранее.
2. (Выполняется не в Terraform)* Создать статический сайт в Object Storage c собственным публичным адресом и сделать доступным по HTTPS:

 - создать сертификат;
 - создать статическую страницу в Object Storage и применить сертификат HTTPS;
 - в качестве результата предоставить скриншот на страницу с сертификатом в заголовке (замочек).

Полезные документы:

- [Настройка HTTPS статичного сайта](https://cloud.yandex.ru/docs/storage/operations/hosting/certificate).
- [Object Storage bucket](https://registry.terraform.io/providers/yandex-cloud/yandex/latest/docs/resources/storage_bucket).
- [KMS key](https://registry.terraform.io/providers/yandex-cloud/yandex/latest/docs/resources/kms_symmetric_key).

--- 



## Выполнение задания 1. Yandex Cloud 

Используя конфигурации, выполненные в рамках [<предыдущего домашнего задания>](https://github.com/wlasoff/netology-devops-home-balance) 

1. Для того что бы создать бакет Object Storage и разместить в нём файл с картинкой, для этого:

- добавляем права доступа для сервисного аккаунта;
![sa](./img/yc_sa.jpg)

- создаем бакет в Object Storage с произвольным именем (например, _имя_студента_дата_);

![backet](./img/yc_backet1.jpg)

- выгрузим в бакет файл с картинкой;

![backet](./img/yc_backet2.jpg)
 
- сделаем картинку доступной из интернета;

![backet](./img/yc_backet3.jpg)


[<Тут конфигурация Bucket'a>](./cfg/bucket.tf)


- Права доступа на картинку;

![backet](./img/yc_backet4.jpg)

2. Создаем группу ВМ в public подсети фиксированного размера с шаблоном LAMP и веб-страницей, содержащей ссылку на картинку из бакета:

![vm](./img/vm_group.jpg)

 - Создаtv Instance Group с тремя ВМ и шаблоном LAMP, где для LAMP рекомендуется использовать `image_id = fd827b91d99psvq5fjit`.
 
![vm](./img/vm1.jpg)
 
 - Для создания стартовой веб-страницы рекомендуется использовать раздел `user_data` в [meta_data](https://cloud.yandex.ru/docs/compute/concepts/vm-metadata).
 
[Задаем параметры cloud-init ](./conf/instance-group.tf)

- При помощи cloud-init задаем насройку при первом запуске VM  где   <h1>LAMP Instance Group</h1>
      <img src="https://vlasov-v-hw15-2-2026.storage.yandexcloud.net/Vladimir_Vlasov.jpg" alt="Picture">

 - Размещаем в стартовой веб-странице шаблонной ВМ ссылку на картинку из бакета.
 
После запуска ВМ страница c картинкой доступна по http://51.250.90.54/

![p1](./img/pic1.jpg)

или по http://51.250.13.175/

![p2](./img/pic2.jpg)

или по http://51.250.84.142/

![p3](./img/pic3.jpg)

Запись производится в корень index.html
 ```
 write_files:
  - path: /var/www/html/index.html
    content: |
 ```

 - Настроим проверку состояния ВМ.

Прописывается в instance-group.tf
```

  health_check {
    interval            = 5
    timeout             = 4 #  таймаут должен быть меньше интервала (5 сек)
    healthy_threshold   = 3 #  здесь три проверки что все окей работает
    unhealthy_threshold = 2 #  проверка на неработу, недоступность по 80 порту
    tcp_options {
      port = 80
    }

*    - health_checks_spec.health_check_specs[0].unhealthy_threshold: Value must be one of [0, 2, 3, 4, 5, 6, 7, 8, 9, 10]


Логика:

healthy_threshold = 3 — нужно три успешные проверки подряд, чтобы ВМ признали здоровой (15 секунд: 3 × 5)

unhealthy_threshold = 2 — достаточно одной неудачной проверки, чтобы ВМ признали нездоровой (10 секунд)

Группа быстро реагирует на отказ и сразу начинает замену. 

Если ВМ перестаёт отвечать на порт 80 три проверки подряд, группа автоматически заменяет её новой.

```
![status](./img/status_vm.jpg)

3. Подключаем группу к сетевому балансировщику:

 - создаем сетевой балансировщик [Сетевой блансировщик](./conf/load-balancer.tf)

![Сетевой балансировщик](./img/lb1.jpg)

![Прослушиватель](./img/lb2.jpg)

![Целевые группы](./img/lb3.jpg)

 - проверяем работоспособность, удалив одну или несколько ВМ

![Удаляем](./img/del_vm1.jpg)

ВМ удаляется

![Удаляем](./img/del_vm2.jpg)

Пропал публичный IP у ВМ

![Удаляем](./img/del_vm3.jpg)

Нет доступа к сайту по http://51.250.90.54/

![Видим что сайт не работает](./img/no_site.jpg)

ВМ пересоздается с новым публичным IP 51.250.80.7:

![Машины пересоздаются](./img/recover_vm.jpg)

Появился доступ к сайту по новому адресу http://51.250.80.7/

![Видим что сайт работает](./img/yes_site.jpg)


[<Файлы для terraform>](./cfg/)

Удаляем все ресурсы на Yandex Cloud командой

```bash
 terraform destroy
   ```
![Удаляем](./img/del_ter.jpg)

