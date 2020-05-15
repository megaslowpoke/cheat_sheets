### Подготовка системы
Запускать bash вместо sh по дефолту, актуально для контейнера с минимальной установкой:
```
chsh -s /bin/bash
```
обновляем бубунту
```
sudo apt update && sudo apt upgrade
```
Установка: мц, эйчтоп(диспетчер задач), аптитуде, ssh для putty, 
secure-delete (для очистки пустого места на виртуалке), 
landscape (типа сусинфо, при входе юзера например прилепить)
```
sudo apt-get install -y mc htop aptitude ssh secure-delete landscape-common zip gnupg2 curl libcap2-bin virtualbox-guest-dkms
```
> или если в минимальную установку:
> ```
> sudo apt-get install -y mc htop aptitude ssh secure-delete landscape-common zip gnupg2 libnss-winbind curl nano libcap2-bin language-pack-ru
> ```
> Русская locale в Ubuntu Server
> ```
> sudo apt-get install language-pack-ru
> sudo update-locale LANG=ru_RU.UTF-8
> ```
> проверить:
> ```
> locale
> ```
> для сложных случаев:
> ```
> sudo dpkg-reconfigure locales
> ```
> в любом случае требует обязательную перезагрузку.
> При использовании контейнера chroot тут могут слетать права:
> ```
> sudo chown -Rf user:user /home/user/.cache/
> sudo chown -Rf user:user /home/user/.local/
> sudo chown -Rf user:user /home/user/.config/
> ```

#### Убираем сообщение о новом релизе:
```
sudo chmod -x /etc/update-motd.d/91-release-upgrade
```
> Убрать выключение экрана консоли:
> ```
> setterm -blank 0 -powerdown 0 -powersave off
> ```
> Но это работает только в текущей сессии, поэтому так:
> тут указано время гашения в секундах:
> ```
> nano /sys/module/kernel/parameters/consoleblank
> ```
> Но менять бесполезно, работает видимо только в старых версиях,
> чтобы отключить это поведение глобально и навсегда,
> следует добавить строку `consoleblank=0` к параметрам
> ядра в конфиге grub и перезагрузить ОС:
> ```
> sudo nano /etc/default/grub
> ```
> Заодно указываем параметры отключение патчей Spectre и Meltdown:
> ```
> GRUB_CMDLINE_LINUX="consoleblank=0 nopti nospectre_v2 pti=off spectre_v2=off"
> ```
> Обновляем граб и перезапускаемся
> ```
> sudo update-grub
> ```

#### Включаем нумлок по умолчанию
```
sudo nano /etc/rc.local
```
> Если нет доступа к сохранению, то используем `sudo -s` например.

Перед `exit 0` вводим:
```
for tty in /dev/tty[1-6]; do
	/usr/bin/setleds -D +num < $tty
done
```

#### Самба
нужна чтобы имя компа с убунтой было видно вендовым машинам без использования своего днс. 
Если самба ставилась сразу при установке, то это уже не нужно:
> ```
> sudo apt-get install samba
> ```

или достаточно поставить только:
```
sudo apt-get install libnss-winbind
```

#### Можно ещё включить винс и тогда бубунта будет тоже видеть вендовые имена:
```
sudo nano /etc/samba/smb.conf
```
В секции `[global]` раскаментить:
```
wins support = yes
```
А в `sudo nano /etc/nsswitch.conf` добавить в конец wins:
```
hosts:          files dns wins
```
Запустить службу winbind
```
service winbind start
```

#### Очищаем бубунту если надо
```
sudo apt-get check && sudo apt-get dist-upgrade && sudo apt-get autoremove && sudo apt-get autoclean && sudo apt-get clean
```



#### Использование secure-delete,
очистка диска:
```
sudo sfill -fllvz /
sudo sfill -fllvz /home
```
очистка свапа на sda1 (ну или где там этот раздел со свапом):
```
sudo sswap -fllvz /dev/sda1
```
или одной строкой сразу всё:
```
sudo sfill -fllvz / && sudo sfill -fllvz /home && sudo sswap -fllvz /dev/sda1
```



#### для вбокса
```
sudo apt-get install build-essential linux-headers-$(uname -r)
sudo apt-get install virtualbox-guest-utils
```
подключение аддонов ВМ, если не установлены ранее, например так:
```
sudo mount /dev/cdrom /media/cdrom
ls -l /media/cdrom
sudo sh /media/cdrom/VBoxLinuxAdditions.run
```
удалить и установить свежий:
```
sudo sh /media/cdrom/VBoxLinuxAdditions.run uninstall
sudo apt-get install build-essential module-assistant virtualbox-guest-dkms
```


#### [htop вместо логина](https://raymii.org/s/tutorials/Run_software_on_tty1_console_instead_of_login_getty.html)
На Ubuntu 16.04 переведён на systemd. Больше не можете использовать перенаправление вывода, теперь все обрабатывается через systemd. getty@tty1
Сначала создайте папку переопределения для службы:
```
sudo mkdir /etc/systemd/system/getty@tty1.service.d/
```
Отредактируйте файл переопределения:
```
sudo nano /etc/systemd/system/getty@tty1.service.d/override.conf
```
Поместите следующее:
```
[Service]
ExecStart=
ExecStart=-/usr/bin/htop
StandardInput=tty
StandardOutput=tty
```
Теперь перезагрузите файлы устройства и перезапустите службу. htop должен появиться в вашем приглашении:
```
systemctl daemon-reload; systemctl restart getty@tty1.service
```
Или просто сделай reboot.
