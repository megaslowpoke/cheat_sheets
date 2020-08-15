## Установка Far2L
Используем [far2l](https://github.com/elfmz/far2l), и добавляем вкл\выкл панелей по Esc. Вот [тут](https://github.com/corporateshark/far2l-macros) можно посмотреть как другие макросы добавлять.
```
git clone https://github.com/elfmz/far2l
cd far2l
mkdir build
cd build
```
Если это безгуёвая ОС, то сначала надо добавить:
```
sudo apt install libwxgtk3.0
```
Тогда должно скомпилиться.
```
cmake -DUSEWX=yes -DCMAKE_BUILD_TYPE=Release ..
make -j4
cd install
```
Копируем содержимое например в `~/far2l` и добавляем в PATH:
```
sudo export PATH="$HOME/far2l:$PATH"
```
И добавим линк, чтобы вызывать можно было по `far` из терминала:
```
ln -s far2l far
```
А так можно сделать линк на него для КДЕ:
```
wget https://upload.wikimedia.org/wikipedia/commons/d/d3/Far_icon.png
wdir=$(pwd)
echo "[Desktop Entry]" > far2l.desktop
echo "Name=Far to Linux" >> far2l.desktop
echo "Comment=File manager" >> far2l.desktop
echo "Exec=${wdir}/far2l" >> far2l.desktop
echo "Icon=${wdir}/Far_icon.png" >> far2l.desktop
echo "Terminal=false" >> far2l.desktop
echo "Type=Application" >> far2l.desktop
echo "Categories=Utility;FileManager;System;FileTools;" >> far2l.desktop
echo "Keywords=file manager;" >> far2l.desktop
mkdir -p ~/.config/far2l/REG/HKU/c/k-Software/k-Far2/k-KeyMacros/k-Shell/k-Esc
cd ~/.config/far2l/REG/HKU/c/k-Software/k-Far2/k-KeyMacros/k-Shell/k-Esc
echo "DisableOutput" > v-DisableOutput
echo "DWORD" >> v-DisableOutput
echo "1" >> v-DisableOutput
echo "EmptyCommandLine" > v-EmptyCommandLine
echo "DWORD" >> v-EmptyCommandLine
echo "1" >> v-EmptyCommandLine
echo "Sequence" > v-Sequence
echo "SZ" >> v-Sequence
echo "CtrlO\0" >> v-Sequence
sudo cp -fl -- ${wdir}/far2l.desktop /usr/share/applications/
```
