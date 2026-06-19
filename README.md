Before use role:

```
ansible-galaxy install -r requirements.yml
```

Описание:
--------------

Создание и удаление linux group (/etc/group) (приведение к состоянию указанному в репозитории), без учета их ID (GID) при сравнении

Доступные теги:
--------------

* show -выведет состояние dict linux_groups определенного в репозитории, а так же какие группы должны присутствовать на хосте, а какие будут удалены (после сравнения)
* debug -покажет какие группы должны присутствовать на хосте, а какие будут удалены (если появились новые группы или группы, подлежащие удалению = покажет их в fatal: ... (failed task)
* restrict_group - задействовать только таску удаления групп
* target_group - задействовать только таску добавления групп


Доступные переменные:
--------------
* linux_groups_role_behaviour - переменная позволяющая задать поведение роли linux_groups (рекомендуется указывать в переменных инвентаря для хоста, а не через extra vars):
  * linux_groups_role_behaviour=='manage_defined_only' (Default) == роль будет управлять только теми группами, что указаны в репозитории (dict linux_groups), если на хосте присутствуют другие группы - роль их игнорирует и ничего с ними не делает
  * linux_groups_role_behaviour=='all_state_in_repo' == состояние в репозитории эталонное: если на хосте присутствуют группы, которые не указаны в репозитории (dict linux_groups) - роль их попытается удалить
* target_group - позволяет сфокусироваться на обработке только конктреной группы (не затрагивая другие), допускается указать несколько групп через запятую ','

Пример описания группы для хоста
--------------
#### Общий вид
```
linus_group_role_behaviour: <string>                    # Задание поведения роли, допустимые значения
                                                        # 'manage_defined_only' (Default) ==> роль будет управлять только теми группами,
                                                        #       что указаны в репозитории (dict linux_groups), если на хосте присутствуют другие
                                                        #       группы - роль их игнорирует и ничего с ними не делает
                                                        # 'all_state_in_repo' ==> состояние в репозитории эталонное: если на хосте присутствуют группы,
                                                        #       которые не указаны в репозитории (dict linux_groups) - роль их попытается удалить
linux_groups:
  <group_name>:
    enabled: <True|False> (required)                    # True - группа должна присутствовать на хосте
                                                        # False - группа не должна присутствовать на хосте
    system: <True|False> (optional, default=False)      # работает только при создании групп:
                                                        # Если True, то у создаваемой группы будет ID<1000
                                                        # Если False, то у создаваемой группы будет ID>=1000 
```
#### Частный пример
```
linux_groups_role_behaviour: 'manage_defined_only'
linux_groups:
  admin:
    enabled: True
    system: True
  sysadmin:
    enabled: True
  somegroup:
    enabled: False
```
В данном случае для хоста:
1. группы admin, sysadmin будут созданы если их не существовало (при этом для группы admin будет использован флаг system (gid<1000)
2. группа somegroup, будет удалена (даже если к ней относятся пользователи и даже если это основная группа пользователя)

Пример playbook
--------------
```
---
- hosts: linux
  become: True
  become_method: sudo
  roles:
    - linux_groups
```
Запуск playbook
--------------
```
ansible-playbook -i inventory -l <some_host> playbooks/linux_groups.yml -t debug 
ansible-playbook -i inventory -l <some_host> playbooks/linux_groups.yml -t show
ansible-playbook -i inventory -l <some_host> playbooks/linux_groups.yml -e target_group=<some_group> [-CD]
ansible-playbook -i inventory playbook/linux_groups.yml [-CD]
```
