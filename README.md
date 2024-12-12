Развёртывание ALDPRO

# Важные замечания



## ❗❗❗ Перед тем, как прописывать команды, прочитайте всё, что написано в этом разделе

iso-образ астры качать только такой - https://dl.astralinux.ru/astra/frozen/1.7_x86-64/1.7.3/iso/1.7.3-03.11.2022_15.53.iso

На виртуалку должно выделяться не менее 8 гб оперативки и 50 гб хранилища. Уровень защищенности - Смоленск. 

ip адрес 192.168.1.26 - это ip адрес, сгенерированный на моей виртуалке, точно так же как gateway и dns 192.168.1.1 . Не надо их использовать в своих настройках, прописывайте свои значения!!!

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

# Настройка групповых политик и паролей

Авторизовавшись в учётке admin, заходим в браузер и заходим в панель управления

```https://dc1/ad/ui/#/```

![Панель админа](https://github.com/user-attachments/assets/17777a81-4ed3-4a65-ad35-f8c2ee06f0a3)

Теперь создадим новую группу

![изображение](https://github.com/user-attachments/assets/eaf8ee16-713c-4187-8afa-7a638ea3406a)

Нажимаем "+ Новая группа"

![изображение](https://github.com/user-attachments/assets/9e3a443a-97fc-4ecd-b127-11f7212259ba)

Сейчас нам доступно только три поля ввода. Остальные станут доступны после применения первичных настроек.

![изображение](https://github.com/user-attachments/assets/55bc28d0-37f9-4faa-9976-2b8fd62fe307)

Сохраняем настройки, после чего переходим в меню "Пользователи" и добавляем наших пользователей

![изображение](https://github.com/user-attachments/assets/28ad5fdf-c947-451c-a08b-4d382b8d111a)

Далее создадим нового пользователя

![изображение](https://github.com/user-attachments/assets/a95ab92e-13be-4cf8-a0f2-45134b72ca04)

Перед нами откроется панель, имеющая функционал по добавлению пользователей. Жмём "+ Новый пользователь"

![изображение](https://github.com/user-attachments/assets/e4944701-933d-4af6-a3f4-c2039a65b845)

Теперь мы можем настроить нового пользователя как нам угодно. Пароль пока что прописываем какой хотим, после применения настроек групповой политики и перезагрузке машины он будет принудительно сброшен.

![изображение](https://github.com/user-attachments/assets/1b8c28b4-93a8-4b11-91bc-1600c169a218)

После этого жмём сохранить

![изображение](https://github.com/user-attachments/assets/85603c6a-6186-4994-89a0-32a0987611ab)

Теперь можно перемещаться по другим аспектам настройки пользователя, в том числе, что самое важное - присвоить ему группу.

![изображение](https://github.com/user-attachments/assets/86ef1d2a-9f49-4453-b49b-cba661234b48)

Теперь настроем групповые политики. Для этого перейдём в раздел Групповые политики --> Политики паролей

![изображение](https://github.com/user-attachments/assets/b7eca785-ad95-4224-807b-469b0b5c8ef5)

Жмём "+ Новая политика паролей", выбираем группу и задаём приоритет, равный 1 (в моём случае 2, потому что до этого я уже добавил политику паролей)

![изображение](https://github.com/user-attachments/assets/6eec9958-60ae-425b-bd88-3ac90548df70)

Теперь настраиваем следующие параметры: максимальный срок действия в днях, минимальный срок действия в часах, размер журнала, классы символов, минимальная длина, максимальное количество ошибок, интервал сбоса ошибок в секундах и длительность блокировки в секундах

![изображение](https://github.com/user-attachments/assets/da8be806-c6e2-47a3-bfcd-07cb0f957aaa)

![изображение](https://github.com/user-attachments/assets/3543aa71-0dd7-412a-b0e2-490a252362c5)

Теперь подробнее о классах символов. Всего их 5. Класс пароля присваивается в зависимости от того, какое количество символов разных групп в нём есть. Сами группы следующие: строчные буквы, заглавные буквы, цифры, спецсимволы и оставшиеся символы кодировки UTF-8. Если какие-то символы будут повторяться, то класс пароля будет понижен.

Теперь перезагружаемся. После ввода пароля на нашем новом пользователе нам будет выдано предупреждение о том, что текущий пароль устарел. Задаём новый пароль в соответствии с политикой паролей.

![изображение](https://github.com/user-attachments/assets/439986a9-2f05-49ba-9593-3343a4e80bba)

Мы получили конечный результат. Добавлен новый пользователь с логином user2, у которого есть строго заданная политика паролей mr_drozdov. 

![изображение](https://github.com/user-attachments/assets/8c2a7d23-4c43-45f1-a94e-ecece75f762f)
