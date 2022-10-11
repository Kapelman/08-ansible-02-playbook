# Домашнее задание к занятию "08.02 Работа с Playbook"

## Подготовка к выполнению
1. Создайте свой собственный (или используйте старый) публичный репозиторий на github с произвольным именем.
2. Скачайте [playbook](./playbook/) из репозитория с домашним заданием и перенесите его в свой репозиторий.
3. Подготовьте хосты в соотвтествии с группами из предподготовленного playbook. 
4. Скачайте дистрибутив [java](https://www.oracle.com/java/technologies/javase-jdk11-downloads.html) и положите его в директорию `playbook/files/`. 
Решение:

vagrant@vagrant:~$ sudo docker run --name ubuntu  --expose 22 -it  ubuntu_kaplin:v1
root@be2f50fc7008:/#

vagrant@vagrant:~$ sudo docker run --name ubuntu2  --expose 22 -it  ubuntu_kaplin:v1
root@2bdcf0c42b09:/#
- скачаем дистрибутивы и java на виртуальную машину:
```
PS C:\Users\kapli\homeworks\08-ansible-02-playbook> scp  playbook.7z vagrant@192.168.33.10:/home/vagrant/ansible-lesson0702
vagrant@192.168.33.10's password:
playbook.7z                                                                                                                                                                    100% 1293   184.3KB/s   00:00
PS C:\Users\kapli\homeworks\08-ansible-02-playbook> scp  jdk-11.0.16.1_linux-aarch64_bin.tar.gz vagrant@192.168.33.10:/home/vagrant/ansible-lesson0702
vagrant@192.168.33.10's password:
jdk-11.0.16.1_linux-aarch64_bin.tar.gz                                                                                                                                         100%  157MB  60.0MB/s   00:02
```
- установим ansible lint
```
vagrant@vagrant:~/ansible-lesson0702/playbook$ sudo apt install ansible-lint
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages were automatically installed and are no longer required:
  libfile-basedir-perl libfile-desktopentry-perl libfile-mimeinfo-perl libfontenc1 libfwupdplugin1 libio-stringy-perl libipc-system-simple-perl libnet-dbus-perl libtie-ixhash-perl libu2f-udev
  libx11-protocol-perl libxft2 libxkbfile1 libxml-parser-perl libxml-twig-perl libxml-xpathengine-perl libxmuu1 libxv1 libxxf86dga1 linux-image-5.4.0-91-generic linux-modules-5.4.0-91-generic
  linux-modules-extra-5.4.0-91-generic x11-utils x11-xserver-utils xdg-utils
Use 'sudo apt autoremove' to remove them.
The following additional packages will be installed:
  python3-ruamel.yaml
The following NEW packages will be installed:
  ansible-lint python3-ruamel.yaml
0 upgraded, 2 newly installed, 0 to remove and 99 not upgraded.
Need to get 269 kB of archives.
After this operation, 1,230 kB of additional disk space will be used.
Do you want to continue? [Y/n] Y
Get:1 http://us.archive.ubuntu.com/ubuntu focal/universe amd64 python3-ruamel.yaml amd64 0.15.89-3build1 [228 kB]
Get:2 http://us.archive.ubuntu.com/ubuntu focal/universe amd64 ansible-lint all 4.2.0-1 [40.5 kB]
Fetched 269 kB in 1s (220 kB/s)
Selecting previously unselected package python3-ruamel.yaml.
(Reading database ... 201494 files and directories currently installed.)
Preparing to unpack .../python3-ruamel.yaml_0.15.89-3build1_amd64.deb ...
Unpacking python3-ruamel.yaml (0.15.89-3build1) ...
Selecting previously unselected package ansible-lint.
Preparing to unpack .../ansible-lint_4.2.0-1_all.deb ...
Unpacking ansible-lint (4.2.0-1) ...
Setting up python3-ruamel.yaml (0.15.89-3build1) ...
Setting up ansible-lint (4.2.0-1) ...
Processing triggers for man-db (2.9.1-1) ...
```

## Основная часть
1. Приготовьте свой собственный inventory файл `prod.yml`.
Решение:
```
vagrant@vagrant:~/ansible-lesson0702/playbook/inventory$ cat prod.yml
---
elasticsearch:
  hosts:
    ubuntu2:
      ansible_connection: docker
kibana:
    hosts:
      ubuntu:
        ansible_connection: docker
```

2. Допишите playbook: нужно сделать ещё один play, который устанавливает и настраивает kibana.
Решение:
 - Дописанный playbook ниже:
```
- name: Install Kibana
  hosts: kibana
  tasks:
    - name: Upload .tar.gz Upload Kibana file containing binaries from local storage
      copy:
        src: "kibana-{{ kibana_version }}-linux-x86_64.tar.gz"
        dest: "/tmp/kibana-{{ kibana_version }}-linux-x86_64.tar.gz"
      register: get_kibana
      until: get_kibana is succeeded
      tags: kibana
    - name: Create directrory for Kibana
      file:
        state: directory
        path: "{{ kibana_home }}"
      tags: kibana
    - name: Extract Kibana in the installation directory
      become: true
      unarchive:
        copy: false
        src: "/tmp/kibana-{{ kibana_version }}-linux-x86_64.tar.gz"
        dest: "{{ kibana_home }}"
        extra_opts: [--strip-components=1]
        creates: "{{ kibana_home }}/bin/kibana"
      tags:
        - kibana
    - name: Set environment Kibana
      become: true
      template:
        src: templates/kibana.sh.j2
        dest: /etc/profile.d/kibana.sh
      tags: kibana.sh.j2
```
4. При создании tasks рекомендую использовать модули: `get_url`, `template`, `unarchive`, `file`.
5. Tasks должны: скачать нужной версии дистрибутив, выполнить распаковку в выбранную директорию, сгенерировать конфигурацию с параметрами.
6. Запустите `ansible-lint site.yml` и исправьте ошибки, если они есть.
Решение:
Ошибок не обнаружено:
```
vagrant@vagrant:~/ansible-lesson0702/playbook$ ansible-lint site1.yml
```

7. Попробуйте запустить playbook на этом окружении с флагом `--check`.
Решение:
- проверка завершается с ошибкой при попытке распаковать файл в несуществующую директорию
```
vagrant@vagrant:~/ansible-lesson0702/playbook$ sudo ansible-playbook --check -i /home/vagrant/ansible-lesson0702/playbook/inventory/prod.yml site1.yml

PLAY [Install Java] *********************************************************************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************************************************************
ok: [ubuntu]
ok: [ubuntu2]

TASK [Set facts for Java 11 vars] *******************************************************************************************************************************************************************************
ok: [ubuntu2]
ok: [ubuntu]

TASK [Upload .tar.gz file containing binaries from local storage] ***********************************************************************************************************************************************
changed: [ubuntu2]
changed: [ubuntu]

TASK [Create directrory /opt/jdk] *******************************************************************************************************************************************************************************
changed: [ubuntu]
changed: [ubuntu2]

TASK [Create directrory for Java Home] **************************************************************************************************************************************************************************
changed: [ubuntu2]
changed: [ubuntu]

TASK [Ensure installation dir exists] ***************************************************************************************************************************************************************************
changed: [ubuntu2]
changed: [ubuntu]

TASK [Extract java in the installation directory] ***************************************************************************************************************************************************************
fatal: [ubuntu]: FAILED! => {"changed": false, "msg": "dest '/opt/jdk/11.0.16.1' must be an existing dir"}
fatal: [ubuntu2]: FAILED! => {"changed": false, "msg": "dest '/opt/jdk/11.0.16.1' must be an existing dir"}

PLAY RECAP ******************************************************************************************************************************************************************************************************
ubuntu                     : ok=6    changed=4    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0
ubuntu2                    : ok=6    changed=4    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0
```

8. Запустите playbook на `prod.yml` окружении с флагом `--diff`. Убедитесь, что изменения на системе произведены.
Решение:
- запустим команду
```
vagrant@vagrant:~/ansible-lesson0702/playbook$ sudo ansible-playbook --diff -i /home/vagrant/ansible-lesson0702/playbook/inventory/prod.yml site1.yml

PLAY [Install Java] *********************************************************************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************************************************************
ok: [ubuntu]
ok: [ubuntu2]

TASK [Set facts for Java 11 vars] *******************************************************************************************************************************************************************************
ok: [ubuntu2]
ok: [ubuntu]

TASK [Upload .tar.gz file containing binaries from local storage] ***********************************************************************************************************************************************
diff skipped: source file size is greater than 104448
changed: [ubuntu2]
diff skipped: source file size is greater than 104448
changed: [ubuntu]

TASK [Create directrory /opt/jdk] *******************************************************************************************************************************************************************************
--- before
+++ after
@@ -1,4 +1,4 @@
 {
     "path": "/opt/jdk",
-    "state": "absent"
+    "state": "directory"
 }

changed: [ubuntu]
--- before
+++ after
@@ -1,4 +1,4 @@
 {
     "path": "/opt/jdk",
-    "state": "absent"
+    "state": "directory"
 }

changed: [ubuntu2]

TASK [Create directrory for Java Home] **************************************************************************************************************************************************************************
--- before
+++ after
@@ -1,4 +1,4 @@
 {
     "path": "/opt/jdk/11.0.16.1",
-    "state": "absent"
+    "state": "directory"
 }

changed: [ubuntu2]
--- before
+++ after
@@ -1,4 +1,4 @@
 {
     "path": "/opt/jdk/11.0.16.1",
-    "state": "absent"
+    "state": "directory"
 }

changed: [ubuntu]

TASK [Ensure installation dir exists] ***************************************************************************************************************************************************************************
ok: [ubuntu]
ok: [ubuntu2]

TASK [Extract java in the installation directory] ***************************************************************************************************************************************************************
changed: [ubuntu]
changed: [ubuntu2]

TASK [Export environment variables] *****************************************************************************************************************************************************************************
--- before
+++ after: /root/.ansible/tmp/ansible-local-124239xygx80t/tmp1qxryj3x/jdk.sh.j2
@@ -0,0 +1,5 @@
+# Warning: This file is Ansible Managed, manual changes will be overwritten on next playbook run.
+#!/usr/bin/env bash
+
+export JAVA_HOME=/opt/jdk/11.0.16.1
+export PATH=$PATH:$JAVA_HOME/bin
\ No newline at end of file

changed: [ubuntu2]
--- before
+++ after: /root/.ansible/tmp/ansible-local-124239xygx80t/tmpm0yb_k_w/jdk.sh.j2
@@ -0,0 +1,5 @@
+# Warning: This file is Ansible Managed, manual changes will be overwritten on next playbook run.
+#!/usr/bin/env bash
+
+export JAVA_HOME=/opt/jdk/11.0.16.1
+export PATH=$PATH:$JAVA_HOME/bin
\ No newline at end of file

changed: [ubuntu]

PLAY [Install Elasticsearch] ************************************************************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************************************************************
ok: [ubuntu2]

TASK [Upload .tar.gz Elastic file containing binaries from local storage] ***************************************************************************************************************************************
diff skipped: source file size is greater than 104448
changed: [ubuntu2]

TASK [Create directrory for Elasticsearch] **********************************************************************************************************************************************************************
--- before
+++ after
@@ -1,4 +1,4 @@
 {
     "path": "/opt/elastic/8.4.3",
-    "state": "absent"
+    "state": "directory"
 }

changed: [ubuntu2]

TASK [Extract Elasticsearch in the installation directory] ******************************************************************************************************************************************************
changed: [ubuntu2]

TASK [Set environment Elastic] **********************************************************************************************************************************************************************************
--- before
+++ after: /root/.ansible/tmp/ansible-local-124239xygx80t/tmpfi_pccat/elk.sh.j2
@@ -0,0 +1,5 @@
+# Warning: This file is Ansible Managed, manual changes will be overwritten on next playbook run.
+#!/usr/bin/env bash
+
+export ES_HOME=/opt/elastic/8.4.3
+export PATH=$PATH:$ES_HOME/bin
\ No newline at end of file

changed: [ubuntu2]

PLAY [Install Kibana] *******************************************************************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************************************************************
ok: [ubuntu]

TASK [Upload .tar.gz Upload Kibana file containing binaries from local storage] *********************************************************************************************************************************
diff skipped: source file size is greater than 104448
changed: [ubuntu]

TASK [Create directrory for Kibana] *****************************************************************************************************************************************************************************
--- before
+++ after
@@ -1,4 +1,4 @@
 {
     "path": "/opt/kibana/8.4.3",
-    "state": "absent"
+    "state": "directory"
 }

changed: [ubuntu]

TASK [Extract Kibana in the installation directory] *************************************************************************************************************************************************************
changed: [ubuntu]

TASK [Set environment Kibana] ***********************************************************************************************************************************************************************************
--- before
+++ after: /root/.ansible/tmp/ansible-local-270577bgfc4qu/tmppo_3leo3/kibana.sh.j2
@@ -0,0 +1,6 @@
+# Warning: This file is Ansible Managed, manual changes will be overwritten on next playbook run.
+#!/usr/bin/env bash
+
+export KIBANA_HOME=/opt/kibana/8.4.3
+export KBN_PATH_CONF=$KIBANA_HOME/config
+export PATH=$PATH:$KIBANA_HOME/bin

changed: [ubuntu]

PLAY RECAP ******************************************************************************************************************************************************************************************************
ubuntu                     : ok=13   changed=9    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
ubuntu2                    : ok=13   changed=9    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

10. Повторно запустите playbook с флагом `--diff` и убедитесь, что playbook идемпотентен.
Решение:
 - Запустим команду повторно
```
vagrant@vagrant:~/ansible-lesson0702/playbook$ sudo ansible-playbook --diff -i /home/vagrant/ansible-lesson0702/playbook/inventory/prod.yml site1.yml

PLAY [Install Java] *********************************************************************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************************************************************
ok: [ubuntu2]
ok: [ubuntu]

TASK [Set facts for Java 11 vars] *******************************************************************************************************************************************************************************
ok: [ubuntu2]
ok: [ubuntu]

TASK [Upload .tar.gz file containing binaries from local storage] ***********************************************************************************************************************************************
ok: [ubuntu2]
ok: [ubuntu]

TASK [Create directrory /opt/jdk] *******************************************************************************************************************************************************************************
ok: [ubuntu2]
ok: [ubuntu]

TASK [Create directrory for Java Home] **************************************************************************************************************************************************************************
ok: [ubuntu2]
ok: [ubuntu]

TASK [Ensure installation dir exists] ***************************************************************************************************************************************************************************
ok: [ubuntu2]
ok: [ubuntu]

TASK [Extract java in the installation directory] ***************************************************************************************************************************************************************
skipping: [ubuntu]
skipping: [ubuntu2]

TASK [Export environment variables] *****************************************************************************************************************************************************************************
ok: [ubuntu2]
ok: [ubuntu]

PLAY [Install Elasticsearch] ************************************************************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************************************************************
ok: [ubuntu2]

TASK [Upload .tar.gz Elastic file containing binaries from local storage] ***************************************************************************************************************************************
ok: [ubuntu2]

TASK [Create directrory for Elasticsearch] **********************************************************************************************************************************************************************
ok: [ubuntu2]

TASK [Extract Elasticsearch in the installation directory] ******************************************************************************************************************************************************
skipping: [ubuntu2]

TASK [Set environment Elastic] **********************************************************************************************************************************************************************************
ok: [ubuntu2]

PLAY [Install Kibana] *******************************************************************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************************************************************
ok: [ubuntu]

TASK [Upload .tar.gz Upload Kibana file containing binaries from local storage] *********************************************************************************************************************************
ok: [ubuntu]

TASK [Create directrory for Kibana] *****************************************************************************************************************************************************************************
ok: [ubuntu]

TASK [Extract Kibana in the installation directory] *************************************************************************************************************************************************************
skipping: [ubuntu]

TASK [Set environment Kibana] ***********************************************************************************************************************************************************************************
ok: [ubuntu]

PLAY RECAP ******************************************************************************************************************************************************************************************************
ubuntu                     : ok=11   changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
ubuntu2                    : ok=11   changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
```

11. Подготовьте README.md файл по своему playbook. В нём должно быть описано: что делает playbook, какие у него есть параметры и теги.
Решение:
- Playbook, теги - kibana и kibana.sh.j2, переменные kibana_version и kibana_home. Каждый шаг описан в названии. Бинарный файл архив загружает на managed hosts
```
- name: Install Kibana
  hosts: kibana
  tasks:
    - name: Upload .tar.gz Upload Kibana file containing binaries from local storage
      copy:
        src: "kibana-{{ kibana_version }}-linux-x86_64.tar.gz"
        dest: "/tmp/kibana-{{ kibana_version }}-linux-x86_64.tar.gz"
      register: get_kibana
      until: get_kibana is succeeded
      tags: kibana
    - name: Create directrory for Kibana
      file:
        state: directory
        path: "{{ kibana_home }}"
      tags: kibana
    - name: Extract Kibana in the installation directory
      become: true
      unarchive:
        copy: false
        src: "/tmp/kibana-{{ kibana_version }}-linux-x86_64.tar.gz"
        dest: "{{ kibana_home }}"
        extra_opts: [--strip-components=1]
        creates: "{{ kibana_home }}/bin/kibana"
      tags:
        - kibana
    - name: Set environment Kibana
      become: true
      template:
        src: templates/kibana.sh.j2
        dest: /etc/profile.d/kibana.sh
      tags: kibana.sh.j2
```
- файл с переменными
```
---
kibana_version: "8.4.3"
kibana_home: "/opt/kibana/{{ kibana_version }}"
```
- файл inventory
```
---
elasticsearch:
  hosts:
    ubuntu2:
      ansible_connection: docker
kibana:
    hosts:
      ubuntu:
        ansible_connection: docker
```
- файл template 
```
# Warning: This file is Ansible Managed, manual changes will be overwritten on next playbook run.
#!/usr/bin/env bash

export KIBANA_HOME={{ kibana_home }}
export KBN_PATH_CONF=$KIBANA_HOME/config
export PATH=$PATH:$KIBANA_HOME/bin
```

13. Готовый playbook выложите в свой репозиторий, в ответ предоставьте ссылку на него.

## Необязательная часть

1. Приготовьте дополнительный хост для установки logstash.
2. Пропишите данный хост в `prod.yml` в новую группу `logstash`.
3. Дополните playbook ещё одним play, который будет исполнять установку logstash только на выделенный для него хост.
4. Все переменные для нового play определите в отдельный файл `group_vars/logstash/vars.yml`.
5. Logstash конфиг должен конфигурироваться в части ссылки на elasticsearch (можно взять, например его IP из facts или определить через vars).
6. Дополните README.md, протестируйте playbook, выложите новую версию в github. В ответ предоставьте ссылку на репозиторий.

---

### Как оформить ДЗ?

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
