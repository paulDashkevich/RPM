# RPM
Домашнее задание: создать свой пакет и опубликовать репозиторий

# Создание rpm nginx с модулем openssl 1.1.1
1. Загружаем сырцы nginx'a и openssl'я
  * wget https://nginx.org/packages/centos/7/SRPMS/nginx-1.18.0-1.el7.ngx.src.rpm
  * wget https://www.openssl.org/source/latest.tar.gz
2. Устанавливаем пакет nginx и получаем директорию rpmbuild с деревом каталогов
```
rpm -ivvh /root/nginx-1.18.0-1.el7.ngx.src.rpm

[root@centos rpmbuild]# ll
total 4
drwxr-xr-x. 4 root root   43 Jul 19 12:57 BUILD
drwxr-xr-x. 2 root root    6 Jul 19 12:59 BUILDROOT
drwxr-xr-x. 3 root root   20 Jun 28 13:42 RPMS
drwxr-xr-x. 2 root root 4096 Jul 19 12:44 SOURCES
drwxr-xr-x. 2 root root   52 Jul 19 12:57 SPECS
drwxr-xr-x. 2 root root   37 Jun 28 13:41 SRPMS

```
3. Распаковываем архив с исходниками openssl:
```
tar -xvf latest.tar.gz
```
4. Проверяем заранее зависимости до сборки nginx
```
yum-builddep /root/rpmbuild/SPECS/nginx.spec
```
5. Вносим изменения в файл спецификации для сборки пакета с нужным по заданию модулем openssl
```
сюда (так как у меня 7 центОсь)
%if 0%{?rhel} == 7
BuildRequires: redhat-lsb-core
%define _group System Environment/Daemons
%define epoch 1
Epoch: %{epoch}
Requires(pre): shadow-utils
Requires: systemd
BuildRequires: systemd
Requires: openssl >= 1.1.1
BuildRequires: openssl-devel >= 1.1.1
%define dist .el7_4
%else
Requires: openssl >= 1.1.1
BuildRequires: openssl-devel >= 1.1.1
%define dist .el7
%endif

и в блок build/configure
%build
./configure %{BASE_CONFIGURE_ARGS} \
    --with-cc-opt="%{WITH_CC_OPT}" \
    --with-ld-opt="%{WITH_LD_OPT}" \
    --with-openssl=/root/openssl-1.1.1g
```
6. Запускаем сборку пакета
```
rpmbuild -bb /root/rpmbuild/SPECS/nginx.spec

По завершению видим:
Checking for unpackaged file(s): /usr/lib/rpm/check-files /root/rpmbuild/BUILDROOT/nginx-1.18.0-1.el7_4.ngx.x86_64
Wrote: /root/rpmbuild/RPMS/x86_64/nginx-1.18.0-1.el7_4.ngx.x86_64.rpm
Wrote: /root/rpmbuild/RPMS/x86_64/nginx-debuginfo-1.18.0-1.el7_4.ngx.x86_64.rpm
Executing(%clean): /bin/sh -e /var/tmp/rpm-tmp.PWxaOZ
+ umask 022
+ cd /root/rpmbuild/BUILD
+ cd nginx-1.18.0
+ /usr/bin/rm -rf /root/rpmbuild/BUILDROOT/nginx-1.18.0-1.el7_4.ngx.x86_64
+ exit 0
```
7. Проверяем (можно и не делать), что пакет лежит и готов к работе
```
[root@centos ~]# ll rpmbuild/RPMS/x86_64/
total 2
-rw-r--r--. 1 root root 2019860 Jul 19 13:53 nginx-1.18.0-1.el7_4.ngx.x86_64.rpm
-rw-r--r--. 1 root root 1914900 Jul 19 13:53 nginx-debuginfo-1.18.0-1.el7_4.ngx.x86_64.rpm
```
8. Устанавливаем полученный пакет, стартуем и проверяем nginx
```
rpm -i /root/rpmbuild/RPMS/x86_64/rpmbuild/RPMS/x86_64/
***
nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
***
systemctl start nginx
[root@centos ~]# systemctl status nginx
● nginx.service - nginx - high performance web server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; vendor preset: disabled)
   Active: active (running) since Sun 2020-07-19 13:27:26 +03; 33min ago
     Docs: http://nginx.org/en/docs/
  Process: 3968 ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx.conf (code=exited, status=0/SUCCESS)
 Main PID: 3972 (nginx)
    Tasks: 2
   CGroup: /system.slice/nginx.service
           ├─3972 nginx: master process /usr/sbin/nginx -c /etc/nginx/nginx.conf
           └─3973 nginx: worker process

Jul 19 13:27:26 centos.local systemd[1]: Starting nginx - high performance web server...
Jul 19 13:27:26 centos.local systemd[1]: Started nginx - high performance web server.
***
```
##II Создаём репозиторий и публикуем для проверки ДЗ##
1. Создаем каталог для размещения пакетов
```
mkdir /usr/share/nginx/html/repo
```
2. Копируем туда собранный ранее пакет nginx и добавляем (просто так) пакет Percona
```
cp /root/rpmbuild/RPMS/x86_64/nginx-1.18.0-1.el7_4.ngx.x86_64.rpm /usr/share/nginx/html/repo/
wget http://www.percona.com/downloads/percona-release/redhat/0.1-6/percona-release-0.1-6.noarch.rpm -O /usr/share/nginx/html/repo/percona-release-0.1-6.noarch.rpm
```
3. Инициализируем репозиторий командой и перезапускаем nginx (nginx -s reload):
[root@centos ~]# createrepo /usr/share/nginx/html/repo/
```
Spawning worker 0 with 1 pkgs
Spawning worker 1 with 1 pkgs
Spawning worker 2 with 0 pkgs
Spawning worker 3 with 0 pkgs
Workers Finished
Saving Primary metadata
Saving file lists metadata
Saving other metadata
Generating sqlite DBs
Sqlite DBs complete
```
4. Вывод репозитория в сети
http://repo.jetrom.by

