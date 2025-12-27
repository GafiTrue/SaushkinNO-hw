# Домашнее задание к занятию "Система мониторинга Zabbix" - Саушкин Николай


---

### Задание 1

Создан шаблон `My Template CPU RAM` с двумя элементами данных:

1. **CPU utilization** — мониторинг загрузки CPU
   - Type: Zabbix agent
   - Key: `system.cpu.util`
   - Units: %
   - Update interval: 1m

2. **Memory utilization** — мониторинг загрузки RAM
   - Type: Zabbix agent
   - Key: `vm.memory.size[<mode>]`
   - Units: %
   - Update interval: 1m

<img width="1280" height="524" alt="image" src="https://github.com/user-attachments/assets/4edf9bd1-8491-4ea9-8eff-f62b3db25faf" />

---

### Задание 2-3

Установлен Zabbix Agent на 2 виртуальные машины:

- **saushkinNO-1** (192.168.195.134) — агент
- **saushkinNO-2** (127.0.0.1) — Zabbix Server

К обоим хостам привязаны шаблоны:
- Linux by Zabbix agent
- My Template CPU RAM

<img width="1280" height="643" alt="image" src="https://github.com/user-attachments/assets/c1d44339-3127-43b4-9176-9d29e090e13f" />


---

### Задание 4

Создан кастомный дашборд с графиками:

- График загрузки CPU (CPU utilization)
- График загрузки RAM (Memory utilization)

<img width="722" height="694" alt="image" src="https://github.com/user-attachments/assets/4ba8dba3-c094-42fb-a042-3670d33e7158" />

