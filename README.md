# 

# Домашнее задание к занятию «Введение в Terraform» - `Фомичев Анатолий`


## Ссылка на домашнее задание - https://github.com/netology-code/ter-homeworks/blob/main/01/hw-01.md


### Задача 1

  

Чеклист:

В данном файле сохранятся секреты в *.tfstate и *.tfstate.*(например .backup), !.terraformrc - восклицательный знак отменяет исключения .gitignore
  
![alt text](https://github.com/SLzDevOps/netology-terraform-1/blob/main/Screenshot_747.png).
  
      "mode": "managed",
      "type": "random_password",
      "name": "random_string",
      "provider": "provider[\"registry.terraform.io/hashicorp/random\"]",
      "instances": [
        {
          "schema_version": 3,
          "attributes": {
            "bcrypt_hash": "$2a$10$Dywg.TwDphWO1/Hux4Ecguij.2fZX06dIIWJAjkK20F7ahvXUNXXm",
            "id": "none",
            "keepers": null,
            "length": 16,
            "lower": true,
            "min_lower": 1,
            "min_numeric": 1,
            "min_special": 0,
            "min_upper": 1,
            "number": true,
            "numeric": true,
            "override_special": null,
            "result": "FQC4KxfgUPko7s50",   <--------------------
            "special": false,
            "upper": true

Ошибка 1:
resource "docker_image" {
Проблема: У ресурса отсутствует имя (второй label).

Ошибка 2:
resource "docker_container" "1nginx" {
Проблема: Имя ресурса начинается с цифры, что недопустимо.

Ошибка 3:
name  = "example_${random_password.random_string_FAKE.resulT}"
Проблемы:
-- random_string_FAKE - такого ресурса нет, правильное имя random_string
-- resulT - неправильное написание атрибута, должно быть result

 ### Фрагмент кода
avfomichev@netology:~/terr1/ter-homeworks/01/src$ cat main.tf 
terraform {
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
    }
  }
  required_version = ">=1.12.0" /*Многострочный комментарий.
 Требуемая версия terraform */
}
provider "docker" {}

#однострочный комментарий

resource "random_password" "random_string" {
  length      = 16
  special     = false
  min_upper   = 1
  min_lower   = 1
  min_numeric = 1
}


resource "docker_image" "nginx" {
  name         = "nginx:latest"
  keep_locally = true
}

resource "docker_container" "nginx" {
  image = docker_image.nginx.image_id
  name  = "hello_world"

  ports {
    internal = 80
    external = 9090
  }
}
  
   
![alt text](https://github.com/SLzDevOps/netology-terraform-1/blob/main/Screenshot_752.png).
![alt text](https://github.com/SLzDevOps/netology-terraform-1/blob/main/Screenshot_751.png).


Опасность ключа -auto-approve:
Удаление ресурсов без подтверждения - если в коде есть удаление ресурсов, Terraform выполнит его сразу без запроса подтверждения
Случайные изменения - можно не заметить критические изменения в плане выполнения
Отсутствие контроля - нет возможности проверить план перед применением
Каскадные эффекты - изменения в одном ресурсе могут повлиять на другие непредвиденным образом

Зачем нужен -auto-approve:
CI/CD пайплайны - в автоматизированных системах развертывания нет человека, который может подтвердить изменения
Тестовые среды - где быстрота развертывания важнее проверки
Скрипты и автоматизация - при вызове Terraform из других скриптов
Обновление non-critical инфраструктуры - где изменения предсказуемы и безопасны
Dev-среда - разработчики могут быстро применять изменения без лишних подтверждений

Правило безопасности: В production среде крайне не рекомендуется использовать -auto-approve без дополнительных проверок в пайплайне (review, тесты, валидация).


![alt text](https://github.com/SLzDevOps/netology-terraform-1/blob/main/Screenshot_753.png).



  В ресурсе docker_image есть параметр keep_locally = true:

resource "docker_image" "nginx" {
  name         = "nginx:latest"
  keep_locally = true
}

keep_locally - (Optional, boolean) If true, the image will be kept locally after the resource is destroyed. Defaults to false.
keep_locally - (Опционально, логическое значение) Если true, образ будет сохранен локально после уничтожения ресурса. По умолчанию false.

Более подробно:
keep_locally (Boolean) If true, the image will be kept when the resource is destroyed. If false or not specified, the image will be removed when the resource is destroyed. Defaults to false.
keep_locally (Логическое значение) Если true, образ будет сохранен при уничтожении ресурса. Если false или не указано, образ будет удален при уничтожении ресурса. По умолчанию false.



  
### Задача 3

Непонравилась строка с версией, установлено -   required_version = ">= 1.6.0, < 2.0.0" и добавлено регистри
avfomichev@netology:~$ cat .terraformrc 
provider_installation {
  network_mirror {
    url = "https://terraform-mirror.yandexcloud.net/"
    include = ["registry.terraform.io/*/*", "registry.opentofu.org/*/*"]
  }
  direct {
    exclude = ["registry.terraform.io/*/*", "registry.opentofu.org/*/*"]
  }
}

Скриншоты выполнения:
![alt text](https://github.com/SLzDevOps/netology-terraform-1/blob/main/Screenshot_754.png).
![alt text](https://github.com/SLzDevOps/netology-terraform-1/blob/main/Screenshot_755.png).

