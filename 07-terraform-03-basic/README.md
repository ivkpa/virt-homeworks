# Домашнее задание к занятию "7.3. Основы и принцип работы Терраформ"

## Задача 1. Создадим бэкэнд в S3 (необязательно, но крайне желательно).

Если в рамках предыдущего задания у вас уже есть аккаунт AWS, то давайте продолжим знакомство со взаимодействием
терраформа и aws. 

1. Создайте s3 бакет, iam роль и пользователя от которого будет работать терраформ. Можно создать отдельного пользователя,
а можно использовать созданного в рамках предыдущего задания, просто добавьте ему необходимы права, как описано 
[здесь](https://www.terraform.io/docs/backends/types/s3.html).
1. Зарегистрируйте бэкэнд в терраформ проекте как описано по ссылке выше. 


## Задача 2. Инициализируем проект и создаем воркспейсы. 

1. Выполните `terraform init`:
    * если был создан бэкэнд в S3, то терраформ создат файл стейтов в S3 и запись в таблице 
dynamodb.
    * иначе будет создан локальный файл со стейтами.  
1. Создайте два воркспейса `stage` и `prod`.
1. В уже созданный `aws_instance` добавьте зависимость типа инстанса от вокспейса, что бы в разных ворскспейсах 
использовались разные `instance_type`.
1. Добавим `count`. Для `stage` должен создаться один экземпляр `ec2`, а для `prod` два. 
1. Создайте рядом еще один `aws_instance`, но теперь определите их количество при помощи `for_each`, а не `count`.
1. Что бы при изменении типа инстанса не возникло ситуации, когда не будет ни одного инстанса добавьте параметр
жизненного цикла `create_before_destroy = true` в один из рессурсов `aws_instance`.
1. При желании поэкспериментируйте с другими параметрами и рессурсами.

В виде результата работы пришлите:
* Вывод команды `terraform workspace list`.  
<b>Ответ:</b>  
```
root@ubuntu01:/home/user/tf-syntax# terraform workspace list
  default
* prod
  stage
```
* Вывод команды `terraform plan` для воркспейса `prod`.  
<b>Ответ:</b>  
```
root@ubuntu01:/home/user/tf-syntax# terraform plan

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # yandex_compute_instance.node["node01"] will be created
  + resource "yandex_compute_instance" "node" {
      + created_at                = (known after apply)
      + folder_id                 = (known after apply)
      + fqdn                      = (known after apply)
      + hostname                  = "node01.netology.yc"
      + id                        = (known after apply)
      + metadata                  = {
          + "ssh-keys" = <<-EOT
                centos:ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDBLNbC7+y/n1a1HudiwsIBRqUc5XzxB06NE+nuNhh5v8TL/ysDq77uYb82m70W0pCOm8FaWDoFWxNikkbZifkRiEdymVqkNfWHCSSLw4gwOSVZUPRcN9EutjghTnm6NrzgBwc5btv1ajsyOLgmeSqOXdp2oKxQL3/WujJEASyujIgPLx41/7pDB7QlQEgO77hOsuhrHnKCAq1CHhGU2oXZN7Yfpe5L0UigcDeSK5BifyCz7ot8ie65oa2HXgEkcsNO7zhHYFH8Xekmltpg+3OdWQ0cqSfLG3c00RC2jsYGE7HRvEhSU139nHD8UZMaSAa32FwYD6IGdVrCPAkvzQkNJIca0TlxJszdw/7IEH0+oL5Y5IEpLsb6A+DG6kKiqbyXqWiOTbbjquUF7tz23hCfl6oTtZ6Xeq3T3L2LZ918xFI4q20cMCANDXQq+H1zznY1UgoZBZ26ixcn+G9e5x/HJ/M2qZzpSxHAjZJfiU3fE6A0lAfsOGE1N98kCY0B9AM= root@ubuntu01
            EOT
        }
      + name                      = "node01"
      + network_acceleration_type = "standard"
      + platform_id               = "standard-v1"
      + service_account_id        = (known after apply)
      + status                    = (known after apply)
      + zone                      = "ru-central1-a"

      + boot_disk {
          + auto_delete = true
          + device_name = (known after apply)
          + disk_id     = (known after apply)
          + mode        = (known after apply)

          + initialize_params {
              + block_size  = (known after apply)
              + description = (known after apply)
              + image_id    = "fd89n3ii46a2m3uffmdu"
              + name        = "root-node01"
              + size        = 10
              + snapshot_id = (known after apply)
              + type        = "network-nvme"
            }
        }

      + network_interface {
          + index              = (known after apply)
          + ip_address         = "192.168.100.11"
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
          + host_affinity_rules = (known after apply)
          + placement_group_id  = (known after apply)
        }

      + resources {
          + core_fraction = 100
          + cores         = 2
          + memory        = 4
        }

      + scheduling_policy {
          + preemptible = false
        }
    }

  # yandex_compute_instance.node["node02"] will be created
  + resource "yandex_compute_instance" "node" {
      + created_at                = (known after apply)
      + folder_id                 = (known after apply)
      + fqdn                      = (known after apply)
      + hostname                  = "node02.netology.yc"
      + id                        = (known after apply)
      + metadata                  = {
          + "ssh-keys" = <<-EOT
                centos:ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDBLNbC7+y/n1a1HudiwsIBRqUc5XzxB06NE+nuNhh5v8TL/ysDq77uYb82m70W0pCOm8FaWDoFWxNikkbZifkRiEdymVqkNfWHCSSLw4gwOSVZUPRcN9EutjghTnm6NrzgBwc5btv1ajsyOLgmeSqOXdp2oKxQL3/WujJEASyujIgPLx41/7pDB7QlQEgO77hOsuhrHnKCAq1CHhGU2oXZN7Yfpe5L0UigcDeSK5BifyCz7ot8ie65oa2HXgEkcsNO7zhHYFH8Xekmltpg+3OdWQ0cqSfLG3c00RC2jsYGE7HRvEhSU139nHD8UZMaSAa32FwYD6IGdVrCPAkvzQkNJIca0TlxJszdw/7IEH0+oL5Y5IEpLsb6A+DG6kKiqbyXqWiOTbbjquUF7tz23hCfl6oTtZ6Xeq3T3L2LZ918xFI4q20cMCANDXQq+H1zznY1UgoZBZ26ixcn+G9e5x/HJ/M2qZzpSxHAjZJfiU3fE6A0lAfsOGE1N98kCY0B9AM= root@ubuntu01
            EOT
        }
      + name                      = "node02"
      + network_acceleration_type = "standard"
      + platform_id               = "standard-v1"
      + service_account_id        = (known after apply)
      + status                    = (known after apply)
      + zone                      = "ru-central1-a"

      + boot_disk {
          + auto_delete = true
          + device_name = (known after apply)
          + disk_id     = (known after apply)
          + mode        = (known after apply)

          + initialize_params {
              + block_size  = (known after apply)
              + description = (known after apply)
              + image_id    = "fd89n3ii46a2m3uffmdu"
              + name        = "root-node02"
              + size        = 10
              + snapshot_id = (known after apply)
              + type        = "network-nvme"
            }
        }

      + network_interface {
          + index              = (known after apply)
          + ip_address         = "192.168.100.12"
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
          + host_affinity_rules = (known after apply)
          + placement_group_id  = (known after apply)
        }

      + resources {
          + core_fraction = 100
          + cores         = 2
          + memory        = 4
        }

      + scheduling_policy {
          + preemptible = false
        }
    }

  # yandex_vpc_network.default will be created
  + resource "yandex_vpc_network" "default" {
      + created_at                = (known after apply)
      + default_security_group_id = (known after apply)
      + folder_id                 = (known after apply)
      + id                        = (known after apply)
      + labels                    = (known after apply)
      + name                      = "net"
      + subnet_ids                = (known after apply)
    }

  # yandex_vpc_subnet.subnet01 will be created
  + resource "yandex_vpc_subnet" "subnet01" {
      + created_at     = (known after apply)
      + folder_id      = (known after apply)
      + id             = (known after apply)
      + labels         = (known after apply)
      + name           = "subnet01"
      + network_id     = (known after apply)
      + v4_cidr_blocks = [
          + "192.168.100.0/24",
        ]
      + v6_cidr_blocks = (known after apply)
      + zone           = "ru-central1-a"
    }

Plan: 4 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + external_network_addresses = {
      + node01 = (known after apply)
      + node02 = (known after apply)
    }
  + internal_network_addresses = {
      + node01 = "192.168.100.11"
      + node02 = "192.168.100.12"
    }
```    

<b>main.tf</b>  
```
locals {
  node_core_fraction_map = {
    stage = 20
    prod = 100
  }
  node_schedule_policy_preemptible_map = {
    stage = true
    prod = false
  }

  instances = {
    "node01" = "192.168.100.11"
    "node02" = "192.168.100.12"

  }
}

resource "yandex_vpc_network" "default" {
    name = "net"
}

resource "yandex_vpc_subnet" "subnet01" {
    name = "subnet01"
    zone           = "ru-central1-a"
    network_id     = "${yandex_vpc_network.default.id}"
    v4_cidr_blocks = ["192.168.100.0/24"]
}


resource "yandex_compute_instance" "node" {

  for_each = local.instances

  name                      = each.key
  zone                      = "ru-central1-a"
  hostname                  = "${each.key}.netology.yc"

  resources {
    cores  = "2"
    memory = "4"
    core_fraction = local.node_core_fraction_map[terraform.workspace]
  }

  boot_disk {
    initialize_params {
      image_id    = "fd89n3ii46a2m3uffmdu"
      name        = "root-${each.key}"
      type        = "network-nvme"
      size        = "10"
    }
  }

  network_interface {
    subnet_id  = "${yandex_vpc_subnet.subnet01.id}"
    nat        = true
    ip_address = each.value
  }

  scheduling_policy {
    preemptible = local.node_schedule_policy_preemptible_map[terraform.workspace]
}

  metadata = {
    ssh-keys = "centos:${file("~/.ssh/id_rsa.pub")}"
  }
}
```

---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
