# SELinux


   1. Запустить nginx на нестандартном порту 3-мя разными способами:
        Необходимо изменить в конфиге /etc/nginx/nginx.conf порт на нестандартный, например 1111
        
        Стандартные порты: 
        http_port_t                    tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000

        1.**переключатели setsebool;**        
        

        2.**добавление нестандартного порта в имеющийся тип;**
         
            semanage port -a -t http_port_t -p tcp 1111
         
        После добавления, проверяем , добавился ли порт 
            
            semanage port -l | grep http_port_t
            http_port_t                    tcp      1111, 80, 81, 443, 488, 8008, 8009, 8443, 9000

        Рестартим nginx
        
            systecmctl restart nginx.service
        Проверка:
        ![Screenshot 2021-09-06 230254](https://i.imgur.com/OxgASOm.png)

        3.**формирование и установка модуля SELinux.**



/var/log/audit/audit.log



1. развернуть приложенный стенд https://github.com/mbfx/otus-linux-adm/tree/master/selinux_dns_problems  

**выяснить причину неработоспособности механизма обновления зоны**


