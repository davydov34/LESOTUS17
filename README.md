# LESOTUS17
## 1. Запуск nginx на нестандартном порту тремя разными способами...

- 1.1.1 Проверяем состояние firewalld:
> [vagrant@selinux ~]$ systemctl status firewalld.service  
> ● firewalld.service - firewalld - dynamic firewall daemon  
>    Loaded: loaded (/usr/lib/systemd/system/firewalld.service; disabled; vendor preset: enabled)  
>     Active: inactive (dead)  
>       Docs: man:firewalld(1)  

- 1.1.2 Проверяем конфигурацию Nginx:
> [root@selinux ~]# nginx -t  
> nginx: the configuration file /etc/nginx/nginx.conf syntax is ok  
> nginx: configuration file /etc/nginx/nginx.conf test is successful

- 1.1.3 Проверяем режиv работы SELinux:
> [root@selinux ~]# getenforce  
> Enforcing

- 1.1.4 Анализируем лог audit.log при помощи audit2why:
> [root@selinux ~]# cat /var/log/audit/audit.log  | audit2why  
> type=AVC msg=audit(1694284351.294:881): avc:  denied  { name_bind } for  pid=3035 comm="nginx" src=4881 scontext=system_u:system_r:httpd_t:s0 tcontext=system_u:object_r:unreserved_port_t:s0 tclass=tcp_socket permissive=0  
>  
> 	Was caused by:  
> 	The boolean nis_enabled was set incorrectly.  
> 	Description:  
> 	Allow nis to enabled  
>  
> 	Allow access by executing:  
> 	**#setsebool -P nis_enabled 1**

- 1.1.5 Включаем параметр nis_enabled и пробуем запустить сервис:
> [root@selinux ~]# setsebool -P nis_enabled on  
> [root@selinux ~]# systemctl start nginx  
> [root@selinux ~]# systemctl status nginx.service  
> ● nginx.service - The nginx HTTP and reverse proxy server  
>    Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)  
>    Active: active (running) since Sat 2023-09-09 18:57:03 UTC; 9s ago  
  Process: 3284 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)  
>   Process: 3282 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)  
>   Process: 3281 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)  
> >  Main PID: 3286 (nginx)  
>   
> CGroup: /system.slice/nginx.service  
>            ├─3286 nginx: master process /usr/sbin/nginx  
>            └─3288 nginx: worker process  
>  
> Sep 09 18:57:03 selinux systemd[1]: Starting The nginx HTTP and reverse proxy server...  
> Sep 09 18:57:03 selinux nginx[3282]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok  
> Sep 09 18:57:03 selinux nginx[3282]: nginx: configuration file /etc/nginx/nginx.conf test is successful  
> Sep 09 18:57:03 selinux systemd[1]: Started The nginx HTTP and reverse proxy server.  

- Nginx успешно запущен. Возвращаем nis_enabled в состояние off и переходим ко второму способу.

- 1.2 Запуск NGINX посредством добавления нестандартного порта в уже имеющиеся типы:
- 1.2.1 Ищем порты, сопоставленные с http:
> [root@selinux ~]# semanage port -l | grep http  
> http_cache_port_t              tcp      8080, 8118, 8123, 10001-10010  
> http_cache_port_t              udp      3130  
> http_port_t                    tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000  
> pegasus_http_port_t            tcp      5988  
> pegasus_https_port_t           tcp      5989  

- 1.2.2 Добавляем порт 4881 к типу http_port_t:
> [root@selinux ~]# semanage port -a -t http_port_t -p tcp 4881  
> [root@selinux ~]# semanage port -l | grep http  
> http_cache_port_t              tcp      8080, 8118, 8123, 10001-10010  
> http_cache_port_t              udp      3130  
> http_port_t                    tcp      4881, 80, 81, 443, 488, 8008, 8009, 8443, 9000  
> pegasus_http_port_t            tcp      5988  
> pegasus_https_port_t           tcp      5989

- 1.2.3 Выполняем запуск NGINX:
> [root@selinux ~]# systemctl start nginx  
> [root@selinux ~]# systemctl status nginx  
> ● nginx.service - The nginx HTTP and reverse proxy server  
>    Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)  
   Active: active (running) since Sat 2023-09-09 19:12:08 UTC; 5s ago  
>   Process: 3603 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)  
>   Process: 3601 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)  
>   Process: 3600 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)  
>  Main PID: 3605 (nginx)  
>    CGroup: /system.slice/nginx.service  
>            ├─3605 nginx: master process /usr/sbin/nginx  
>            └─3607 nginx: worker process  
>   
Sep 09 19:12:08 selinux systemd[1]: Starting The nginx HTTP and reverse proxy server...  
Sep 09 19:12:08 selinux nginx[3601]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok  
Sep 09 19:12:08 selinux nginx[3601]: nginx: configuration file /etc/nginx/nginx.conf test is successful  
Sep 09 19:12:08 selinux systemd[1]: Started The nginx HTTP and reverse proxy server.
