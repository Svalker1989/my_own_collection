# Домашнее задание к занятию 6 «Создание собственных модулей»

## Подготовка к выполнению

1. Создайте пустой публичный репозиторий в своём любом проекте: `my_own_collection`.
2. Скачайте репозиторий Ansible: `git clone https://github.com/ansible/ansible.git` по любому, удобному вам пути.
3. Зайдите в директорию Ansible: `cd ansible`.
4. Создайте виртуальное окружение: `python3 -m venv venv`.
5. Активируйте виртуальное окружение: `. venv/bin/activate`. Дальнейшие действия производятся только в виртуальном окружении.
6. Установите зависимости `pip install -r requirements.txt`.
7. Запустите настройку окружения `. hacking/env-setup`.
8. Если все шаги прошли успешно — выйдите из виртуального окружения `deactivate`.
9. Ваше окружение настроено. Чтобы запустить его, нужно находиться в директории `ansible` и выполнить конструкцию `. venv/bin/activate && . hacking/env-setup`.

## Основная часть

Ваша цель — написать собственный module, который вы можете использовать в своей role через playbook. Всё это должно быть собрано в виде collection и отправлено в ваш репозиторий.

**Шаг 1.** В виртуальном окружении создайте новый `my_own_module.py` файл.

**Шаг 2.** Наполните его содержимым:
Или возьмите это наполнение [из статьи](https://docs.ansible.com/ansible/latest/dev_guide/developing_modules_general.html#creating-a-module).

**Шаг 3.** Заполните файл в соответствии с требованиями Ansible так, чтобы он выполнял основную задачу: module должен создавать текстовый файл на удалённом хосте по пути, определённом в параметре `path`, с содержимым, определённым в параметре `content`.
```python
# Аргументы
    module_args = dict(
        path=dict(type='str', required=True),
        content=dict(type='str', required=False, default = "homework")
    )
# функции, которые создают файл с содержимым и выводят сообщение о создании файла, либо о его существовании.
    if module.check_mode:
        if not os.path.exists(module.params['path']):
          result['changed'] = True
        module.exit_json(**result)

    if not os.path.exists(module.params['path']):
        with open(module.params['path'], 'w') as f:
            f.write(module.params['content'])
        result['changed'] = True
        result['original_message'] = 'File created'
        result['message'] = 'File created'
    else:
        result['original_message'] = 'File is already exists'
        result['message'] = 'File is already exists'

    module.exit_json(**result)
```

**Шаг 4.** Проверьте module на исполняемость локально.  
`python -m ansible.modules.my_own_module payload.json `  
[payload.json ](https://github.com/Svalker1989/my_own_collection/blob/main/my_own_namespace/yandex_cloud_elk/payload.json)  
![](https://github.com/Svalker1989/my_own_collection/blob/main/my_own_namespace/yandex_cloud_elk/1.PNG)  
**Шаг 5.** Напишите single task playbook и используйте module в нём.  
[site.yml](https://github.com/Svalker1989/my_own_collection/blob/main/my_own_namespace/yandex_cloud_elk/site.yml)  
`sudo ansible-playbook ./site.yml`  
**Шаг 6.** Проверьте через playbook на идемпотентность.
![](https://github.com/Svalker1989/my_own_collection/blob/main/my_own_namespace/yandex_cloud_elk/3.PNG)  
**Шаг 7.** Выйдите из виртуального окружения.

**Шаг 8.** Инициализируйте новую collection: `ansible-galaxy collection init my_own_namespace.yandex_cloud_elk`.

**Шаг 9.** В эту collection перенесите свой module в соответствующую директорию.
Путь:   
`/home/str/ansible/08-ansible-06-module/ansible/ansible/my_own_namespace/yandex_cloud_elk/plugins/modules/my_own_module.py`  
**Шаг 10.** Single task playbook преобразуйте в single task role и перенесите в collection. У role должны быть default всех параметров module.

**Шаг 11.** Создайте playbook для использования этой role.
[site.yml](https://github.com/Svalker1989/my_own_collection/blob/main/my_own_namespace/yandex_cloud_elk/roles/my_own_module/site.yml)  
**Шаг 12.** Заполните всю документацию по collection, выложите в свой репозиторий, поставьте тег `1.0.0` на этот коммит.

**Шаг 13.** Создайте .tar.gz этой collection: `ansible-galaxy collection build` в корневой директории collection.  
[my_own_namespace-yandex_cloud_elk-1.0.0.tar.gz](https://github.com/Svalker1989/my_own_collection/blob/main/my_own_namespace/yandex_cloud_elk/single_task_pb/my_own_namespace-yandex_cloud_elk-1.0.0.tar.gz)  
**Шаг 14.** Создайте ещё одну директорию любого наименования, перенесите туда single task playbook и архив c collection.  
`./single_task_pb/`  
**Шаг 15.** Установите collection из локального архива: `ansible-galaxy collection install <archivename>.tar.gz`.  
По умолчанию устаналивает в /home/str/.ansible/collections/ansible_collections.  
Для указания директории для установки флаг -p  
![](https://github.com/Svalker1989/my_own_collection/blob/main/my_own_namespace/yandex_cloud_elk/2.PNG)

**Шаг 16.** Запустите playbook, убедитесь, что он работает.
![](https://github.com/Svalker1989/my_own_collection/blob/main/my_own_namespace/yandex_cloud_elk/4.PNG)
**Шаг 17.** В ответ необходимо прислать ссылки на collection и tar.gz архив, а также скриншоты выполнения пунктов 4, 6, 15 и 16.

## Необязательная часть

1. Реализуйте свой модуль для создания хостов в Yandex Cloud.
2. Модуль может и должен иметь зависимость от `yc`, основной функционал: создание ВМ с нужным сайзингом на основе нужной ОС. Дополнительные модули по созданию кластеров ClickHouse, MySQL и прочего реализовывать не надо, достаточно простейшего создания ВМ.
3. Модуль может формировать динамическое inventory, но эта часть не является обязательной, достаточно, чтобы он делал хосты с указанной спецификацией в YAML.
4. Протестируйте модуль на идемпотентность, исполнимость. При успехе добавьте этот модуль в свою коллекцию.
5. Измените playbook так, чтобы он умел создавать инфраструктуру под inventory, а после устанавливал весь ваш стек Observability на нужные хосты и настраивал его.
6. В итоге ваша коллекция обязательно должна содержать: clickhouse-role (если есть своя), lighthouse-role, vector-role, два модуля: my_own_module и модуль управления Yandex Cloud хостами и playbook, который демонстрирует создание Observability стека.

---

### Как оформить решение задания

Выполненное домашнее задание пришлите в виде ссылки на .md-файл в вашем репозитории.

---
