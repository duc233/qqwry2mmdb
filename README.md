混合了(qqwry-纯真 IP库CN段)和(MaxMind Geoip2.mmdb)的IP数据库。也就是：提取了qqwry的CN段覆盖MaxMind的IP库。最终格式为MaxMind的*.mmdb
因为感觉qqwry的海外IP不准，而MaxMind的中国IP不准。才这么搞。(再之前因为折腾NexusPT站，新版本网站用的是mmdb识别IP地址。)

源ip库文件：
qqwary.dat，2024年4月24日版。MD5: c3ac7c7606bdfd83f31907f060b7b607
GeoLite2-City.mmdb 2024年5月01日版 MD5: b4e66cdb59dcb1297ffea4e8582e3540
GeoLite2-Country.mmdb 2024年5月01日版 MD5: b72b6db6d4b10ddec23daccd06145f90


制作步骤：本人不会代码，全用工具搞的。
大纲，转换步骤：qqwry.dat>qqwry.mmdb>qqwry.json>qqwry-CN.json>qqwry-CN.json+GeoLite2.mmdb=Geo+qqwryCN.mmdb

1.qqwry转mmdb，工具qqwry2mmdb，https://github.com/leolovenet/qqwry2mmdb 。运行这个我用的Linux环境
  由于MaxMind官方软件更新，新版本不支持（MaxMind::DB::Writer::Tree）指令。需要安装旧版软件。
  需要按照qqwry2mmdb的安装再执行以下命令，安装旧版Writer。
  cpanm http://www.cpan.org/authors/id/M/MA/MAXMIND/MaxMind-DB-Writer-0.300003.tar.gz
  得到qqwry.mmdb
  
2.qqwry转json，我用的win环境。工具mmdbinspect  https://github.com/maxmind/mmdbinspect 。运行这个我用的Win10环境
  mmdbinspect.exe -db qqwry.mmdb 0.0.0.0/0 >> qqwry.json

3.qqwry.json合并入GeoLite2.mmdb，工具mmedit https://github.com/iglov/mmdb-editor。运行这个我用的Win10环境

  首先需要检查一下qqwry.json是否符合工具识别的json格式，qqwry.json应该需要修改一些：
  运行下行命令，json不符合的地方，mmedit会给出提示。参照mmedit主页的mmdb-editor/testdata/dataset.json，的标准格式，进行修改qqwry.json。
  
  mmdb-editor-windows-amd64.exe -d qqwry.json -i GeoLite2.mmdb -o merge.mmdb
  
  3.>提取中国IP，我用的UltraEditor和EmEditor。
  
     3.1> UltraEditor通配符转换替换，制作成每个json段为一行。记得标记回车符，例如\n替换为RRR233，这样做为了之后恢复文件成json。
     
     3.2> 再用EmEditor使用通配符，查找+提取为新文件。下行为通配符
     .*北京.*|.*天津.*|.*河北.*|.*山西.*|.*内蒙古.*|.*辽宁.*|.*吉林.*|.*黑龙江.*|.*上海.*|.*江苏.*|.*浙江.*|.*安徽.*|.*福建.*|.*江西.*|.*山东.*|.*河南.*|.*湖北.*|.*湖南.*|.*广东.*|.*广西.*|.*海南.*|.*重庆.*|.*四川.*|.*贵州.*|.*云南.*|.*西藏.*|.*陕西.*|.*甘肃.*|.*青海.*|.*宁夏.*|.*新疆.*|.*中国.*
     
     3.3> 恢复为json。查找替换标记的回车符RRR233为\n。得到qqwryCN.json

  4.进行最后的合并，使用工具mmedit。
     1>以GeoIp的城市版，做基底，就用GeoLite2-City.mmdb。这样外国ip会精确到城市+国内的qqwry是城市ip段。
        mmdb-editor-windows-amd64.exe -d qqwryCN.json -i GeoLite2-City.mmdb -o mergeCity.mmdb
   
     2>以GeoIp的国家版，做基底，就用GeoLite2-Country.mmdb。这样外国ip会精确到国家+国内的qqwry是城市ip段。
       mmdb-editor-windows-amd64.exe -d qqwryCN.json -i GeoLite2-Country.mmdb -o mergeCountry.mmdb

完成。


 
 
 
-----------以下是PT Nexus的mmdb识别问题，与主题ip库合并无关-----------
















# qqwry2mmdb
 为抓包软件 Wireshark 能使用纯真网络 IP 数据库而提供的格式转换工具

![WireShark 集成纯真网络IP数据库](./wireshark-with-qqwry-ip-data.jpg)

## 特性

