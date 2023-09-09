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
