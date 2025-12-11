# Домашнее задание к занятию "`Подъём инфраструктуры в Yandex Cloud`" - `Саушкин Николай Олегович`

---

### Задание 1
Задание 1: Развернуть VPC, 2 веб-сервера, бастион
`Приведите ответ в свободной форме........`

Файл meta.txt
```
#cloud-config
users:
  - name: user
    groups: sudo
    shell: /bin/bash
    sudo: ['ALL=(ALL) NOPASSWD:ALL']
    ssh-authorized-keys:
      - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQC03G0kUsV1cySUvp/boZln7Fab6sqayAsFbzvNoFOi1r89RtjpWp/NigiqRSvB5K/0SMYfX457z3ekON/LyRWAHPuG5NDaBgb5nKm7eeEeC8wbD8Hp7YyhzU43Je50497HLpX/HJ6cb5iCMUjXDoo0xcbeu1JbEz52xnLOE/OeoDG7gVTE/RnPUO3sXKlal0iE5p3Y7DPHAnPBn2Sv3pbfPyUkdPkIoZAEvTpVEiuWykxnWCcPlNc8cBbjynYqCLaJz7K0h5k9ufk1xJw+CcGS+Y0NBk6hJr+0TaAOtn/7yHpUmdyppDRStfacnQu8DQuk1bDnU6QatewRk0UyuEuSeiLhMihNovJZEvioujHrBzF3iZ1KUo57BPzdXB4rHrUu329TT9pIOBQ4++o/va9MdmmfVo5WJg+MQ3FQh5ff9bzsTDGFbTpOnKuwiY69FO+YMm0idJZELByB2wuURHUAN+KkNcIpzo1vRkmIvU1UqncLQa0HxLCEj8eSYmRy3l9nMveaLui/JSBk1A7tDLWPrpnlOHHql2wCtjiVCRmJagPLTSV4VIlGM3C9SU2W80sfD+yxcD1moQ4+xr0Lisrsxbf1S1JQpmgQ1Bd+WoXFhs2KEWYExsr02LF/mYZlCrr3NUKszEzLSFx6DccTJTAZKMgXgRsj/Cg14yvOW+XAcw== nikolay@nikolay-VMware-Virtual-Platform
```

Файл main.tf
```
terraform {
  required_providers {
    yandex = {
      source = "yandex-cloud/yandex"
    }
  }
}

provider "yandex" {
  token     = "OAuth_токен"
  cloud_id  = "b1g4i65mi6ibc9vhu838"
  folder_id = "b1gm7bqp3rg3ek7turqi"
  zone      = "ru-central1-a"
}

resource "yandex_vpc_network" "network-1" {
  name = "network-1"
}

resource "yandex_vpc_subnet" "subnet-1" {
  name           = "subnet-1"
  zone           = "ru-central1-a"
  network_id     = yandex_vpc_network.network-1.id
  v4_cidr_blocks = ["192.168.10.0/24"]
}

resource "yandex_compute_instance" "bastion" {
  name        = "bastion"
  platform_id = "standard-v1"
  zone        = "ru-central1-a"
  resources {
    cores  = 2
    memory = 2
  }
  boot_disk {
    initialize_params {
      image_id = "fd8s4a9mnca2bmgol2r8"
      size     = 10
    }
  }
  network_interface {
    subnet_id = yandex_vpc_subnet.subnet-1.id
    nat       = true
  }
  metadata = {
    user-data = file("./meta.txt")
  }
}

resource "yandex_compute_instance" "web-a" {
  name        = "web-a"
  platform_id = "standard-v1"
  zone        = "ru-central1-a"
  resources {
    cores  = 2
    memory = 2
  }
  boot_disk {
    initialize_params {
      image_id = "fd8s4a9mnca2bmgol2r8"
      size     = 10
    }
  }
  network_interface {
    subnet_id = yandex_vpc_subnet.subnet-1.id
    nat       = true
  }
  metadata = {
    user-data = file("./meta.txt")
  }
}

resource "yandex_compute_instance" "web-b" {
  name        = "web-b"
  platform_id = "standard-v1"
  zone        = "ru-central1-a"
  resources {
    cores  = 2
    memory = 2
  }
  boot_disk {
    initialize_params {
      image_id = "fd8s4a9mnca2bmgol2r8"
      size     = 10
    }
  }
  network_interface {
    subnet_id = yandex_vpc_subnet.subnet-1.id
    nat       = true
  }
  metadata = {
    user-data = file("./meta.txt")
  }
}

output "bastion_ip" {
  value = yandex_compute_instance.bastion.network_interface.0.nat_ip_address
}

output "web_a_ip" {
  value = yandex_compute_instance.web-a.network_interface.0.nat_ip_address
}

output "web_b_ip" {
  value = yandex_compute_instance.web-b.network_interface.0.nat_ip_address
}
```

Результат выполнения terraform apply
![photo_2025-12-12_00-17-53](https://github.com/user-attachments/assets/d44e605f-edac-429e-97aa-4c143f32e107)

![photo_2025-12-12_00-18-09](https://github.com/user-attachments/assets/97f7e611-e8b7-4cb8-bd89-e39d7edb0338)



---

### Задание 2

Установка Nginx через Ansible
Файл inventory.ini
```
[webservers]
web-a ansible_host=89.169.157.245
web-b ansible_host=158.160.49.218

[webservers:vars]
ansible_user=user
ansible_ssh_private_key_file=~/.ssh/id_rsa
```
Файл nginx.yml (Ansible Playbook)
```
---
- name: Install Nginx on web servers
  hosts: webservers
  become: yes

  tasks:
    - name: Update apt cache
      ansible.builtin.apt:
        update_cache: yes

    - name: Install Nginx
      ansible.builtin.apt:
        name: nginx
        state: present

    - name: Start and enable Nginx
      ansible.builtin.systemd:
        name: nginx
        state: started
        enabled: yes
```
![photo_2025-12-12_00-19-26](https://github.com/user-attachments/assets/47e9161f-9646-43fa-9cc4-e05b59bbe1af)
![photo_2025-12-12_00-19-36](https://github.com/user-attachments/assets/8a8fc6c1-16ba-4f3f-ac80-58b4acd6a48e)

