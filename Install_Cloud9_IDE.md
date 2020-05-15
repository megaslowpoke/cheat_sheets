### Установка Cloud9
> Cтарый ноджиэс и нпм
> ```
> sudo apt install nodejs-legacy npm
> ```
> Устанавливаем ноджыэс [посвежее](https://github.com/nodesource/distributions/blob/master/README.md), в конце пробел тире.

На теле в x32 будет ставиться только 9, старше только для x64
```
curl -sL https://deb.nodesource.com/setup_9.x | sudo -E bash -
```
> ```
> curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash -
> curl -sL https://deb.nodesource.com/setup_11.x | sudo -E bash -
> curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
> curl -sL https://deb.nodesource.com/setup_13.x | sudo -E bash -
> ```

Проверить что будет устанавливаться:
```
apt-cache policy nodejs
```
Накатываем:
```
sudo apt install -y nodejs
```
Посмотреть что установлено `nodejs --version` или `node -v`

Разрешаем запуск ноджыэса на 80 порту без судо, `/usr/bin/node` или `/usr/bin/nodejs` это путь где установлен ноджыэс
```
sudo setcap cap_net_bind_service=+ep /usr/bin/node
sudo setcap cap_net_bind_service=+ep /usr/bin/nodejs
```
Для запуска как сервис устанавливаем:
```
sudo apt install npm
sudo npm install pm2 -g
```

#### Для c9
```
sudo apt install git libssl-dev sqlite3 build-essential python-minimal tmux
```

> На теле в контейнере могут слетать права тут:
> ```
> sudo chown -Rf user:user /home/user/.cache/
> sudo chown -Rf user:user /home/user/.local/
> sudo chown -Rf user:user /home/user/.config/
> ```


```
mkdir c9sdk c9project && cd c9sdk
git clone git://github.com/c9/core.git core
./core/scripts/install-sdk.sh
```
Для проверки можно запустить:
```
node /home/user/c9sdk/core/server.js -p 80 -l "0.0.0.0" -a : -w /home/user/c9project
nodejs /home/user/c9sdk/core/server.js -p 80 -l "0.0.0.0" -a : -w /home/user/c9project
```
### pm2
Создаём файл `nano c9run.yml` и заполняем настройками (две копии на соседних портах):
```
apps:
 - name  : "c9-80"
   script: "/home/user/c9sdk/core/server.js"
   args  : "--listen 0.0.0.0 --port 80 --auth : -w /home/user/c9project"
 - name  : "c9-81"
   script: "/home/user/c9sdk/core/server.js"
   args  : "--listen 0.0.0.0 --port 81 --auth : -w /home/user"
```

pm2 позволяет запомнить запущенные сейчас и после перезагрузки запустить их же заново.
Запуск:
```
pm2 start /home/user/c9sdk/c9run.yml --restart-delay 10000
```
Готово, это наша строка для [запуска](http://pm2.keymetrics.io/docs/usage/startup/)
> ```
> pm2 status
> pm2 monit
> pm2 del 'id'
> pm2 stop 'id'
> ```

Чтобы запомнить текущее состояние и после перезагрузки всё запускалось заново:
```
pm2 startup
```
В строке вывода будет команда для настроки автозапуска в этой системе, соответственно делаем так:
```
pm2 startup > pm2startup.sh
nano pm2startup.sh
```
Удаляем всё лишнее, оставляя только строку запуска:
```
sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u user --hp /home/user
```
> В wsl переменные уже не надо, оно там неправильно реагирует, достаточно оставить:
> ```
> sudo pm2 startup systemd -u user --hp /home/user
> ```

И запускаем:
```
sh pm2startup.sh
```

> Отключить автозапуск `pm2 unstartup`, или `pm2 unstartup systemd`
> И тоже самое для полного отключения:
> ```
> Pm2 unstartup > pm2unstartup.sh
> nano pm2unstartup.sh
> sh pm2unstartup.sh
> ```
> Чтобы сохранить список процессов для автозапуска:
> ```
> pm2 save
> ```
> Вручную перезапустить
> ```
> pm2 resurrect
> ```
> 
> Обновить автозапуск
> ```
> sh pm2unstartup.sh
> sh pm2startup.sh
> ```
> 

Для установки в контейнере chroot на теле, открываем:
```
sudo nano /etc/rc.local
```
добавляем перед `exit 0`
```
pm2 start /home/user/c9sdk/c9run.yml --restart-delay 10000
```
> после восстановления образа могут слетать некоторые права, поэтому можно например туда же добавить в начале заново установку разрешений ноде:
> ```
> sudo setcap cap_net_bind_service=+ep /usr/bin/node
> ```

> В Linux Deploy указываем:
> * _Разрешить запуск пользовательских сценариев_
> * _Система инициализации "run-parts"_
> * __ОБЯЗАТЕЛЬНО: Параметы инициализации: Пользователь: user, иначе иде будет от рута запускаться__


### Ставим [rvm для руби и гемов](https://rvm.io/):
```
gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB
curl -sSL https://get.rvm.io | bash -s stable
source /home/user/.rvm/scripts/rvm
```
Если ошибка, значит мы не в баше, запускаем баш сначала: `bash`, ставим последний руби через rvm:
```
rvm install ruby-2.7.0
```
Если надо задаём область для гемов, называем её `[_номер_руби_]@[_common_]`
```
rvm gemset create common
rvm ruby27@common
```
Но вообще уже есть `global`, отключаем генерацию оффлайн докментации и ставим гемы:
```
echo 'gem: --no-document' >> ~/.gemrc
gem install faraday faraday_middleware eventmachine faye-websocket sqlite3 socksify sinatra sinatra-reloader sinatra-activerecord activerecord tux rails bundle shell e2mmap sync
```

### Очищаем если надо:
```
sudo apt-get check && sudo apt-get dist-upgrade && sudo apt-get autoremove && sudo apt-get autoclean && sudo apt-get clean
```

### Добавляем руннер для синатры `Sinatra.run`:
```
mkdir ~/c9project/.c9 && mkdir ~/c9project/.c9/runners
nano ~/c9project/.c9/runners/Sinatra.run
```
С таким содержимым:
```
// Create a custom Cloud9 runner - similar to the Sublime build system
// For more information see https://docs.c9.io/custom_runners.html
{
    "cmd" : ["bash", "--login", "-c", "ruby $file -p $port -o $ip $args"],
    // "info" : "Sinatra started $project_path$file_name in http://$ip:$port",
    "info" : "Sinatra started $file_path/$file_name in http://$ip:$port",
    "env" : {},
    // "working_dir" : "$project_path",
    "working_dir" : "$file_path",
    "selector" : "source.rb"
}
```

#### Тут в контейнере могут слетать права:
```
sudo chown -Rf user:user /home/user/.c9/
sudo chown -Rf user:user /home/user/c9project/.c9/
sudo chown -Rf user:user /home/user/.local/
sudo chown -Rf user:user /home/user/.config/
```
