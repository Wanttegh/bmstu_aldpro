Развёртывание ALDPRO

# Важные замечания



## ❗❗❗ Перед тем, как прописывать команды, прочитайте всё, что написано в этом разделе

iso-образ астры качать только такой - https://dl.astralinux.ru/astra/frozen/1.7_x86-64/1.7.3/iso/1.7.3-03.11.2022_15.53.iso

Сначала пропишите hostname -I, запишите свой ip без маски, а затем пропишите nmcli d show и сделайте скрин.

На виртуалку должно выделяться не менее 8 гб оперативки и 50 гб хранилища. Уровень защищенности - Смоленск. 

Команда для удаления network manager - apt remove network-manager.

ip адрес контроллера домена везде писать в таком виде (пример): 192.168.1.26

Все сетевые значения в гайде прописывайте под свою виртуалку

Основа методички --> https://habr.com/ru/articles/747182/



# Развёртывание



Сохраняем где-нибудь на своей основной ос ip адрес, который получите с помощью следующей команды

```sudo hostname -I```

Теперь прописываем nmcli d show и скриншотим весь вывод, далее нам понадобятся значения полей ip4.address, ip4.dns и ip4.gateway

```nmcli d show```

Теперь удаляем network manager

```
sudo systemctl stop network-manager
sudo systemctl disable network-manager
sudo apt remove network-manager
```

Добавляем настройки в /etc/network/interfaces

```
sudo nano /etc/network/interfaces

auto eth0
iface eth0 inet static
address [значение поля ip4.address]
netmask 255.255.255.0
gateway [зачение поля ip4.dns]
dns-nameservers [значение поля ip4.dns]
dns-search test.org
```

Добавляем настройки в /etc/resolv.conf

```
sudo nano /etc/resolv.conf

nameserver 1.1.1.1
search test.org
```

Указываем имя контроллера

```sudo hostnamectl set-hostname dc1.test.org```

Меняем настройки в /etc/hosts

```
sudo nano /etc/hosts

127.0.0.1     localhost.localdomain localhost
[значение поля ip4.address]  dc1.test.org dc1
127.0.1.1     dc1
```

Подключаем репозитории

```
sudo nano /etc/apt/sources.list

deb https://dl.astralinux.ru/astra/frozen/1.7_x86-64/1.7.3/repository-base 1.7_x86-64 main non-free contrib
deb https://dl.astralinux.ru/astra/frozen/1.7_x86-64/1.7.3/repository-extended 1.7_x86-64 main contrib non-free


sudo nano /etc/apt/sources.list.d/aldpro.list

deb https://dl.astralinux.ru/aldpro/stable/repository-main/ 1.4.1 main
deb https://dl.astralinux.ru/aldpro/stable/repository-extended/ generic main
```

Создаём файл приоритета 

```
sudo nano /etc/apt/preferences.d/aldpro

Package: *
Pin: release n=generic
Pin-Priority: 900
```

Обновляем и перезагружаем

```sudo apt update && sudo apt dist-upgrade -y && sudo reboot```

Устанавливаем портал ALDPro

```sudo DEBIAN_FRONTEND=noninteractive apt-get install -q -y aldpro-mp```

Редактируем /etc/resolv.conf

```
sudo nano /etc/resolv.conf

nameserver 127.0.0.1
search test.org
```

Перезапускаем сеть

```sudo systemctl restart networking```

Продвигаем сервер до роли контроллера домена

```sudo /opt/rbta/aldpro/mp/bin/aldpro-server-install.sh -d test.org -n dc1 -p [ваш пароль] --ip [значение поля ip4.address] --no-reboot```

Теперь установка закончена. Далее перезагружаемся и заходим в учётную запись admin домена test.org с тем паролем, который вы указали в последней команде. 

![окно входа](https://github.com/user-attachments/assets/b4756055-a045-44d6-a3ae-224124fefc3d)
