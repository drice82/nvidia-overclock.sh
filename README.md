# 矿机系统
## 安装Ubuntu18.06 到U盘
## 安装nvidia驱动
然后运行以下命令以获取有关图形卡和可用驱动程序的信息：
```
sudo apt install ubuntu-drivers-common
ubuntu-drivers devices
sudo apt install nvidia-driver-470
```
安装完成后，重新启动系统：
```
sudo reboot
```
当系统返回时，您可以使用nvidia-smi监视工具查看图形卡的状态：
```
nvidia-smi
```
## 安装AMD驱动
参考AMD官方网站
如果遇到
The following packages have unmet dependencies:
 amdgpu-lib32 : Depends: libdrm2-amdgpu:i386 (= 1:2.4.106-1286092)
 是因为系统是64位的。
 ```
sudo dpkg --add-architecture i386
sudo apt update 
 ```
 解压安装驱动，用headless模式
```
./amdgpu-pro-install -y --opencl=pal,legacy --headless
```
 
## 安装 xorg，openbox
sudo apt-get install xorg openbox

## 关闭休眠
https://blog.csdn.net/ciel_yu/article/details/117130048

## 安装runit守护
```
sudo apt install runit
sudo apt install runit-systemd
```

# settings
copy edid.txt to /etc/
```
sudo nvidia-xconfig -a --allow-empty-initial-configuration \
--use-display-device="DFP-0" --connected-monitor="DFP-0" \
--custom-edid="DFP-0:/etc/edid.txt"  --cool-bits=28

sudo reboot
```
cool-bits 要用28，如果用31，在装完AMD驱动后，可能遇到无法超频显存的问题。原因不清楚。

ubuntu 18.04 adjust fan
```
sudo DISPLAY=:0 XAUTHORITY=/run/user/121/gdm/Xauthority nvidia-settings -c :0 -a [gpu:GPUID]/GPUFanControlState=1
sudo DISPLAY=:0 XAUTHORITY=/run/user/121/gdm/Xauthority  nvidia-settings -c :0 -a [fan:GPUID]/GPUTargetFanSpeed=70
```
ubuntu 18.04 overclock
```
sudo DISPLAY=:0 XAUTHORITY=/run/user/121/gdm/Xauthority nvidia-settings -c :0 -a [gpu:GPUID]/GPUGraphicsClockOffset[3]=64
sudo DISPLAY=:0 XAUTHORITY=/run/user/121/gdm/Xauthority nvidia-settings -c :0 -a [gpu:GPUID]/GPUMemoryTransferRateOffset[3]=500
```
20 和30系显卡的话一般吧[3]改成[4]。

# nvidia-overclock.sh

A simple script to manage overclocking of NVIDIA graphics cards on Linux.  You can use it to manually overclock the card values each time or install a systemd service that will be run at startup to automatically perform the overclocking.

## Requirements

nvidia-overclock.sh requires the following software to be installed:

* POSIX compliant shell
* systemd (needed for the optional service)

## Installation

1. Download the nvidia-overclock.sh script and install it to a directory in your path.

Example: curl the script to /usr/local/bin
```
$ sudo curl https://raw.githubusercontent.com/plyint/nvidia-overclock.sh/master/nvidia-overclock.sh -o /usr/local/bin/nvidia-overclock.sh
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  10670  100  10670    0     0  34874      0 --:--:-- --:--:-- --:--:-- 35106
```

2. Ensure the script has the executable permissions set
```
sudo chmod 755 /usr/local/bin/nvidia-overclock.sh
```

3. Update the values in the overclock() function with values for your GPUs. 

Remove or add lines for calling nvidia-settings based on the number of graphics cards in your system.  You will have to experiment with the *Graphics Clock* and *Memory Transfer Rate* values that work best for your graphics cards.  Each graphics card is different and even cards that are the same model may tolerate overclocking differently.

```
overclock () {
  # The following default overclock values for the NVIDIA GTX 1070 
  # were found on average to be stable:
  # - Graphics Clock       = 100
  # - Memory Transfer Rate = 1300
  #
  # Adjust these values for each card as needed.  Some cards are more 
  # unstable than others and will only tolerate less overclocking.  Other
  # cards might tolerate above normal overclocking. If you are unsure of the 
  # starting values to use for your graphics cards, try searching online
  # for what other people with your same graphics cards have found to be stable.
  #
  # Note: The lines below were used to configure a system with 6 graphics cards.
  # You will neeed to add/remove lines based on the number of graphics cards in 
  # your particular system.
  log "Calling nvidia-settings to overclock GPU(s).."
  log "$(nvidia-settings -c :0 -a '[gpu:0]/GPUGraphicsClockOffset[3]=100' -a '[gpu:0]/GPUMemoryTransferRateOffset[3]=1300')"
  log "$(nvidia-settings -c :0 -a '[gpu:1]/GPUGraphicsClockOffset[3]=100' -a '[gpu:1]/GPUMemoryTransferRateOffset[3]=1300')"
  log "$(nvidia-settings -c :0 -a '[gpu:2]/GPUGraphicsClockOffset[3]=100' -a '[gpu:2]/GPUMemoryTransferRateOffset[3]=1300')"
  log "$(nvidia-settings -c :0 -a '[gpu:3]/GPUGraphicsClockOffset[3]=100' -a '[gpu:3]/GPUMemoryTransferRateOffset[3]=1300')"
  log "$(nvidia-settings -c :0 -a '[gpu:4]/GPUGraphicsClockOffset[3]=100' -a '[gpu:4]/GPUMemoryTransferRateOffset[3]=1300')"
  log "$(nvidia-settings -c :0 -a '[gpu:5]/GPUGraphicsClockOffset[3]=100' -a '[gpu:5]/GPUMemoryTransferRateOffset[3]=1300')"
}
```

4. Install the systemd services to run at startup (Optional)

Use the following command to install the systemd services that will start the X server using the *startx* command and then run the *nvidia-overclock.sh* script.

```
sudo /usr/local/bin/nvidia-overclock.sh install-svc -x
```

This will install the service and use "startx" to automatically start
XWindows.  If you already have XWindows installed and configured to run
automatically on boot, then omit the -x option:

```
sudo /usr/local/bin/nvidia-overclock.sh install-svc
```

## Usage

If the systemd script was installed, then you should not have to do anything going forward.  Everytime you start your machine, the service will be run and it will automatically overclock the graphics cards.

If the systemd script was not installed or you want to execute the commands manually, then you can run the commands directly.  Be sure that an X server has started first before running the script; otherwise, the script will be unable to overclock the graphics cards.

```
# Run the overclock command of the script
/usr/local/bin/nvidia-overclock.sh overclock
```

The basic commands available are:
* overclock - sets the overclock values for the graphics card(s) as defined in the overclock function
* auto - same as the overclock method except, optional parameters can be passed to perform logging and check that an X server is started

To see a list of available commands along with detailed descriptions, just run the script without any parameters.

```
# Display the available commands
/usr/local/bin/nvidia-overclock.sh
```

## Uninstall

To remove the script from your system, run the uninstall-svc command as root (if the systemd service was installed) and then delete the script.

```
sudo /usr/local/bin/nvidia-overclock.sh uninstall-svc
rm -f /usr/local/bin/nvidia-overclock.sh
```

## Troubleshooting

### Systemd
The installed systemd services are designed to run after runlevel4 in the hopes that most other scripts that might load kernel modules have already run.  You may need to adjust the "After" dependency listed under the Unit section if you observe the service running earlier than expected.

### Logging
By default the installed systemd service will log output to the logfile /var/log/nvidia-overclock.log.  Check this log for errors if you encounter issues.
