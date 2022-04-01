## Домашнее задание к занятию "7.3. Основы и принцип работы Терраформ"

### Задача 1. Создадим бэкэнд в S3 (необязательно, но крайне желательно).
### Задача 2. Инициализируем проект и создаем воркспейсы.

Создадим бакет для хранения состояния s3 через web gui от YC:
![imageban](https://i6.imageban.ru/out/2022/04/01/cada6679782ceaa900250d38c0d9e3ea.png)
Создадим файл `main.tf` и опишем в нём подключение к яндексу и его s3:
```go
terraform {
  required_providers {
    yandex = {
      source = "yandex-cloud/yandex"
    }
  }

  backend "s3" {
    endpoint   = "storage.yandexcloud.net"
    bucket     = "terrabasic-01"
    region     = "us-east-1"
    key        = "terrabasic-01.tfstate" #имя созданного бакета для хранения состояния терраформ
    access_key = "access_key"
    secret_key = "secret_key"

    skip_region_validation      = true
    skip_credentials_validation = true
  }
}

provider "yandex" {
  token     = token     "
  cloud_id  = "cloud_id"
  folder_id = "folder_id"
  zone      = "ru-central1-a"
}
```
Убедимся, что на текущий момент в проекте один воркспейс:
```
$ sudo terraform workspace list
* default
```
Создадим ещё 2 воркспейса:

-   stage
-   prod
```
$ sudo terraform workspace new stage
Created and switched to workspace "stage"!

$ sudo terraform workspace new prod
Created and switched to workspace "prod"!
```
Проверим, что воркспейсы появились:
```
$ terraform workspace list
  default
  stage
* prod
```
Сделаем образ виртуальной машины для деплоя, на основе образа Ubuntu, собранного ранее и выполним `terraform plan`.
Задеплоим образ, используя  `terraform apply --auto-approve`:  
`Apply complete! Resources: 3 added, 0 changed, 0 destroyed`  
Переключимся на другой вокрспейс, используя  `terraform workspace select prod`  и выполним  `terraform plan`:
```
sudo terraform plan

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # yandex_compute_instance.vm-1 will be created
  + resource "yandex_compute_instance" "vm-1" {
      + created_at                = (known after apply)
      + folder_id                 = (known after apply)
      + fqdn                      = (known after apply)
      + hostname                  = (known after apply)
      + id                        = (known after apply)
      + name                      = "terraform1"
      + network_acceleration_type = "standard"
      + platform_id               = "standard-v1"
      + service_account_id        = (known after apply)
      + status                    = (known after apply)
      + zone                      = (known after apply)

      + boot_disk {
          + auto_delete = true
          + device_name = (known after apply)
          + disk_id     = (known after apply)
          + mode        = (known after apply)

          + initialize_params {
              + block_size  = (known after apply)
              + description = (known after apply)
              + image_id    = "fd80jfslq61mssea4ejn"
              + name        = (known after apply)
              + size        = (known after apply)
              + snapshot_id = (known after apply)
              + type        = "network-hdd"
            }
        }

      + network_interface {
          + index              = (known after apply)
          + ip_address         = (known after apply)
          + ipv4               = true
          + ipv6               = (known after apply)
          + ipv6_address       = (known after apply)
          + mac_address        = (known after apply)
          + nat                = true
          + nat_ip_address     = (known after apply)
          + nat_ip_version     = (known after apply)
          + security_group_ids = (known after apply)
          + subnet_id          = (known after apply)
        }

      + placement_policy {
          + placement_group_id = (known after apply)
        }

      + resources {
          + core_fraction = 100
          + cores         = 2
          + memory        = 2
        }

      + scheduling_policy {
          + preemptible = (known after apply)
        }
    }

  # yandex_vpc_network.net will be created
  + resource "yandex_vpc_network" "net" {
      + created_at                = (known after apply)
      + default_security_group_id = (known after apply)
      + folder_id                 = (known after apply)
      + id                        = (known after apply)
      + labels                    = (known after apply)
      + name                      = "network"
      + subnet_ids                = (known after apply)
    }

  # yandex_vpc_subnet.subnet will be created
  + resource "yandex_vpc_subnet" "subnet" {
      + created_at     = (known after apply)
      + folder_id      = (known after apply)
      + id             = (known after apply)
      + labels         = (known after apply)
      + name           = "subnet"
      + network_id     = (known after apply)
      + v4_cidr_blocks = [
          + "192.168.10.0/24",
        ]
      + v6_cidr_blocks = (known after apply)
      + zone           = "ru-central1-a"
    }

Plan: 3 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + external_ip_address_vm_1 = (known after apply)
  + internal_ip_address_vm_1 = (known after apply)
```
Таким образом видим, что мы можем хранить разное состояние между разными воркспейсами.
Проверим, что бакеты в YC сохраняют наш tfstate:
[![imageban](https://i5.imageban.ru/out/2022/04/01/0d5775c64aa8d9ba0e5bc0817607c97f.png)](https://imageban.ru)
[![imageban](https://i7.imageban.ru/out/2022/04/01/736b8944f8d59de5e47eefee5fdcd4e8.png)](https://imageban.ru)
Таким образом, мы можем хранить наши воркспейсы в облаке вместе с состоянием. Удалим ресурсы, чтобы не тратить деньги при помощи `terraform destroy --auto-approve`