* 支持自定义 IP 数据库记录，例如向库中增加私网 IP 的归属信息。编辑 `qqwry2mmdb.pl`，打开 `remove_reserved_networks` 开关，新增自己的私人记录。

```perl
$tree->insert_range("192.168.1.1", "192.168.1.1", {
        city    => {
            names => {
                en => "众里寻她千百度，蓦然回首阑珊处"
            }
        },
        country => {
            names => {
                en => "就是您"
            }
        }
    });
```

* 支持根据 IP 属地的数据包过滤， 例如 `ip.geoip.dst_city contains "辽宁"`

![根据IP地址属地过滤](./wireshark-filter.jpg)

## 描述

[Wireshark](https://www.wireshark.org/) 是网络抓包、协议分析、网络问题定位的"瑞士军刀"，功能非常强大，我本人非常喜欢的开源网络工具。在网络分析行为中 IP 地址归属地信息查询也是一个很重要的方面。在国内非常出名的，以及维护时间最久的IP归属地数据库应该非 [纯真网络IP数据库](http://www.cz88.net/ip/) 莫属了。至少对于我而言，我希望在 Wireshark 的程序界面里能同时看到捕获到的IP数据包的归属地信息，因此我研究了一下 Wireshark 的源代码，发现它本身已经基于 [libmaxminddb](https://github.com/maxmind/libmaxminddb) 库实现了这个功能（再次为它的强大点赞👍），只不过默认不携带具体的 IP 归属地数据库，需要自行安装。另外 [MaxMind DB](https://github.com/maxmind/MaxMind-DB/blob/master/MaxMind-DB-spec.md) 为开源格式的数据库，因此剩下的工作就简单了，只需要把纯真IP数据库的格式转换到 mmdb 格式，并配置 Wireshark 读取就大功告成了。


## 安装 qqwry.mmdb 数据库, 配置 Wireshark

配置 Wireshark 指定包含 qqwry.mmdb 的目录(⚠️此处是需要指明包含数据库文件的目录，不是数据库文件的绝对路径⚠️)

![WireShark 配置使用纯真网络IP数据库](./wireshark-config.jpg)

点击OK以后，重启 wireshark 抓包，你会发现在 “Packet List” 面版，看不到我上面截图中的那些中文IP信息，还需要进行下面的操作

![WireShark apply GeoIP As column](./wireshark-config-as-column.jpg)
## 安装 qqwry2mmdb.pl，自己生成特定数据库文件

MaxMind DB 自家的 [数据库生成工具](https://github.com/maxmind/MaxMind-DB-Writer-perl) 是基于 Perl 语言编写，因此本项目也是用 Perl 来实现(本想为了便携性用Go重写，但是看了看工作量又懒的再造轮子了)，使用并扩展了 [IP::QQWry](https://metacpan.org/pod/IP::QQWry) 库来读取纯真IP库的所有记录。

首先安装 [Perl](https://www.perl.org/get.html) ，然后安装 [cpanm](https://cpanmin.us/), 以及依赖库，`MaxMind::DB::Writer` 与 `IP::QQWry::Decoded`

### Linux (CentOS7)

```shell
yum -y install perl perl-CPAN

#使用 cpan 安装 cpanm 命令
cpan App::cpanminus
#或者, 使用 curl 安装 cpanm 命令
#curl -L https://cpanmin.us | perl - App::cpanminus

#使用 cpanm 命令安装所需模块
cpanm --notest --force MaxMind::DB::Writer IP::QQWry::Decoded

#下载安装 qqwry2mmdb
wget https://github.com/leolovenet/qqwry2mmdb/archive/refs/heads/master.zip
unzip master.zip
cd qqwry2mmdb-master

#请自行下载最新版本的 qqwry.dat
#请注意，qqwry2mmdb.pl 脚本依赖本项目中的 lib/IP/QQWry/Dumper.pm 库文件
perl qqwry2mmdb.pl /your/save/path/of/qqwry.dat
```

### macOS

```shell
brew install cpanm
cpanm --notest MaxMind::DB::Writer IP::QQWry::Decoded

#下载安装 qqwry2mmdb
wget https://github.com/leolovenet/qqwry2mmdb/archive/refs/heads/master.zip
unzip master.zip
cd qqwry2mmdb-master

#请自行下载最新版本的 qqwry.dat
#请注意，qqwry2mmdb.pl 脚本依赖本项目中的 lib/IP/QQWry/Dumper.pm 库文件
perl qqwry2mmdb.pl /your/save/path/of/qqwry.dat
```

### windows

`MaxMind::DB::Writer` 不支持该平台
