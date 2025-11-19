---
title: Ansible
icon: lucide/workflow
---

# Ansible - инструмент автоматизации

Ansible - это система управления конфигурациями, которая упрощает автоматизацию задач деплоя и управления конфигурацией серверов, без необходимости установки агентов на целевые системы.

## Основные концепции

- **Агентная модель**: Ansible использует SSH для выполнения задач на удаленных машинах (не требуется установка агентов)
- **Идемпотентность**: Операции могут быть применены многократно без изменения конечного состояния системы
- **YAML-форматирование**: Playbooks и конфигурации записываются в читаемом формате YAML
- **Инвентарь**: Список целевых хостов, организованный в группы

## Содержание раздела

- [Установка и настройка Ansible](installation.md) - Гайд по установке и первоначальной настройке
- [Написание Playbooks](playbooks.md) - Руководство по созданию и структурированию плейбуков
- [Ansible Roles](roles.md) - Организация кода с помощью ролей
- [Инвентаризация](inventory.md) - Настройка и управление инвентарем
- [Лучшие практики](best-practices.md) - Рекомендации по организации проектов

## Основные команды Ansible

### Проверка доступности хостов

```bash
# Пинг всех хостов в инвентаре
ansible all -m ping

# Пинг конкретной группы
ansible webservers -m ping
```

### Выполнение команд

```bash
# Выполнение ad-hoc команды на всех хостах
ansible all -a "uptime"

# Выполнение с повышением привилегий
ansible all -a "apt update" -b
```

### Работа с Playbooks

```bash
# Запуск плейбука
ansible-playbook deploy.yml

# Запуск с ограничением на конкретные хосты
ansible-playbook deploy.yml -l webservers

# Проверка синтаксиса плейбука
ansible-playbook deploy.yml --syntax-check

# Dry-run (показать изменения без применения)
ansible-playbook deploy.yml --check
```

## Пример простого Playbook

```yaml
---
- name: Установка и настройка Nginx
  hosts: webservers
  become: true
  
  tasks:
    - name: Установка Nginx
      apt:
        name: nginx
        state: present
        update_cache: yes
      
    - name: Запуск и включение сервиса Nginx
      systemd:
        name: nginx
        state: started
        enabled: yes
      
    - name: Создание корневого каталога для сайта
      file:
        path: /var/www/example
        state: directory
        owner: www-data
        group: www-data
        mode: '0755'
      
    - name: Создание конфигурации виртуального хоста
      template:
        src: templates/nginx-vhost.conf.j2
        dest: /etc/nginx/sites-available/example.conf
      notify: reload nginx
      
    - name: Включение сайта
      file:
        src: /etc/nginx/sites-available/example.conf
        dest: /etc/nginx/sites-enabled/example.conf
        state: link
      notify: reload nginx
  
  handlers:
    - name: reload nginx
      systemd:
        name: nginx
        state: reloaded
```

## Полезные ресурсы

- [Официальная документация Ansible](https://docs.ansible.com/)
- [Ansible Galaxy](https://galaxy.ansible.com/) - репозиторий готовых ролей
- [AWX](https://github.com/ansible/awx) - веб-интерфейс управления Ansible (открытая версия Ansible Tower)
