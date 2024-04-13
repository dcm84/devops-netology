# Домашнее задание к занятию 10 «Jenkins»

## Подготовка к выполнению

1. Создать два VM: для jenkins-master и jenkins-agent.
2. Установить Jenkins при помощи playbook.
3. Запустить и проверить работоспособность.
4. Сделать первоначальную настройку.
![alt text](image.png)
## Основная часть

1. Сделать Freestyle Job, который будет запускать `molecule test` из любого вашего репозитория с ролью.
![alt text](image-1.png)
![alt text](image-2.png)
![alt text](image-3.png)
![alt text](image-4.png)
...
![alt text](image-5.png)
2. Сделать Declarative Pipeline Job, который будет запускать `molecule test` из любого вашего репозитория с ролью.
![alt text](image-6.png)
![alt text](image-7.png)
3. Перенести Declarative Pipeline в репозиторий в файл `Jenkinsfile`.
[Jenkinsfile](https://github.com/dcm84/devops-netology/blob/main/ci-04-jenkins/Jenkinsfile)
![alt text](image-8.png)
4. Создать Multibranch Pipeline на запуск `Jenkinsfile` из репозитория.
![alt text](image-9.png)
5. Создать Scripted Pipeline, наполнить его скриптом из [pipeline](./pipeline).

6. Внести необходимые изменения, чтобы Pipeline запускал `ansible-playbook` без флагов `--check --diff`, если не установлен параметр при запуске джобы (prod_run = True). По умолчанию параметр имеет значение False и запускает прогон с флагами `--check --diff`.Исправленный Pipeline вложить в репозиторий в файл `ScriptedJenkinsfile`.
[ScriptedJenkinsfile](ScriptedJenkinsfile)


