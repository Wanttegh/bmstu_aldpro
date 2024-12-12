Развёртывание ALDPRO

# Важные замечания



## ❗❗❗ Перед тем, как прописывать команды, прочитайте всё, что написано в этом разделе

iso-образ астры качать только такой - https://dl.astralinux.ru/astra/frozen/1.7_x86-64/1.7.3/iso/1.7.3-03.11.2022_15.53.iso

Сначала пропишите hostname -I, запишите свой ip без маски, а затем пропишите nmcli d show и сделайте скрин.

На виртуалку должно выделяться не менее 8 гб оперативки и 50 гб хранилища. Уровень защищенности - Смоленск. 

Команда для удаления network manager - apt remove network-manager.

ip адрес контроллера домена везде писать в таком виде (пример): 192.168.1.26

Все сетевые значения в гайде прописывайте под свою виртуалку

ip адрес 192.168.1.26 - это ip адрес, сгенерированный на моей виртуалке. Не надо его использовать в своих настройках!!!

Основа методички --> https://habr.com/ru/articles/747182/



# Развёртывание



Сохраняем где-нибудь на своей основной ос ip адрес, который получите с помощью следующей команды

```sudo hostname -I```

Теперь прописываем nmcli d show и скриншотим весь вывод, далее нам понадобятся значения полей ip4.address, ip4.dns и ip4.gateway

```nmcli d show```

![параметры сети](https://github.com/user-attachments/assets/7e2ec21c-5f8d-4726-a302-2ef669e25c3c)


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

![настройки интерфейсов](https://github.com/user-attachments/assets/f9be85ec-8c3b-4893-87d2-03e025a2b272)


Добавляем настройки в /etc/resolv.conf

```
sudo nano /etc/resolv.conf

nameserver 1.1.1.1
search test.org
```

![настройки resolv.conf](https://github.com/user-attachments/assets/82dd55fd-97ac-46c0-bdcb-f875d24ef0c1)


Указываем имя контроллера

```sudo hostnamectl set-hostname dc1.test.org```

Меняем настройки в /etc/hosts

```
sudo nano /etc/hosts

127.0.0.1     localhost.localdomain localhost
[значение поля ip4.address]  dc1.test.org dc1
127.0.1.1     dc1
```

![настройки хостов](https://github.com/user-attachments/assets/32158525-d53e-40cf-8d73-d0cac1dcb178)


Подключаем репозитории

```
sudo nano /etc/apt/sources.list

deb https://dl.astralinux.ru/astra/frozen/1.7_x86-64/1.7.3/repository-base 1.7_x86-64 main non-free contrib
deb https://dl.astralinux.ru/astra/frozen/1.7_x86-64/1.7.3/repository-extended 1.7_x86-64 main contrib non-free


sudo nano /etc/apt/sources.list.d/aldpro.list

deb https://dl.astralinux.ru/aldpro/stable/repository-main/ 1.4.1 main
deb https://dl.astralinux.ru/aldpro/stable/repository-extended/ generic main
```

![файл sources.list](https://github.com/user-attachments/assets/21f53957-2b0c-4a12-b71b-5fc9b4bab82a)

![файл aldpro.list](https://github.com/user-attachments/assets/560665cb-202d-4a38-848e-adea462075c3)

Создаём файл приоритета 

```
sudo nano /etc/apt/preferences.d/aldpro

Package: *
Pin: release n=generic
Pin-Priority: 900
```

![настройки файла приоритета](https://github.com/user-attachments/assets/d3ffed02-399c-47eb-8a37-ee53ae6ae808)


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

![меняем resolv.conf](https://github.com/user-attachments/assets/ca17a480-9197-4358-ac67-32ce34224b49)


Перезапускаем сеть

```sudo systemctl restart networking```

Продвигаем сервер до роли контроллера домена

```sudo /opt/rbta/aldpro/mp/bin/aldpro-server-install.sh -d test.org -n dc1 -p [ваш пароль] --ip [значение поля ip4.address] --no-reboot```

Теперь установка закончена. Далее перезагружаемся и заходим в учётную запись admin домена test.org с тем паролем, который вы указали в последней команде. 

![окно входа](https://github.com/user-attachments/assets/b4756055-a045-44d6-a3ae-224124fefc3d)
