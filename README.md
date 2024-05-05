# драйвера RTL8125 для Synology

Этот репозиторий содержит код и инструкции по компиляции и установке драйвера RTL8125 для Synology DSM.

Моя текущая установка:

- Synology RS1219+ (avoton)
- DSM 7.2 (Linux 3.10.108)
- TRENDnet 2.5GBASE-T PCIe Network Adapter (TEG-25GECTX)


## Компилировать модуль

Приведенные ниже шаги основаны [this](https://help.synology.com/developer-guide/getting_started/prepare_environment.html) и [this](https://help.synology.com/developer-guide/compile_applications/compile_open_source_projects.html) документах.

1. Создайте подходящую среду Linux (не ваш NAS!). Я использовал образ Docker Ubuntu 22.04 LTS.
2. Настройка среды:

```bash
$ apt-get install git python3 python3-pip
$ mkdir -p /toolkit
$ cd /toolkit
$ git clone https://github.com/SynologyOpenSource/pkgscripts-ng
```

3. Разверните среду chroot:

```bash
$ cd /toolkit/pkgscripts-ng
$ git checkout DSM7.2
$ ./EnvDeploy -v 7.2 -p avoton # replace 'avoton' with your platform
```

4. Chroot into environment:

```bash
$ chroot /toolkit/build_env/ds.avoton-7.2
```

5. Скачать код:

```bash
$ mkdir -p /usr/src
$ cd /usr/src
$ git clone https://github.com/tabrezm/r8125-synology
$ cd r8125-synology/src
```

Если вы компилируете для другой версии DSM, вам необходимо обновить BASEDIRmake-файл:

```
	BASEDIR := /usr/local/x86_64-pc-linux-gnu/x86_64-pc-linux-gnu/sys-root/usr/lib/modules/DSM-7.2
```

6. Скомпилировать модуль:

```bash
$ make
```

У вас не должно быть ошибок сборки и r8215.koв текущем каталоге.

## Установить модуль

1. Скопируйте r8215.koв папку, доступную на вашем устройстве Synology, например в общую папку или с помощью SCP.

2. Подключитесь по SSH и выполните следующие команды:

```bash
$ sudo -i
$ cp r8215.ko /lib/modules
$ insmod /lib/modules/r8125.ko
```

3. Подтвердите, что модуль загружен:

```
root@synology:~# lspci -v | grep r8125
	Kernel driver in use: r8125
```

4. Настройте новый интерфейс:

```bash
$ ip link set up eth4 # replace 'eth4' with your interface
```

На этом этапе вы сможете увидеть детали интерфейса, используя ifconfig eth4.

5. Обновите модули, чтобы r8125они загружались автоматически при следующей загрузке:

```bash
$ ln -s /bin/kmod /sbin/depmod
$ depmod -a # warnings are safe to ignore
```

