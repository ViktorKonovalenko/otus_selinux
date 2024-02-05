<h2>Часть 1</h2>
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


<h2>2 часть ДЗ</h2>
Данную задачу можно решить двумя способами, это правкой SELinux отталкиваясь от конфигов и сохраняя строй ФС, либо перемещением файла DNS-зоны, и изменением конфига named.conf, без изменения SELinux. Выбирая из двух вариантов, я выбиру второй, потому что он прозрачен, хоть и неочевиден, всегда можно посмотреть что где лежит, и будет легче объяснить настройку другому специалисту, чем изменение контекстов SELinux. Выполнение его я и продемонстрирую<br>
Подключаемся на клиент и пробуем добавить зону. Видим, что возникает ошибка<br>
![image](https://github.com/ViktorKonovalenko/otus_selinux/assets/32430041/ec0cd76a-6aa6-4f24-b48a-fb6f9f4f6ae0)
Подключаемся к клиенту и проверяем с помощью audit2why логи <br>
<pre>cat /var/log/audit/audit.log | audit2why</pre>
![image](https://github.com/ViktorKonovalenko/otus_selinux/assets/32430041/673d544f-30f1-4002-a52a-f340a0834746)<br>
Исходя из логов видим разные права на файл зоны.<br>
Проверяем файл конфига зон 
<pre>cat /etc/named.conf</pre>
![image](https://github.com/ViktorKonovalenko/otus_selinux/assets/32430041/6975bda7-a3f6-4bc3-a1da-9daa2180ff47)<br>
Видим расположение файла и проверим его контекст<br>
![image](https://github.com/ViktorKonovalenko/otus_selinux/assets/32430041/3b16929d-7310-4305-8015-a3da62951af3)<br>
Видим что тип контекста несоответствует. Посмотрим контексты для каталога<br>
![image](https://github.com/ViktorKonovalenko/otus_selinux/assets/32430041/e2bd1f72-c962-4d75-89c1-2d9cb6e76951)<br>
Из вывода видим что нужный нам контекст настроен на пути /var/named/dynamic/, переносим нужный журнал зоны в настроеный каталог меняя пользователя на named чтобы bind мог с ним работать, а также редактируем named.conf<br>
![image](https://github.com/ViktorKonovalenko/otus_selinux/assets/32430041/d63e36fe-3d3d-48d5-839b-70dfdd5e1d1e)<br>
![image](https://github.com/ViktorKonovalenko/otus_selinux/assets/32430041/efc4021d-1012-4cbd-a7ab-ee0b70c83e2e)<br>
Перезапускаем службу<br>
<pre>systemctl restart named</pre>
Проверяем изменения контекста<br>
![image](https://github.com/ViktorKonovalenko/otus_selinux/assets/32430041/c774caab-f3e3-46d4-a5b6-bbb1899505bd)<br>
Пробуем добавить зону на клиенте
![image](https://github.com/ViktorKonovalenko/otus_selinux/assets/32430041/c892ec36-6a47-4228-bbd9-193e5d02d3f8)<br>
Правила применились<br>
Проверяем еще раз 
![image](https://github.com/ViktorKonovalenko/otus_selinux/assets/32430041/8b74481f-5b9a-4129-9b87-2ce3ac38f419)<br>
Работает!




