Запускаем Vagrantfile, который создает ВМ с установленным nginx на порту 4881<br>
1) Проверяем запущен ли nginx<br>
 ![image](https://github.com/ViktorKonovalenko/otus_selinux/assets/32430041/f6437842-b095-4d50-9922-70ac2fb38e17)
видим что nginx в ошибке<br>
2) Проверяем работу Selinux
![image](https://github.com/ViktorKonovalenko/otus_selinux/assets/32430041/91b19115-942c-464e-895d-5e1d935ef2e2)
Selinux запущен в режиме блокировки запрещенных активностей<br>
3) Проверим слушается ли в системе порт 4881<br>
[root@selinux vagrant]# netstat -tulnp | grep 4881<br>
[root@selinux vagrant]#<br>
Данный порт не прослушивается<br>
4) Делаем grep /var/log/audit/audit.log на 4881 порт<br>
   ![image](https://github.com/ViktorKonovalenko/otus_selinux/assets/32430041/a7a5abc5-c167-41d4-8718-a0aa6aa0017c)
5)Проверяем с помощью утилиты audit2why почему блокируется трафик<br>
![image](https://github.com/ViktorKonovalenko/otus_selinux/assets/32430041/abc4aa1c-d99f-445f-8e71-becbf424fae6)
Видим, что нужно поменять параметр nis_enabled.<br>
6) Включаем параметр nis_enabled и перезапускаем nginx<br>
![image](https://github.com/ViktorKonovalenko/otus_selinux/assets/32430041/922ec1be-0d68-4695-b635-cfc4a1bc2129)
7) Делаем curl странички<br>
![image](https://github.com/ViktorKonovalenko/otus_selinux/assets/32430041/fb77e116-ade8-47ed-842e-cf2bee98dc04)
8) Проверяем параметр nis_enabled с помощью команды:<br>
[root@selinux vagrant]# getsebool -a | grep nis_enabled<br>
nis_enabled --> on<br>
9) Отключим данный параметр и попробуем второй вариант включения порта в Selinux<br>
[root@selinux vagrant]# setsebool -P nis_enabled off<br>
[root@selinux vagrant]# getsebool -a | grep nis_enabled<br>
nis_enabled --> off<br>
10) Просмотрим типы http трафика<br>
[root@selinux vagrant]# semanage port -l | grep http<br>
http_cache_port_t              tcp      8080, 8118, 8123, 10001-10010<br>
http_cache_port_t              udp      3130<br>
http_port_t                    tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000<br>
pegasus_http_port_t            tcp      5988<br>
pegasus_https_port_t           tcp      5989<br>
Видим что порта 4881 нет<br>
11) Добавляем порт 4881 <br>
[root@selinux vagrant]# semanage port -l | grep http<br>
http_cache_port_t              tcp      8080, 8118, 8123, 10001-10010<br>
http_cache_port_t              udp      3130<br>
http_port_t                    tcp      4881, 80, 81, 443, 488, 8008, 8009, 8443, 9000<br>
pegasus_http_port_t            tcp      5988<br>
pegasus_https_port_t           tcp      5989<br>
Порт 4881 стал доступен<br>
12) Перезагружаем nginx и видим статус active<br>
[root@selinux vagrant]# systemctl restart nginx<br>
[root@selinux vagrant]# systemctl status nginx<br>
● nginx.service - The nginx HTTP and reverse proxy server<br>
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)<br>
   Active: active (running) since Sat 2024-01-20 09:31:33 UTC; 15s ago<br>
  Process: 5337 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)<br>
  Process: 5334 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)<br>
  Process: 5332 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)<br>
 Main PID: 5340 (nginx)<br>
   CGroup: /system.slice/nginx.service<br>
           ├─5340 nginx: master process /usr/sbin/nginx<br>
           └─5341 nginx: worker process<br>

Jan 20 09:31:32 selinux systemd[1]: Starting The nginx HTTP and reverse proxy server...<br>
Jan 20 09:31:33 selinux nginx[5334]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok<br>
Jan 20 09:31:33 selinux nginx[5334]: nginx: configuration file /etc/nginx/nginx.conf test is successful<br>
Jan 20 09:31:33 selinux systemd[1]: Started The nginx HTTP and reverse proxy server.<br>
13) Удаляем порт<br>
[root@selinux vagrant]# semanage port -d -t http_port_t -p tcp 4881<br>
[root@selinux vagrant]# semanage port -l | grep http<br>
http_cache_port_t              tcp      8080, 8118, 8123, 10001-10010<br>
http_cache_port_t              udp      3130<br>
http_port_t                    tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000<br>
pegasus_http_port_t            tcp      5988<br>
pegasus_https_port_t           tcp      5989<br>
14) С помощью утилиты audit2allow чтобы сделать модуль на основе логов для Selinux, который разрешит работу nginx на нестандартном порту<br>
![image](https://github.com/ViktorKonovalenko/otus_selinux/assets/32430041/8cededdd-a510-48a8-8f21-da279ba7289e)
15) Модуль сообщил команду которую необходимо запустить для открытия порта <br>
[root@selinux vagrant]# semodule -i nginx.pp<br>
[root@selinux vagrant]#<br>
16) Проверяем работу nginx<br>
![image](https://github.com/ViktorKonovalenko/otus_selinux/assets/32430041/3ae69b94-8772-47f0-9d14-771fa2cf26b2)
Все работает!
