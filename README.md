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
> 
