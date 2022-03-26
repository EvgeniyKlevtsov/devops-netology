## Домашнее задание к занятию "7.2. Облачные провайдеры и синтаксис Терраформ."

### Задача 1 (вариант с AWS). Регистрация в aws и знакомство с основами (необязательно, но крайне желательно).

К сожалению зарегистрироваться в AWS не удалось. Поэтому выполнять задание будем в Yandex.Cloud.

### Задача 1 (вариант с Yandex.Cloud). Регистрация в ЯО и знакомство с основами (необязательно, но крайне желательно).
Регистрацию, авторизацию и первичную настройку мы проходили на занятии "Оркестрация группой Docker контейнеров на примере Docker Compose."
1.  Воспользуйтесь  [инструкцией](https://registry.terraform.io/providers/yandex-cloud/yandex/latest/docs)  на сайте терраформа, что бы не указывать авторизационный токен в коде, а терраформ провайдер брал его из переменных окружений.

Для этого создаем файл `variable.tf` в котором указываем наши переменные:
```
variable "yc_token" {
  type        = string
  description = "AQA...."
}

variable "yc_region" {
  type        = string
  description = "ru-central1-a"
}

variable "yc_cloud_id" {
  type        = string
  description = "b1ga3eofr6rp88nrlnlj"
}

variable "yc_folder_id" {
  type        = string
  description = "b1g8teguto487h0gvuvp"
}
```
### Задача 2. Создание aws ec2 или yandex_compute_instance через терраформ.

В каталоге `terraform` создадим файл манифест `main.tf` с содержимым:

```
terraform {
  required_providers {
    yandex = {
      source = "terraform-registry.storage.yandexcloud.net/yandex-cloud/yandex"
    }
  }
  required_version = ">= 0.13"
}

# <настройки провайдера с нашими переменными >

provider "yandex" {
  token = var.yc_token
  cloud_id  = var.yc_cloud_id
  folder_id = var.yc_folder_id
  zone = var.yc_region
}

resource "yandex_compute_instance" "vm-1" {
  name = "terraform1"

  resources {
    cores  = 2
    memory = 2
  }

  boot_disk {
    initialize_params {
      image_id = "fd80jfslq61mssea4ejn"
    }
  }

  network_interface {
    subnet_id = yandex_vpc_subnet.subnet.id
    nat       = true
  }

  metadata = {
    ssh-keys = "ubuntu:${file("~/.ssh/id_rsa.pub")}"
  }
}

resource "yandex_vpc_network" "net" {
  name = "network"
}

resource "yandex_vpc_subnet" "subnet" {
  name           = "subnet"
  zone           = "ru-central1-a"
  network_id     = "${yandex_vpc_network.net.id}"
  v4_cidr_blocks = ["192.168.10.0/24"]
}

output "internal_ip_address_vm_1" {
  value = yandex_compute_instance.vm-1.network_interface.0.ip_address
}

output "external_ip_address_vm_1" {
  value = yandex_compute_instance.vm-1.network_interface.0.nat_ip_address
}
```
Для поиска образа Ubuntu используем команду:
`yc compute image list --folder-id standard-images` 
Находим `image_id` и вставляем его соответствующую строку.
```
boot_disk {
    initialize_params {
      image_id = "fd80jfslq61mssea4ejn"
    }
  }
```
После выполняем команду `terraform validate` . Если конфигурация является допустимой, появится сообщение: `Success! The configuration is valid.`
После проверки конфигурации выполните команду: `terraform plan`. В терминале будет выведен список ресурсов с параметрами. Это проверочный этап: ресурсы не будут созданы. Если в конфигурации есть ошибки, Terraform на них укажет.
Результат выполнения `terraform plan`:
```
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
      + metadata                  = {
          + "ssh-keys" = <<-EOT
                ubuntu:ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDBaZ3o9KvWBO9P09mvM5iHo6OZTX3eN2paSrhUJpRN2I7A7H8/2IcDULJjJZC25c8JtSPBk8rg9BabEc7SBblCPszX5PaGi5Z+DdXfFjC6wTxGScvx7kmHZ2EKRpBUlrO5TKahHxIVlvwccXKfAlJmd6D4mECTYx+wQRkyweh4GJp/95959PMqqSwPNcs+Hm0Pvxgupax4c4l00F/dgzV/xi6Vw4BDDmjOIWYgZsjSOtXdFqWqzAY/R2VCCmFvInUsShHl2d3ZokXwv2ixvKWpMWUrHeS3NFKz66Z7YgC7o8yLvtB9kOjE9oWp/pLRp0ccdhKfixcvTyBONDQVjd7R evgeniy@home
            EOT
        }
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

─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.
```
Чтобы создать ресурсы выполните команду: `terraform apply`.  Подтвердите создание ресурсов: введите в терминал слово  `yes`  и нажмите  **Enter**.

Чтобы удалить ресурсы, созданные с помощью Terraform: `terraform destroy`

При помощи какого инструмента (из разобранных на прошлом занятии) можно создать свой образ ami?
При помощи **Packer**.
