### supervisor配置
```
[program:lfc-worker]
process_name=%(program_name)s_%(process_num)01d
command=php /mnt/www/html/lfc/artisan queue:work redis --sleep=3 --tries=3
autostart=true
autorestart=true
user=www
numprocs=1
redirect_stderr=true
stdout_logfile=/var/log/laravel_lfc_queue_worker.log
```
更新新的配置到supervisord
```
supervisorctl update
```
重新启动配置中的所有程序
```
supervisorctl reload
```