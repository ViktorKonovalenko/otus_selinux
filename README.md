Запускаем Vagrantfile, который создает ВМ с установленным nginx на порту 4881<br>
1) Проверяем запущен ли nginx
 ![image](https://github.com/ViktorKonovalenko/otus_selinux/assets/32430041/f6437842-b095-4d50-9922-70ac2fb38e17)
видим что nginx в ошибке
2) Проверяем работу Selinux
![image](https://github.com/ViktorKonovalenko/otus_selinux/assets/32430041/91b19115-942c-464e-895d-5e1d935ef2e2)
Selinux запущен в режиме блокировки запрещенных активностей
3) Проверим слушается ли в системе порт 4881
[root@selinux vagrant]# netstat -tulnp | grep 4881
[root@selinux vagrant]#
Данный порт не прослушивается
4) Делаем grep /var/log/audit/audit.log на 4881 порт
   ![image](https://github.com/ViktorKonovalenko/otus_selinux/assets/32430041/a7a5abc5-c167-41d4-8718-a0aa6aa0017c)

6)Проверяем с помощью утилиты audit2why почему блокируется трафик
![image](https://github.com/ViktorKonovalenko/otus_selinux/assets/32430041/abc4aa1c-d99f-445f-8e71-becbf424fae6)
Видим, что нужно поменять параметр nis_enabled.
7)
8) Включаем параметр nis_enabled и перезапускаем nginx
![image](https://github.com/ViktorKonovalenko/otus_selinux/assets/32430041/922ec1be-0d68-4695-b635-cfc4a1bc2129)
9) Делаем curl странички
![image](https://github.com/ViktorKonovalenko/otus_selinux/assets/32430041/fb77e116-ade8-47ed-842e-cf2bee98dc04)
10) Проверяем параметр nis_enabled с помощью команды:
[root@selinux vagrant]# getsebool -a | grep nis_enabled
nis_enabled --> on
11) Отключим данный параметр и попробуем второй вариант включения порта в Selinux
[root@selinux vagrant]# setsebool -P nis_enabled off
[root@selinux vagrant]# getsebool -a | grep nis_enabled
nis_enabled --> off
12) Просмотрим типы http трафика
[root@selinux vagrant]# semanage port -l | grep http
http_cache_port_t              tcp      8080, 8118, 8123, 10001-10010
http_cache_port_t              udp      3130
http_port_t                    tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000
pegasus_http_port_t            tcp      5988
pegasus_https_port_t           tcp      5989
Видим что порта 4881 нет
13) Добавляем порт 4881 
[root@selinux vagrant]# semanage port -l | grep http
http_cache_port_t              tcp      8080, 8118, 8123, 10001-10010
http_cache_port_t              udp      3130
http_port_t                    tcp      4881, 80, 81, 443, 488, 8008, 8009, 8443, 9000
pegasus_http_port_t            tcp      5988
pegasus_https_port_t           tcp      5989
Порт 4881 стал доступен
14) Перезагружаем nginx и видим статус active
[root@selinux vagrant]# systemctl restart nginx
[root@selinux vagrant]# systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: active (running) since Sat 2024-01-20 09:31:33 UTC; 15s ago
  Process: 5337 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
  Process: 5334 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
  Process: 5332 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
 Main PID: 5340 (nginx)
   CGroup: /system.slice/nginx.service
           ├─5340 nginx: master process /usr/sbin/nginx
           └─5341 nginx: worker process

Jan 20 09:31:32 selinux systemd[1]: Starting The nginx HTTP and reverse proxy server...
Jan 20 09:31:33 selinux nginx[5334]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Jan 20 09:31:33 selinux nginx[5334]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Jan 20 09:31:33 selinux systemd[1]: Started The nginx HTTP and reverse proxy server.
15) Удаляем порт
[root@selinux vagrant]# semanage port -d -t http_port_t -p tcp 4881
[root@selinux vagrant]# semanage port -l | grep http
http_cache_port_t              tcp      8080, 8118, 8123, 10001-10010
http_cache_port_t              udp      3130
http_port_t                    tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000
pegasus_http_port_t            tcp      5988
pegasus_https_port_t           tcp      5989
16) С помощью утилиты audit2allow чтобы сделать модуль на основе логов для Selinux, который разрешит работу nginx на нестандартном порту
![image](https://github.com/ViktorKonovalenko/otus_selinux/assets/32430041/8cededdd-a510-48a8-8f21-da279ba7289e)
17) Модуль сообщил команду которую необходимо запустить для открытия порта 
[root@selinux vagrant]# semodule -i nginx.pp
[root@selinux vagrant]#
18) Проверяем работу nginx
![image](https://github.com/ViktorKonovalenko/otus_selinux/assets/32430041/3ae69b94-8772-47f0-9d14-771fa2cf26b2)
Все работает!
