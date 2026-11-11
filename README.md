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

## Выполнение пункта 1 задания 1. Yandex Cloud 

Используя конфигурации, выполненные в рамках [предыдущего домашнего задания](https://github.com/wlasoff/netology-devops-home-balance) 

1. Для того что бы с помощью ключа в KMS зашифровать содержимое бакета:
   - первым делом создаю ключ в KMS прописывая параметры в отдельном файле [kms.tf ](./conf/kms.tf):
 ```
 resource "yandex_kms_symmetric_key" "bucket_encryption" {
  name               = "${var.bucket_name}-encryption-key"
  description        = "Ключ для шифрования бакета ${var.bucket_name}"
  default_algorithm  = "AES_256"
  rotation_period    = "8760h" # 1 год
  deletion_protection = false
 ```
 и ключ автоматически создается:
![key](./img/key.jpg)

 - для того чтобы все объекты в вашем бакете автоматически шифровались при загрузке новым ключом из KMS, добавим файле [backet.tf ](./conf/backet.tf):
```
 server_side_encryption_configuration { 
	rule{
	  apply_server_side_encryption_by_default {
            sse_algorithm = "aws:kms"
              kms_master_key_id = yandex_kms_symmetric_key.bucket_encryption.id
}
 ```

- проверим, что объект действительно зашифрован командой:
```
  $ curl -sI https://storage.yandexcloud.net/vlasov-v-hw15-2-2026/Vladimir_Vlasov.jpg | head -3
 ```
![backet1](./img/bucket1.jpg)

или на YC:
![backet2](./img/bucket2.jpg)

 видим то, что sse_kms_key_id объекта совпадает с идентификатором созданного ключа — файл зашифрован именно им.

Доступность объекта не изменилась и расшифровка происходит на стороне Object Storage прозрачно для клиента, поэтому публичная ссылка продолжает работать:

```
curl -sI https://storage.yandexcloud.net/vlasov-v-hw15-2-2026/Vladimir_Vlasov.jpg | head -3
 ```

![backet3](./img/bucket3.jpg)

## Выполнение пункта 2 задания 1. Yandex Cloud 

Будем использовать домен hw15.vlasov.tech, файлы страницы лежат [тут](./site/).

Создаю статический сайт в Object Storage c собственным публичным адресом и сделать доступным по HTTPS:

- для начала нужен бакет с именем, совпадающим с доменом.
```
yc storage bucket create --name hw15.vlasov.tech --public-read
 ```
![backet4](./img/bucket4.jpg)

- загрузим страницы:
```
yc storage s3 cp index.html s3://hw15.vlasov.tech/index.html
yc storage s3 cp error.html s3://hw15.vlasov.tech/error.html
```
![backet5](./img/bucket5.jpg)

- включf. в настройках бакета [остинг статического сайта : главная страница - index.html, страница ошибки - error.html.
  Проверим настройки:
  ![backet6](./img/bucket6.jpg)

  

[<Файлы для terraform>](./cfg/)

Удаляем все ресурсы на Yandex Cloud командой

```bash
 terraform destroy
   ```
![Удаляем](./img/del_ter.jpg)

