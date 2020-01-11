#### 1. Laravel SQL执行语句抓取--sql当前执行日志
```
\DB::enableQueryLog();
User::find(1);
dd(\DB::getQueryLog());
```