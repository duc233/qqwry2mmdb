   做成了Docker:
   ```
   docker run -itd -p 11451:11451 --name q2m-ok2 gduc233/qqwry2mmdb
   ```

   ssh-Port:11451 用户/密码:root/114514。转换脚本在目录/home/qqwry2mmdb*下  
   
   ↓↓↓转换命令,得到qqwry.mmdb  
   ```
perl qqwry2mmdb.pl qqwry.dat
```
-- 
-- 
--   
--  
--  
--  


手动安装方法，Ubuntu中：
-------------leolovenet/qqwry2mmdb的环境安装方式----------------
```
apt update
apt install git curl wget unzip gcc make perl libmaxmind-db-writer-perl 
cpan App::cpanminus
cpanm --notest --force IP::QQWry::Decoded
cpanm --notest --force http://www.cpan.org/authors/id/M/MA/MAXMIND/MaxMind-DB-Writer-0.300003.tar.gz

wget https://github.com/leolovenet/qqwry2mmdb/archive/refs/heads/master.zip
unzip master.zip
cd qqwry2mmdb-master
```

由于MaxMind官方软件更新，新版本不支持（MaxMind::DB::Writer::Tree）指令。需要安装旧版软件。
