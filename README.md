		Домашнее задание 6

1) Создать свой RPM пакет (можно взять свое приложение, либо собрать, например,
апач с определенными опциями)
2) Создать свой репозиторий и разместить там ранее собранный RPM

Реализовать это все либо в Vagrant, либо развернуть у себя через NGINX и дать ссылку на репозиторий.


1. Подготавливаем vagrantfile  для нашего стенда и включаем вирт машину.

```

# Vagrant test RMP
MACHINES = {
  :"test-rpm" => {
              :box_name => "centos/8",
              :box_version => "2011.0",
              :cpus => 2,
              :memory => 2048,
            }
}

Vagrant.configure("2") do |config|
  MACHINES.each do |boxname, boxconfig|
        config.vm.synced_folder ".", "/vagrant", disabled: true
        config.vm.define boxname do |box|
      box.vm.box = boxconfig[:box_name]
      box.vm.box_version = boxconfig[:box_version]
      box.vm.host_name = boxname.to_s
      box.vm.provider "virtualbox" do |v|
        v.memory = boxconfig[:memory]
        v.cpus = boxconfig[:cpus]
 	 end
    end
  end
end

```

2. После подключения к ней обновляем наш Centos 8

```

Добавляем зеркала 

[root@test-rpm ~]sudo -i
[root@test-rpm ~]# sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*
[root@test-rpm ~]# sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*
[root@test-rpm ~]# yum update -y

```

3. Создаем свой RPM пакет

```

[root@test-rpm ~]yum install -y \redhat-lsb-core \wget \rpmdevtools \rpm-build \createrepo \yum-utils \gcc

```

4. Загрузим SRPM пакет NGINX для дальнейшей работы над ним:

```

[root@test-rpm ~]# wget https://nginx.org/packages/centos/8/SRPMS/nginx-1.24.0-1.el8.ngx.src.rpm
--2023-05-24 20:46:10--  https://nginx.org/packages/centos/8/SRPMS/nginx-1.24.0-1.el8.ngx.src.rpm
Resolving nginx.org (nginx.org)... 3.125.197.172, 52.58.199.22, 2a05:d014:edb:5702::6, ...
Connecting to nginx.org (nginx.org)|3.125.197.172|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1138057 (1.1M) [application/x-redhat-package-manager]
Saving to: 'nginx-1.24.0-1.el8.ngx.src.rpm'



nginx-1.24.0-1.el8.ngx.src.rpm                0%[                                                                                           ]       0  --.-KB/s               
nginx-1.24.0-1.el8.ngx.src.rpm               82%[==========================================================================>                ] 919.71K  4.49MB/s               
nginx-1.24.0-1.el8.ngx.src.rpm              100%[==========================================================================================>]   1.08M  4.88MB/s    in 0.2s    

2023-05-24 20:46:10 (4.88 MB/s) - 'nginx-1.24.0-1.el8.ngx.src.rpm' saved [1138057/1138057]


```

5. При установке такого пакета в домашней директории создается древо каталогов для
сборки. Так как мы от рута работаем собственно он ругается на это.

```

[root@test-rpm ~]# rpm -i nginx-1.*
warning: nginx-1.24.0-1.el8.ngx.src.rpm: Header V4 RSA/SHA256 Signature, key ID 7bd9bf62: NOKEY
warning: user builder does not exist - using root
warning: group builder does not exist - using root
warning: user builder does not exist - using root
warning: group builder does not exist - using root
warning: user builder does not exist - using root
warning: group builder does not exist - using root
warning: user builder does not exist - using root
warning: group builder does not exist - using root
warning: user builder does not exist - using root
warning: group builder does not exist - using root
warning: user builder does not exist - using root
warning: group builder does not exist - using root
warning: user builder does not exist - using root
warning: group builder does not exist - using root
warning: user builder does not exist - using root
warning: group builder does not exist - using root
warning: user builder does not exist - using root
warning: group builder does not exist - using root
warning: user builder does not exist - using root
warning: group builder does not exist - using root
warning: user builder does not exist - using root

```

6. Скачиваем и разархивируем исходник для openssl - он потребуется при сборке

