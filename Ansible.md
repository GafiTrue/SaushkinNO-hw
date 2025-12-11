# Домашнее задание к занятию "`Ansible. Часть 2`" - `Саушкин Николай Олегович`
### Задание 1

Плейбук 1: Скачивание и распаковка архива Apache Kafka
```
---
- name: Download and extract Apache Kafka
  hosts: all
  become: yes

  vars:
    kafka_url: "https://archive.apache.org/dist/kafka/3.6.1/kafka_2.13-3.6.1.tgz"
    kafka_dir: "/opt/kafka"

  tasks:
    - name: Install tar
      ansible.builtin.package:
        name: tar
        state: present

    - name: Download Kafka archive
      ansible.builtin.get_url:
        url: "{{ kafka_url }}"
        dest: /tmp/kafka.tgz

    - name: Create directory
      ansible.builtin.file:
        path: "{{ kafka_dir }}"
        state: directory

    - name: Extract archive
      ansible.builtin.unarchive:
        src: /tmp/kafka.tgz
        dest: "{{ kafka_dir }}"
        remote_src: yes
        extra_opts: [--strip-components=1]
```

Результат выполнения:
![photo_2025-12-11_22-58-07](https://github.com/user-attachments/assets/1694eca3-c987-4f9d-a588-efb422df99d5)

Плейбук 2: Установка и настройка tuned
```
---
- name: Install and configure tuned
  hosts: all
  become: yes

  tasks:
    - name: Install tuned (RedHat)
      ansible.builtin.dnf:
        name: tuned
        state: present
      when: ansible_os_family == "RedHat"

    - name: Install tuned (Debian)
      ansible.builtin.apt:
        name: tuned
        state: present
        update_cache: yes
      when: ansible_os_family == "Debian"

    - name: Start tuned
      ansible.builtin.systemd:
        name: tuned
        state: started

    - name: Enable tuned on boot
      ansible.builtin.systemd:
        name: tuned
        enabled: yes
```

Результат выполнения:
![photo_2025-12-11_23-04-57](https://github.com/user-attachments/assets/dc92e166-747b-4ecf-bded-962e5c145cf1)

Плейбук 3: Изменение MOTD с использованием переменной
```
---
- name: Configure MOTD
  hosts: all
  become: yes

  vars:
    motd_message: |
      ================================
        Welcome to the server!
        Managed by Ansible
      ================================

  tasks:
    - name: Set MOTD
      ansible.builtin.copy:
        content: "{{ motd_message }}"
        dest: /etc/motd
```

Результат выполнения:
![photo_2025-12-11_23-05-56](https://github.com/user-attachments/assets/298c4a41-a25a-47d6-84c9-72077526c7af)

---

### Задание 2
Модифицированный плейбук MOTD
Плейбук устанавливает приветствие с IP-адресом, hostname и пожеланием хорошего дня системному администратору.

```
---
- name: Configure advanced MOTD
  hosts: all
  become: yes
  gather_facts: yes

  vars:
    greeting: "Хорошего дня, системный администратор!"

  tasks:
    - name: Set MOTD with system info
      ansible.builtin.copy:
        content: |
          ================================
          Hostname:   {{ ansible_hostname }}
          IP-адрес:   {{ ansible_default_ipv4.address }}
          ================================
          {{ greeting }}
          ================================
        dest: /etc/motd
```

![photo_2025-12-11_23-06-15](https://github.com/user-attachments/assets/d2d03f84-ca29-4857-9720-256b270d65e0)



---

### Задание 3
Apache
Плейбук site.yml
```
---
- name: Deploy Apache Web Server
  hosts: all
  become: yes
  gather_facts: yes

  roles:
    - apache_webserver
```

tasks/main.yml
```
---
- name: Install Apache (RedHat)
  ansible.builtin.dnf:
    name: httpd
    state: present
  when: ansible_os_family == "RedHat"
  notify: Restart Apache

- name: Install Apache (Debian)
  ansible.builtin.apt:
    name: apache2
    state: present
    update_cache: yes
  when: ansible_os_family == "Debian"
  notify: Restart Apache

- name: Open port 80 (firewalld)
  ansible.posix.firewalld:
    port: 80/tcp
    permanent: yes
    state: enabled
    immediate: yes
  when: ansible_os_family == "RedHat"
  ignore_errors: yes

- name: Deploy index.html
  ansible.builtin.template:
    src: index.html.j2
    dest: /var/www/html/index.html
  notify: Restart Apache

- name: Start Apache
  ansible.builtin.systemd:
    name: "{{ 'httpd' if ansible_os_family == 'RedHat' else 'apache2' }}"
    state: started
    enabled: yes

- name: Check website (HTTP 200)
  ansible.builtin.uri:
    url: "http://{{ ansible_default_ipv4.address }}/"
    status_code: 200
```

handlers/main.yml
```
---
- name: Restart Apache
  ansible.builtin.systemd:
    name: "{{ 'httpd' if ansible_os_family == 'RedHat' else 'apache2' }}"
    state: restarted
```

templates/index.html.j2
```
<!DOCTYPE html>
<html>
<head>
    <title>Server Info - {{ ansible_hostname }}</title>
    <style>
        body { font-family: Arial; margin: 40px; background: #f0f0f0; }
        .container { background: white; padding: 30px; border-radius: 10px; max-width: 600px; }
        h1 { color: #333; }
        table { width: 100%; border-collapse: collapse; }
        td { padding: 10px; border-bottom: 1px solid #ddd; }
        .label { font-weight: bold; width: 40%; }
    </style>
</head>
<body>
    <div class="container">
        <h1>Информация о сервере</h1>
        <table>
            <tr><td class="label">Hostname:</td><td>{{ ansible_hostname }}</td></tr>
            <tr><td class="label">IP-адрес:</td><td>{{ ansible_default_ipv4.address }}</td></tr>
            <tr><td class="label">CPU:</td><td>{{ ansible_processor[2] }} ({{ ansible_processor_cores }} cores)</td></tr>
            <tr><td class="label">RAM:</td><td>{{ (ansible_memtotal_mb / 1024) | round(2) }} GB</td></tr>
            <tr><td class="label">HDD:</td><td>{% for dev, info in ansible_devices.items() %}{% if dev.startswith('sd') or dev.startswith('vd') %}{{ dev }}: {{ info.size }}{% endif %}{% endfor %}</td></tr>
        </table>
        <p><small>Generated by Ansible</small></p>
    </div>
</body>
</html>
```
![photo_2025-12-11_23-06-52](https://github.com/user-attachments/assets/a8e4aa3a-2696-4230-91af-b5581616f6cb)


https://drive.google.com/file/d/1MAOr108rQ1niW0VIHDj15QUWAkyKHHm9/view?usp=sharing

![photo_2025-12-11_23-07-24](https://github.com/user-attachments/assets/cb18c552-ec0c-4e3b-b46f-c8ec0f6c24c1)
