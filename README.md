# Домашнее задание № 8 по теме: "Загрузка системы". К курсу Administrator Linux. Professional

## Задание

- Попасть в систему без пароля несколькими способами
- Установить систему с LVM, после чего переименовать VG
- Добавить модуль в initrd

- Дистрибутив выбран - CentOS 7 с SELinux

## Задание 1. Попасть в систему без пароля

- Способ rd.break

```bash
# Нажать _e_ в меню загрузчика grub. Удалить: "console=tty0 console=ttyS0,115200n8 quiet" и добавить в конце rd.break
#/vmlinuz-3.10.0-862.2.3.el7.x86_64 root=/dev/mapper/VolGroup00-LogVol00 ro no_timer_check net.ifnames=0 biosdevname=0 elevator=noop crashkernel=auto rd.lvm.lv=VolGroup00/LogVol00 rd.lvm.lv=VolGroup00/LogVol01 rhgb rd.break
# Нажать Ctrl + x

mount -o remount,rw /sysroot
chroot /sysroot
passwd
touch /.autorelabel
exit
exit
```

- Способ init=/sysroot/bin/sh

Этот способ не подходит для дистрибутивов с SELinux. Т.к. параметры selinux определяются в userspace, они определяются в момент загрузки системы, своими действиями (передача параметра init) мы прерываем загрузку параметров. Поэтому после смены пароля этим способом файлы passwd и shadow будут иметь не верные метки selinux (перестанут быть доверенными) и ни один пользователь уже не сможет авторизоваться без передачи параметра во время загрузки selinux=0

```bash
# Нажать _e_ в меню загрузчика grub. Удалить: "console=tty0 console=ttyS0,115200n8 quiet", заменить ro на rw и добавить в конце init=/sysroot/bin/sh
#/vmlinuz-3.10.0-862.2.3.el7.x86_64 root=/dev/mapper/VolGroup00-LogVol00 rw no_timer_check net.ifnames=0 biosdevname=0 elevator=noop crashkernel=auto rd.lvm.lv=VolGroup00/LogVol00 rd.lvm.lv=VolGroup00/LogVol01 rhgb init=/sysroot/bin/sh
# Нажать Ctrl + x

chroot /sysroot
passwd
exit
reboot
```

- Делаем вывод, что сменить пароль в нашем случае можно только первым способом.

## Задание 2. Переименовать Volume Group

```bash
vgs
vgrename VolGroup00 OtusRoot
sed -i "s/VolGroup00/OtusRoot/g" /etc/fstab
sed -i "s/VolGroup00/OtusRoot/g" /etc/default/grub
sed -i "s/VolGroup00/OtusRoot/g" /boot/grub2/grub.cfg
systemctl reboot
vgs
```

## Задание 3. Добавить модуль в initrd

```bash
mkdir /usr/lib/dracut/modules.d/01test
# Файлы test.sh и module-setup.sh находятся в директории dracut этого репозитория
vi /usr/lib/dracut/modules.d/01test/test.sh
vi /usr/lib/dracut/modules.d/01test/module-setup.sh
dracut -f -v

lsinitrd -m /boot/initramfs-$(uname -r).img | grep test

vi  /etc/default/grub
# Удалить опции: console=tty0 console=ttyS0,115200n8 rghb quiet
grub2-mkconfig -o /boot/grub2/grub.cfg
systemctl reboot
```