```

wget https://www.openssl.org/source/openssl-1.1.1t.tar.gz
--2023-05-24 21:01:29--  https://www.openssl.org/source/openssl-1.1.1t.tar.gz
Resolving www.openssl.org (www.openssl.org)... 96.17.6.81, 2a02:2d8:0:799c::c1e, 2a02:2d8:0:79a1::c1e
Connecting to www.openssl.org (www.openssl.org)|96.17.6.81|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 9881866 (9.4M) [application/x-gzip]
Saving to: 'openssl-1.1.1t.tar.gz'


openssl-1.1.1t.tar.gz                         0%[                                                                                           ]       0  --.-KB/s               
openssl-1.1.1t.tar.gz                         1%[>                                                                                          ] 126.39K   626KB/s               
openssl-1.1.1t.tar.gz                         7%[======>                                                                                    ] 770.64K  1.87MB/s               
openssl-1.1.1t.tar.gz                        18%[================>                                                                          ]   1.79M  2.58MB/s               
openssl-1.1.1t.tar.gz                        27%[========================>                                                                  ]   2.59M  2.76MB/s               
openssl-1.1.1t.tar.gz                        57%[===================================================>                                       ]   5.45M  4.43MB/s               
openssl-1.1.1t.tar.gz                        66%[===========================================================>                               ]   6.25M  4.19MB/s               
openssl-1.1.1t.tar.gz                        98%[========================================================================================>  ]   9.28M  5.49MB/s               
openssl-1.1.1t.tar.gz                       100%[==========================================================================================>]   9.42M  5.52MB/s    in 1.7s    

2023-05-24 21:01:31 (5.52 MB/s) - 'openssl-1.1.1t.tar.gz' saved [9881866/9881866]

```

7. Разархивируем наш пакот 

```

[root@test-rpm ~]# tar -xvf popenssl-1.1.1t.tar.gz openssl-1.1.1t/

```

8. Заранее поставим все зависимости, чтобы в процессе сборки не было ошибок

```

[root@test-rpm ~]# yum-builddep rpmbuild/SPECS/nginx.spec
Failed to set locale, defaulting to C.UTF-8
Last metadata expiration check: 0:43:20 ago on Wed May 24 20:18:54 2023.
Package systemd-239-51.el8_5.2.x86_64 is already installed.
Dependencies resolved.
===============================================================================================================================================================================
 Package                                          Architecture                        Version                                        Repository                           Size
===============================================================================================================================================================================
Installing:
 openssl-devel                                    x86_64                              1:1.1.1k-5.el8_5                               baseos                              2.3 M
 pcre2-devel                                      x86_64                              10.32-2.el8                                    baseos                              605 k
 zlib-devel                                       x86_64                              1.2.11-17.el8                                  baseos                               58 k
Installing dependencies:
 keyutils-libs-devel                              x86_64                              1.5.10-9.el8                                   baseos                               48 k
 krb5-devel                                       x86_64                              1.18.2-14.el8                                  baseos                              560 k
 libcom_err-devel                                 x86_64                              1.45.6-2.el8                                   baseos                               38 k
 libkadm5                                         x86_64                              1.18.2-14.el8                                  baseos                              187 k
 libselinux-devel                                 x86_64                              2.9-5.el8                                      baseos                              200 k
 libsepol-devel                                   x86_64                              2.9-3.el8                                      baseos                               87 k
 libverto-devel                                   x86_64                              0.3.0-5.el8                                    baseos                               18 k
 pcre2-utf16                                      x86_64                              10.32-2.el8                                    baseos                              229 k
 pcre2-utf32                                      x86_64                              10.32-2.el8                                    baseos                              220 k

Transaction Summary
===============================================================================================================================================================================
Install  12 Packages

Total download size: 4.5 M
Installed size: 8.1 M
Is this ok [y/N]: y
Downloading Packages:
(1/12): libcom_err-devel-1.45.6-2.el8.x86_64.rpm                                                                                                99 kB/s |  38 kB     00:00
(2/12): keyutils-libs-devel-1.5.10-9.el8.x86_64.rpm                                                                                            117 kB/s |  48 kB     00:00
(3/12): krb5-devel-1.18.2-14.el8.x86_64.rpm                                                                                                    889 kB/s | 560 kB     00:00
(4/12): libkadm5-1.18.2-14.el8.x86_64.rpm                                                                                                      759 kB/s | 187 kB     00:00
(5/12): libselinux-devel-2.9-5.el8.x86_64.rpm                                                                                                  772 kB/s | 200 kB     00:00
(6/12): libverto-devel-0.3.0-5.el8.x86_64.rpm                                                                                                  140 kB/s |  18 kB     00:00
(7/12): libsepol-devel-2.9-3.el8.x86_64.rpm                                                                                                    597 kB/s |  87 kB     00:00
(8/12): pcre2-utf16-10.32-2.el8.x86_64.rpm                                                                                                     722 kB/s | 229 kB     00:00
(9/12): pcre2-devel-10.32-2.el8.x86_64.rpm                                                                                                     1.4 MB/s | 605 kB     00:00
(10/12): openssl-devel-1.1.1k-5.el8_5.x86_64.rpm                                                                                               3.7 MB/s | 2.3 MB     00:00
(11/12): pcre2-utf32-10.32-2.el8.x86_64.rpm                                                                                                    987 kB/s | 220 kB     00:00
(12/12): zlib-devel-1.2.11-17.el8.x86_64.rpm                                                                                                   418 kB/s |  58 kB     00:00
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                          3.4 MB/s | 4.5 MB     00:01
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction

```

