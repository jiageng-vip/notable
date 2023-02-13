---
attachments: [Clipboard_2022-05-03-17-55-37.png, Clipboard_2022-05-03-17-59-34.png]
tags: [MacOs]
title: Mac 配置iterm2以使用rz/sz
created: '2022-05-03T09:51:53.139Z'
modified: '2022-05-03T10:02:17.980Z'
---

# Mac 配置iterm2以使用rz/sz

### 安装lrzsz
##### brew install lrzsz
##### 下载iterm2-zmodem脚本
地址: https://github.com/mmastrac/iterm2-zmodem
然后将iterm2-send-zmodem.sh & iterm2-recv-zmodem.sh放到/usr/local/bin/
<center class="half">
    <img src="@attachment/Clipboard_2022-05-03-17-55-37.png" width="80%">
</center>

##### iterm2-recv-zmodem.sh

```
#!/bin/bash
# Author: Matt Mastracci (matthew@mastracci.com)
# AppleScript from http://stackoverflow.com/questions/4309087/cancel-button-on-osascript-in-a-bash-script
# licensed under cc-wiki with attribution required
# Remainder of script public domain
​
osascript -e 'tell application "iTerm2" to version' > /dev/null 2>&1 && NAME=iTerm2 || NAME=iTerm
if [[ $NAME = "iTerm" ]]; then
    FILE=$(osascript -e 'tell application "iTerm" to activate' -e 'tell application "iTerm" to set thefile to choose folder with prompt "Choose a folder to place received files in"' -e "do shell script (\"echo \"&(quoted form of POSIX path of thefile as Unicode text)&\"\")")
else
    FILE=$(osascript -e 'tell application "iTerm2" to activate' -e 'tell application "iTerm2" to set thefile to choose folder with prompt "Choose a folder to place received files in"' -e "do shell script (\"echo \"&(quoted form of POSIX path of thefile as Unicode text)&\"\")")
fi
​
if [[ $FILE = "" ]]; then
    echo Cancelled.
    # Send ZModem cancel
    echo -e \\x18\\x18\\x18\\x18\\x18
    sleep 1
    echo
    echo \# Cancelled transfer
else
    cd "$FILE"
    /usr/local/bin/rz --rename --escape --binary --bufsize 4096
    sleep 1
    echo
    echo
    echo \# Sent \-\> $FILE
fi
```
##### iterm2-send-zmodem.sh
```
#!/bin/bash
# Author: Matt Mastracci (matthew@mastracci.com)
# AppleScript from http://stackoverflow.com/questions/4309087/cancel-button-on-osascript-in-a-bash-script
# licensed under cc-wiki with attribution required
# Remainder of script public domain
​
osascript -e 'tell application "iTerm2" to version' > /dev/null 2>&1 && NAME=iTerm2 || NAME=iTerm
if [[ $NAME = "iTerm" ]]; then
    FILE=$(osascript -e 'tell application "iTerm" to activate' -e 'tell application "iTerm" to set thefile to choose file with prompt "Choose a file to send"' -e "do shell script (\"echo \"&(quoted form of POSIX path of thefile as Unicode text)&\"\")")
else
    FILE=$(osascript -e 'tell application "iTerm2" to activate' -e 'tell application "iTerm2" to set thefile to choose file with prompt "Choose a file to send"' -e "do shell script (\"echo \"&(quoted form of POSIX path of thefile as Unicode text)&\"\")")
fi
if [[ $FILE = "" ]]; then
    echo Cancelled.
    # Send ZModem cancel
    echo -e \\x18\\x18\\x18\\x18\\x18
    sleep 1
    echo
    echo \# Cancelled transfer
else
    /usr/local/bin/sz "$FILE" --escape --binary --bufsize 4096
    sleep 1
    echo
    echo \# Received "$FILE"
fi
```
##### 配置iterm2 Trigger
选择Profiles > Default > Advanced > Triggers > Edit
```
# 发送 sz
Regular expression: rz waiting to receive.\*\*B0100 (注意这里是这样)
Action: Run Silent Coprocess
Parameters: /usr/local/bin/iterm2-send-zmodem.sh
```
```
# 接收 rz
Regular expression:\*\*B00000000000000
Action: Run Silent Coprocess
Parameters: /usr/local/bin/iterm2-recv-zmodem.sh
```
配置完成如下:
<center class="half">
    <img src="@attachment/Clipboard_2022-05-03-17-59-34.png" width="80%">
</center>
配置完成之后就可以使用了.
