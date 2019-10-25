# Подключение к узлу по SSH

Чтобы подключиться к узлу по SSH необходимо добавить публичный ключ в метаинформацию при создании группы узлов.

{% note info %}

В образах Linux, которые используются на узлах, возможность подключения по протоколу SSH с использованием логина и пароля по умолчанию отключена.

{% endnote %}

Подробнее о подключении по SSH читайте в разделе [Подключение к ВМ по SSH](../../compute/operations/vm-connect/ssh.md).

## Создайте пары ключей SSH {#creating-ssh-keys}

Подготовьте ключи для использования с вашим узлом. Для этого:

{% list tabs %}

- Linux/MacOS
  
  1. Откройте терминал.
  1. Создайте новый ключ с помощью команды `ssh-keygen`:
  
     ```
     $ ssh-keygen -t rsa -b 2048
     ```
  
     После выполнения команды вам будет предложено указать имена файлов, в которые будут сохранены ключи и ввести пароль для закрытого ключа. По умолчанию используется имя `id_rsa`, ключи создаются в директории `~./ssh`.
  
     Публичная часть ключа будет сохранена в файле с названием `<имя_ключа>.pub`.
  
- Windows 10
  
  1. Запустите `cmd.exe` или `powershell.exe`.
  1. Создайте новый ключ с помощью команды `ssh-keygen`. Выполните команду:
  
     ```
     $ ssh-keygen -t rsa -b 2048
     ```
  
     После выполнения команды вам будет предложено указать имя файлов, в которые будут сохранены ключи и ввести пароль для закрытого ключа. По умолчанию используется имя `id_rsa`, ключи создаются в директории `C:\Users\<имя_пользователя>\.ssh\`.
  
     Публичная часть ключа будет сохранена в файле с названием `<имя_ключа>.pub`.
  
- Windows 7/8
  
  Создание ключей для Windows будет выполняться с помощью приложения PuTTY.
  
  1. [Скачайте](https://www.putty.org) и установите PuTTY.
  1. Убедитесь, что директория, куда вы установили PuTTY, присутствует в `PATH`:
     1. Нажмите правой кнопкой на **Мой компьютер**. Выберите пункт **Свойства**.
     1. В открывшемся окне выберите **Дополнительные параметры системы**, затем **Переменные среды** (находится в нижней части окна).
     1. В разделе **Системные переменные** найдите `PATH` и нажмите **Изменить**.
     1. В поле **Значение переменной** допишите путь к директории, куда вы установили PuTTY.
  1. Запустите приложение PuTTYgen.
  1. В качестве типа генерируемой пары выберите **RSA** и укажите длину `2048`. Нажмите **Generate** и поводите курсором в поле выше до тех пор, пока не закончится создание ключа.
  
     ![ssh_generate_key](../../compute/_assets/ssh-putty/ssh_generate_key.png)
  
  1. В поле **Key passphrase** введите надежный пароль. Повторно введите его в поле ниже.
  1. Нажмите кнопку **Save private key** и сохраните закрытый ключ. Никогда его никому не передавайте и не говорите никому ключевую фразу от него.
  1. Сохраните ключ в текстовом файле одной строкой. Для этого скопируйте открытый ключ из текстового поля в текстовый файл с названием `id_rsa.pub`.
  
{% endlist %}

## Приведите публичный ключ к необходимому формату {#key-format}

Управление пользователями и SSH ключами осуществляется с помощью [OS Login](https://cloud.google.com/compute/docs/oslogin/), поэтому ключи необходимо передавать в определенном формате.

Файл с публичным ключом создается в таком виде:

```
ssh-rsa AAAAB3NzaC*********** rsa-key-20190412
```

Необходимо привести ключ к формату `<имя пользователя>:ssh-rsa <тело ключа> <имя пользователя>`, чтобы он выглядел следующим образом:

```
username:ssh-rsa AAAAB3NzaC***********zo/lP1ww== username
```

В одном файле можно передать несколько публичных ключей для доступа разных пользователей:

```
username:ssh-rsa AAAAB3NzaC***********zo/lP1ww== username
username2:ssh-rsa ONEMOREkey***********88OavEHw== username2
```

## Создайте группу узлов и добавьте публичный ключ {#node-create}

Чтобы создать группу узлов с необходимыми параметрами, воспользуйтесь следующей командой:

```
$ yc managed-kubernetes node-group create \
--name <имя группы узлов> \
--cluster-name <имя кластера Kubernetes> \
--fixed-size <кол-во узлов в группе> \
--location zone=<зона доступности>,subnet-name=<имя подсети> \
--public-ip \
--metadata-from-file=ssh-keys=<имя файла с публичными ключами> \
```


## Получите публичный IP-адрес узла {#node-public-ip}

Для подключения необходимо указать [публичный IP-адрес](../../vpc/concepts/address.md#publichnye-adresa) узла. Его можно узнать одним из следующих способов.

{% list tabs %}

- kubectl CLI
  
  Воспользуйтесь следующей командой для kubectl. Публичный IP-адрес указан в столбце `EXTERNAL-IP`.
  
  ```
  $ kubectl get nodes -o wide
  NAME                        STATUS   ROLES    AGE   VERSION   INTERNAL-IP   EXTERNAL-IP      OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
  cl17i6943n92sb98jifg-itif   Ready    <none>   31m   v1.13.3   10.0.0.27     84.201.145.251   Ubuntu 18.04.1 LTS   4.15.0-29-generic   docker://18.6.2
  cl17i6943n92sb98jifg-ovah   Ready    <none>   31m   v1.13.3   10.0.0.22     84.201.149.184   Ubuntu 18.04.1 LTS   4.15.0-29-generic   docker://18.6.2
  ```
  
- Консоль управления
  
  1. Откройте раздел **Compute Cloud** в каталоге, где создан ваш кластер Kubernetes.
  1. На странице **Виртуальные машины** перейдите на вкладку **Группы виртуальных машин**.
  1. Нажмите на группу виртуальных машин, имя которой соответствует идентификатору группы узлов.
  1. В открышемся окне перейдите на вкладку **Список ВМ**.
  1. Нажмите на виртуальную машину, публичный адрес которой хотите узнать.
  1. Публичный IP-адрес указан в блоке **Сеть** в строке **Публичный IPv4**.
  
- YC CLI
  
  1. Узнайте идентификатор группы виртуальных машин, которая соответствует группе узлов.
  
      Необходимый параметр находится в столбце `INSTANCE GROUP ID`.
  
      ```
      $ yc managed-kubernetes node-group list
      +----------------------+----------------------+----------------+----------------------+---------------------+---------+------+
      |          ID          |      CLUSTER ID      |      NAME      |  INSTANCE GROUP ID   |     CREATED AT      | STATUS  | SIZE |
      +----------------------+----------------------+----------------+----------------------+---------------------+---------+------+
      | cat684ojo3irchtpeg84 | cata9ertn6tcr09bh9rm | test-nodegroup | cl17i6943n92sb98jifg | 2019-04-12 12:38:35 | RUNNING |    2 |
      +----------------------+----------------------+----------------+----------------------+---------------------+---------+------+
      ```
  
  1. Посмотрите список узлов, которые принадлежат данной группе.
  
      Публичный IP-адрес узла указан в столбце `IP` после символа `~`.
  
      ```
      $ yc compute instance-group list-instances cl17i6943n92sb98jifg
      +----------------------+---------------------------+--------------------------+---------------+----------------+
      |     INSTANCE ID      |           NAME            |            IP            |    STATUS     | STATUS MESSAGE |
      +----------------------+---------------------------+--------------------------+---------------+----------------+
      | ef31h24k03pg0mhunfm1 | cl17i6943n92sb98jifg-itif | 10.0.0.27~84.201.145.251 | RUNNING [53m] |                |
      | ef37ddhg9i7jhs7tc3pe | cl17i6943n92sb98jifg-ovah | 10.0.0.22~84.201.149.184 | RUNNING [53m] |                |
      +----------------------+---------------------------+--------------------------+---------------+----------------+
      ```
  
{% endlist %}

## Подключитесь к узлу {#node-connect}

Вы можете подключиться к узлу по протоколу SSH, когда он будет запущен (в статусе `RUNNING`). Для этого можно использовать утилиту `ssh` в Linux и macOS и программу [PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/) для Windows.

{% list tabs %}

- Linux/macOS/Windows 10
  
  В терминале выполните следующую команду:
  
  ```
  $ ssh <имя пользователя>@<публичный IP-адрес узла>
  ```
  
  Если вы подключаетесь к узлу в первый раз, может появиться предупреждение о неизвестном хосте:
  
  ```
  The authenticity of host '130.193.40.101 (130.193.40.101)' can't be established.
  ECDSA key fingerprint is SHA256:PoaSwqxRc8g6iOXtiH7ayGHpSN0MXwUfWHkGgpLELJ8.
  Are you sure you want to continue connecting (yes/no)?
  ```
  
  Введите в терминале слово `yes` и нажмите `Enter`.
  
- Windows 7/8
  
  В Windows соединение устанавливается с помощью приложения PuTTY.
  
  1. Запустите приложение Pageant.
     1. Нажмите правой кнопкой мыши на значок Pageant на панели задач.
     1. В контекстном меню выберите пункт **Add key**.
     1. Выберите сгенерированный PuTTY приватный ключ в формате `.ppk`. Если для ключа задан пароль, введите его.
  1. Запустите приложение PuTTY.
     1. В поле **Host Name (or IP address)** введите публичный IP-адрес виртуальной машины, к которой вы хотите подключиться. Укажите порт `22` и тип соединения **SSH**.
  
        ![ssh_add_ip](../../compute/_assets/ssh-putty/ssh_add_ip.png)
  
     1. Откройте в дереве слева пункт **Connection** - **SSH** - **Auth**.
     1. Установите флаг **Allow agent forwarding**.
     1. В поле **Private key file for authentication** выберите файл с приватным ключом.
  
        ![ssh_choose_private_key](../../compute/_assets/ssh-putty/ssh_choose_private_key.png)
  
     1. Вернитесь в меню **Sessions**. В поле **Saved sessions** введите любое название для сессии и нажмите кнопку **Save**. Настройки сессии сохранятся под указанным именем. Вы сможете использовать этот профиль сессии для подключения с помощью Pageant.
  
        ![ssh_save_session](../../compute/_assets/ssh-putty/ssh_save_session.png)
  
     1. Нажмите кнопку **Open**. Если вы подключаетесь к узлу в первый раз, может появиться предупреждение о неизвестном хосте:
  
        ![ssh_unknown_host_warning](../../compute/_assets/ssh-putty/ssh_unknown_host_warning.png)
  
        Нажмите кнопку **Да**. Откроется окно терминала с предложением ввести логин пользователя, от имени которого устанавливается соединение. Введите имя пользователя, которое вы указали в файле с публичным ключом и нажмите `Enter`. Если все настроено верно, будет установлено соединение с сервером.
  
        ![ssh_login](../../compute/_assets/ssh-putty/ssh_login.png)
  
  Если вы сохранили профиль сессии в PuTTY, в дальнейшем для установки соединения можно использовать Pageant:
  
  1. Нажмите правой кнопкой мыши на значок pageant на панели задач.
  1. Выберите пункт меню **Saved sessions**.
  1. В списке сохраненных сессий выберите нужную сессию.
  
{% endlist %}