9. Создаем каталоги для сборки 

```

[root@test-rpm rpmbuild]# rpmdev-setuptree
[root@test-rpm rpmbuild]# ls
BUILD  RPMS  SOURCES  SPECS  SRPMS

```

10. Приступаем к сборке RPM пакета:

```

[root@test-rpm ~]# rpmbuild -bb rpmbuild/SPECS/nginx.spec
Executing(%prep): /bin/sh -e /var/tmp/rpm-tmp.1xOc3p
+ umask 022
+ cd /root/rpmbuild/BUILD
+ cd /root/rpmbuild/BUILD
+ rm -rf nginx-1.24.0
+ /usr/bin/gzip -dc /root/rpmbuild/SOURCES/nginx-1.24.0.tar.gz
+ /usr/bin/tar -xof -
+ STATUS=0
+ '[' 0 -ne 0 ']'
+ cd nginx-1.24.0
+ /usr/bin/chmod -Rf a+rX,u+w,g-w,o-w .
+ exit 0
Executing(%build): /bin/sh -e /var/tmp/rpm-tmp.4R9glr
+ umask 022
+ cd /root/rpmbuild/BUILD
+ cd nginx-1.24.0
++ echo '--prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --group=nginx --with-compat --with-file-aio --with-threads --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-mail --with-mail_ssl_module --with-stream --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module'
+++ pcre2-config --cflags
++ echo -O2 -g -pipe -Wall -Werror=format-security -Wp,-D_FORTIFY_SOURCE=2 -Wp,-D_GLIBCXX_ASSERTIONS -fexceptions -fstack-protector-strong -grecord-gcc-switches -specs=/usr/lib/rpm/redhat/redhat-hardened-cc1 -specs=/usr/lib/rpm/redhat/redhat-annobin-cc1 -m64 -mtune=generic -fasynchronous-unwind-tables -fstack-clash-protection -fcf-protection
+ ./configure --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --group=nginx --with-compat --with-file-aio --with-threads --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-mail --with-mail_ssl_module --with-stream --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module '--with-cc-opt=-O2 -g -pipe -Wall -Werror=format-security -Wp,-D_FORTIFY_SOURCE=2 -Wp,-D_GLIBCXX_ASSERTIONS -fexceptions -fstack-protector-strong -grecord-gcc-switches -specs=/usr/lib/rpm/redhat/redhat-hardened-cc1 -specs=/usr/lib/rpm/redhat/redhat-annobin-cc1 -m64 -mtune=generic -fasynchronous-unwind-tables -fstack-clash-protection -fcf-protection -fPIC' '--with-ld-opt=-Wl,-z,relro -Wl,-z,now -pie' --with-debug
checking for OS
 + Linux 4.18.0-240.1.1.el8_3.x86_64 x86_64
checking for C compiler ... found
 + using GNU C compiler
 + gcc version: 8.5.0 20210514 (Red Hat 8.5.0-4) (GCC)
checking for gcc -pipe switch ... found
checking for --with-ld-opt="-Wl,-z,relro -Wl,-z,now -pie" ... found
checking for -Wl,-E switch ... found
checking for gcc builtin atomic operations ... found

...................................

```

11. Убедимся, что пакеты создались:

```

[root@test-rpm ~]# ll rpmbuild/RPMS/x86_64/
total 3200
-rw-r--r--. 1 root root  855032 May 24 21:36 nginx-1.24.0-1.el8.ngx.x86_64.rpm
-rw-r--r--. 1 root root 2417900 May 24 21:36 nginx-debuginfo-1.24.0-1.el8.ngx.x86_64.rpm

```

12.  Теперь можно установить наш пакет и убедиться, что nginx работает

