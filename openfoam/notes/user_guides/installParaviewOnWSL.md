## install requirement packages

`sudo apt install qt5-qmake libegl1-mesa-dev qtbase5-dev qtdeclarative5-dev qt5-default libqt5x11extras5-dev qttools5-dev `

`sudo apt-get install apt-file `

`sudo apt-file update`

`./makeParaView `

- use apt-file to get the lost packages,which is requirement by cmake, such as
  `apt-file search Qt5HelpConfig.cmake`

-references: https://zhuanlan.zhihu.com/p/82423956

## 
## Installation/On-Screen Mesa/Ubuntu

- references: https://openfoamwiki.net/index.php/Installation/On-Screen_Mesa/Ubuntu#Ubuntu_18.04

- notes: change the scons command to

  `cd mesa<latest version>`

  `scons build=release texture_float=yes force_scons=1 libgl-xlib > log.makeMesa 2>&1 && cp -vr build/linux-x86_64/gallium/targets/libgl-xlib/* $ParaView_DIR/lib/`



