# SELinux


   1. Запустить nginx на нестандартном порту 3-мя разными способами:
        Необходимо изменить в конфиге /etc/nginx/nginx.conf порт на нестандартный, например 1111
        
        Стандартные порты: 
        http_port_t                    tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000



        1.**переключатели setsebool;**        
        
        Можно посмотреть рекомендации из вывода команды 

            cat  /var/log/audit/audit.log | audit2why

        Выполняем команду 
            
            setsebool -P nis_enabled 1

        Изменение применилось 

        ![Screenshot 2021-09-06 233837](https://i.imgur.com/jSLNXxV.png)

        Сервис nginx запустился

        ![Screenshot 2021-09-06 233937](https://i.imgur.com/zNH5pnM.png)

        Отмена изменений: setsebool -P nis_enabled 0

        2.**добавление нестандартного порта в имеющийся тип;**
         
            semanage port -a -t http_port_t -p tcp 1111
         
        После добавления, проверяем , добавился ли порт 
            
            semanage port -l | grep http_port_t
            http_port_t                    tcp      1111, 80, 81, 443, 488, 8008, 8009, 8443, 9000

        Рестартим nginx
        
            systecmctl restart nginx.service

        Проверка:
        ![Screenshot 2021-09-06 230254](https://i.imgur.com/OxgASOm.png)

        Отмена изменений:  semanage port -d -t http_port_t -p tcp 1111


        3.**формирование и установка модуля SELinux.**

        Копируем из лога /var/log/audit/audit.log последнее сообщение в файл log_nginx

        Формируем модуль с правилами  

            audit2allow -M httpd_allow_port --debug < log_nginx

        Загружаем модуль 
        
            semodule -i httpd_allow_port.pp

        Проверка:
        ![Screenshot 2021-09-06 235136](https://i.imgur.com/53SmBel.png)





1. Развернуть приложенный стенд https://github.com/mbfx/otus-linux-adm/tree/master/selinux_dns_problems и выяснить причину неработоспособности механизма обновления зоны

При попытке удаленно (с рабочей станции) внести изменения в зону ddns.lab происходит следующее:

    [vagrant@client ~]$ nsupdate -k /etc/named.zonetransfer.key
    > server 192.168.50.10
    > zone ddns.lab
    > update add www.ddns.lab. 60 A 192.168.50.15
    > send
    update failed: SERVFAIL

Для выяснения причины сначала выполняем на сервере 

    cat  /var/log/audit/audit.log | audit2why

![Screenshot 2021-09-07 231917](https://i.imgur.com/T4Vawpq.png)

Из вывода видно, что проблема с named.ddns.lab.view1.jnl
Также смотрим статус named.service
![Screenshot 2021-09-08 232831](https://i.imgur.com/OsLthjK.png)

Проблема с созданием файла в директории /etc/named/dynamic

Смотрим контекст для директории

    ls -Z /etc/named/dynamic/
![Screenshot 2021-09-08 232904](https://i.loli.net/2021/09/10/3DS9ztpLQFVBjdc.png)
Установлен контекст etc_t. 
По документу https://www.systutorials.com/docs/linux/man/8-bind_selinux/, контекст для папки /etc/named/dynamic  должен быть named_cache_t ( по доке размещается в другой директории )

Изменяем контекст на нужный 

    chcon -t named_cache_t -R /etc/named/dynamic/

Проверяем изменение зоны на клиенте 

![Screenshot 2021-09-09 184713](https://i.imgur.com/9j1mrOQ.png)

Проверяем статус named.service

![Screenshot 2021-09-09 212507](https://i.imgur.com/ie3GcK2.png)

Зона успешно обновилась

Для сохранения изменений после перезагрузки необходимо выполнить команду 

     semanage fcontext -a -t named_cache_t "/etc/named/dynamic(/.*)?"



Полезные команды: 

- sesearch
- seinfo
- findcon
- getsebool
- setsebool
- audit2allow
- audit2why

<<<<<<< HEAD
Контексты /etc/selinux/targeted/contexts/files
информацию о правах пользователей
semanage login -l
ps -Z 6798
sesearch -A -s httpd_t | grep 'allow httpd_t' разрешающих правил для типа httpd_t
/etc/selinux/config  Конфигурация SELinux:
sestatus или getenforce
setenforce 0
restorecon -v /home/user Восстанавливаем контекст каталога

audit2allow -M httpd_add --debug < /var/log/audit![Screenshot 2021-09-08 232904](undefined)/audit.log![Screenshot 2021-09-08 232904](https://i.imgur.com/RiVcE9a.png)
semodule -i httpd_add.pp
Параметризованные политики SELinux
getsebool -a | grep samba
setsebool -P samba_share_fusefs on
=======
Контексты /etc/selinux/targeted/contexts/files  
информацию о правах пользователей  	
semanage login -l 	
ps -Z 6798	
sesearch -A -s httpd_t | grep 'allow httpd_t' разрешающих правил для типа httpd_t	
/etc/selinux/config  Конфигурация SELinux:	
sestatus или getenforce		
setenforce 0	
restorecon -v /home/user Восстанавливаем контекст каталога	
	
audit2allow -M httpd_add --debug < /var/log/audit/audit.log	
semodule -i httpd_add.pp	
Параметризованные политики SELinux	
getsebool -a | grep samba	
setsebool -P samba_share_fusefs on	
>>>>>>> cf3d4307af4fb8623a827979ae01b211345f7c75
