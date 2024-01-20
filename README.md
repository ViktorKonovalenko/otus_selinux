Запускаем Vagrantfile, который создает ВМ с установленным nginx на порту 4881<br>
1) Проверяем запущен ли nginx<br>
 ![image](https://github.com/ViktorKonovalenko/otus_selinux/assets/32430041/f6437842-b095-4d50-9922-70ac2fb38e17)
видим что nginx в ошибке<br>
2) Проверяем работу Selinux<br>
![image](https://github.com/ViktorKonovalenko/otus_selinux/assets/32430041/91b19115-942c-464e-895d-5e1d935ef2e2)<br>
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
![image](https://github.com/ViktorKonovalenko/otus_selinux/assets/32430041/4e7125bc-7359-4146-8f69-43ba1c97efcb)<br>
9) Отключим данный параметр и попробуем второй вариант включения порта в Selinux<br>
![image](https://github.com/ViktorKonovalenko/otus_selinux/assets/32430041/9d9f6d82-08d2-4ba0-b371-0598446ac9e1)<br>
10) Просмотрим типы http трафика<br>
![image](https://github.com/ViktorKonovalenko/otus_selinux/assets/32430041/6f9b997d-ffc5-47f5-af9a-32ef1223e01c)<br>
Видим что порта 4881 нет<br>
11) Добавляем порт 4881 <br>
![image](https://github.com/ViktorKonovalenko/otus_selinux/assets/32430041/c19e966e-79a9-443d-bd17-ee8c96ff52e5)<br>
Порт 4881 стал доступен<br>
12) Перезагружаем nginx и видим статус active<br>
![image](https://github.com/ViktorKonovalenko/otus_selinux/assets/32430041/8841bd1b-4058-4f0a-a10a-0b06b5f98aca)<br>
13) Удаляем порт<br>
![image](https://github.com/ViktorKonovalenko/otus_selinux/assets/32430041/9c16644f-5b06-48be-8a05-4ebd8225a1af)<br>
14) С помощью утилиты audit2allow чтобы сделать модуль на основе логов для Selinux, который разрешит работу nginx на нестандартном порту<br>
![image](https://github.com/ViktorKonovalenko/otus_selinux/assets/32430041/8cededdd-a510-48a8-8f21-da279ba7289e)<br>
15) Модуль сообщил команду которую необходимо запустить для открытия порта <br>
[root@selinux vagrant]# semodule -i nginx.pp<br>
[root@selinux vagrant]#<br>
16) Проверяем работу nginx<br>
![image](https://github.com/ViktorKonovalenko/otus_selinux/assets/32430041/3ae69b94-8772-47f0-9d14-771fa2cf26b2)
Все работает!