```

[root@test-rpm ~]# yum localinstall -y \rpmbuild/RPMS/x86_64/nginx-1.24.0-1.el8.ngx.x86_64.rpm
Failed to set locale, defaulting to C.UTF-8
Last metadata expiration check: 1:25:10 ago on Wed May 24 20:18:54 2023.
Dependencies resolved.
===============================================================================================================================================================================
 Package                             Architecture                         Version                                             Repository                                  Size
===============================================================================================================================================================================
Installing:
 nginx                               x86_64                               1:1.24.0-1.el8.ngx                                  @commandline                               835 k

Transaction Summary
===============================================================================================================================================================================
Install  1 Package

Total size: 835 k
Installed size: 2.8 M
Downloading Packages:
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                                                                       1/1
  Running scriptlet: nginx-1:1.24.0-1.el8.ngx.x86_64                                                                                                                       1/1
  Installing       : nginx-1:1.24.0-1.el8.ngx.x86_64                                                                                                                       1/1
  Running scriptlet: nginx-1:1.24.0-1.el8.ngx.x86_64                                                                                                                       1/1
----------------------------------------------------------------------

Thanks for using nginx!

Please find the official documentation for nginx here:
* https://nginx.org/en/docs/

Please subscribe to nginx-announce mailing list to get
the most important news about nginx:
* https://nginx.org/en/support.html

Commercial subscriptions for nginx are available on:
* https://nginx.com/products/

----------------------------------------------------------------------

  Verifying        : nginx-1:1.24.0-1.el8.ngx.x86_64                                                                                                                       1/1

Installed:
  nginx-1:1.24.0-1.el8.ngx.x86_64

Complete!

[root@test-rpm ~]# systemctl start nginx
[root@test-rpm ~]#
[root@test-rpm ~]# systemctl status nginx.service
● nginx.service - nginx - high performance web server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: active (running) since Wed 2023-05-24 21:44:35 UTC; 16s ago
     Docs: http://nginx.org/en/docs/
  Process: 52213 ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx.conf (code=exited, status=0/SUCCESS)
 Main PID: 52214 (nginx)
    Tasks: 3 (limit: 12421)
   Memory: 3.0M
   CGroup: /system.slice/nginx.service
           ├─52214 nginx: master process /usr/sbin/nginx -c /etc/nginx/nginx.conf
           ├─52215 nginx: worker process
           └─52216 nginx: worker process

May 24 21:44:35 test-rpm systemd[1]: Starting nginx - high performance web server...
May 24 21:44:35 test-rpm systemd[1]: Started nginx - high performance web server.


```

13. Приступим к созданию своего репозитория. Директория для статики у NGINX по умолчанию /usr/share/nginx/html. Создадим там каталог repo:

```

[root@test-rpm ~]# mkdir /usr/share/nginx/html/repo

```

14. Копируем туда наш собранный RPM и, например, RPM для установки репозитория Percona-Server:

```

[root@test-rpm ~]# cp rpmbuild/RPMS/x86_64/nginx-1.24.0-1.el8.ngx.x86_64.rpm /usr/share/nginx/html/repo/

[root@test-rpm ~]# wget https://downloads.percona.com/downloads/percona-distribution-mysql-ps/percona-distribution-mysql-ps-8.0.28/binary/redhat/8/x86_64/percona-orchestrator-3.2.6-2.el8.x86_64.rpm -O /usr/share/nginx/html/repo/percona-orchestrator-3.2.6-2.el8.x86_64.rpm
--2023-05-24 21:49:33--  https://downloads.percona.com/downloads/percona-distribution-mysql-ps/percona-distribution-mysql-ps-8.0.28/binary/redhat/8/x86_64/percona-orchestrator-3.2.6-2.el8.x86_64.rpm
Resolving downloads.percona.com (downloads.percona.com)... 162.220.4.222, 162.220.4.221, 74.121.199.231
Connecting to downloads.percona.com (downloads.percona.com)|162.220.4.222|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 5222976 (5.0M) [application/octet-stream]
Saving to: '/usr/share/nginx/html/repo/percona-orchestrator-3.2.6-2.el8.x86_64.rpm'

/usr/share/nginx/html/repo/percona-orchestr 100%[==========================================================================================>]   4.98M  3.31MB/s    in 1.5s

2023-05-24 21:49:35 (3.31 MB/s) - '/usr/share/nginx/html/repo/percona-orchestrator-3.2.6-2.el8.x86_64.rpm' saved [5222976/5222976]

```

15. Инициализируем репозиторий

```

root@test-rpm ~]# createrepo /usr/share/nginx/html/repo/
Directory walk started
Directory walk done - 2 packages
Temporary output repo path: /usr/share/nginx/html/repo/.repodata/
Preparing sqlite DBs
Pool started (with 5 workers)
Pool finished

```

16. Для прозрачности настроим в NGINX доступ к листингу каталога В location / в
    файле /etc/nginx/conf.d/default.conf добавим директиву autoindex on. В результате location
    будет выглядеть так: location / {
			root /usr/share/nginx/html;
			index index.html index.htm;
			autoindex on; < Добавили эту директиву
			}	

```

sed -i '10i\autoindex on;' /etc/nginx/conf.d/default.conf

ключ -i дает возможность сохранить изменения в наш файл при добавлении 10 строки с текстом autoindex on;

```

17. Проверяем синтаксис и перезапускаем NGINX

```

root@test-rpm ~]# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

[root@test-rpm ~]# nginx -s reload

```

18. Ради интереса можно посмотреть в браузере или через curl

```

[root@test-rpm ~]# curl -a http://localhost/repo/
<html>
<head><title>Index of /repo/</title></head>
<body>
<h1>Index of /repo/</h1><hr><pre><a href="../">../</a>
<a href="repodata/">repodata/</a>                                          24-May-2023 21:49                   -
<a href="nginx-1.24.0-1.el8.ngx.x86_64.rpm">nginx-1.24.0-1.el8.ngx.x86_64.rpm</a>                  24-May-2023 21:45              855032
<a href="percona-orchestrator-3.2.6-2.el8.x86_64.rpm">percona-orchestrator-3.2.6-2.el8.x86_64.rpm</a>        16-Feb-2022 15:57             5222976
</pre><hr></body>
</html>

```

19. Тестируем репозиторий, добавим его в /etc/yum.repos.d:

```

[root@test-rpm ~]# cat >> /etc/yum.repos.d/otus.repo << EOF
> [otus]
> name=otus-linux
> baseurl=http://localhost/repo
> gpgcheck=0
> enabled=1
> EOF

```

20. Убедимся, что репозиторий подключился и посмотрим, что в нем есть

```

[root@test-rpm ~]# yum repolist enabled | grep otus
Failed to set locale, defaulting to C.UTF-8
otus                            otus-linux
[root@test-rpm ~]#
[root@test-rpm ~]#
[root@test-rpm ~]# yum list | grep otus
Failed to set locale, defaulting to C.UTF-8
otus-linux                                      345 kB/s | 2.8 kB     00:00

percona-orchestrator.x86_64                            2:3.2.6-2.el8                          otus

```

21. Так как NGINX у нас уже стоит, установим репозиторий percona-release:

```

[root@test-rpm ~]#  yum install percona-orchestrator.x86_64 -y
Failed to set locale, defaulting to C.UTF-8
Last metadata expiration check: 0:00:11 ago on Wed May 24 22:13:14 2023.
Dependencies resolved.
===============================================================================================================================================================================
 Package                                           Architecture                        Version                                    Repository                              Size
===============================================================================================================================================================================
Installing:
 percona-orchestrator                              x86_64                              2:3.2.6-2.el8                              otus                                   5.0 M
Installing dependencies:
 jq                                                x86_64                              1.5-12.el8                                 appstream                              161 k
 oniguruma                                         x86_64                              6.8.2-2.el8                                appstream                              187 k

Transaction Summary
===============================================================================================================================================================================
Install  3 Packages

Total download size: 5.3 M
Installed size: 17 M
Downloading Packages:
(1/3): percona-orchestrator-3.2.6-2.el8.x86_64.rpm                                                                                              74 MB/s | 5.0 MB     00:00
(2/3): jq-1.5-12.el8.x86_64.rpm                                                                                                                300 kB/s | 161 kB     00:00
(3/3): oniguruma-6.8.2-2.el8.x86_64.rpm                                                                                                        335 kB/s | 187 kB     00:00
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                          9.4 MB/s | 5.3 MB     00:00
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                                                                       1/1
  Installing       : oniguruma-6.8.2-2.el8.x86_64                                                                                                                          1/3
  Running scriptlet: oniguruma-6.8.2-2.el8.x86_64                                                                                                                          1/3
  Installing       : jq-1.5-12.el8.x86_64                                                                                                                                  2/3
  Installing       : percona-orchestrator-2:3.2.6-2.el8.x86_64                                                                                                             3/3
  Running scriptlet: percona-orchestrator-2:3.2.6-2.el8.x86_64                                                                                                             3/3
  Verifying        : jq-1.5-12.el8.x86_64                                                                                                                                  1/3
  Verifying        : oniguruma-6.8.2-2.el8.x86_64                                                                                                                          2/3
  Verifying        : percona-orchestrator-2:3.2.6-2.el8.x86_64                                                                                                             3/3

Installed:
  jq-1.5-12.el8.x86_64                            oniguruma-6.8.2-2.el8.x86_64                            percona-orchestrator-2:3.2.6-2.el8.x86_64

Complete!

```

 
 















# dz-6
