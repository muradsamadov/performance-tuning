# performance-tuning
performance tuning basics

# performance tools

https://netflixtechblog.com/linux-performance-analysis-in-60-000-milliseconds-accc10403c55  --  Linux Performance Analysis in 60,000 Milliseconds\
https://github.com/iovisor/bcc/blob/master/docs/tutorial.md  --  bcc-tools-a aid lazim olan tool-lar haqqinda(esas sayt)

```
# vmstat 1  --  It prints a summary of key server statistics on each line.
```
r: Number of processes running on CPU and waiting for a turn.\
si, so: Swap-ins and swap-outs. If these are non-zero, you’re out of memory.
```
# pidstat 1  --  Pidstat is a little like top’s per-process summary, but prints a rolling summary instead of clearing the screen. 

# iostat -xz 1  --  This is a great tool for understanding block devices (disks), both the workload applied and the resulting performance.
```
r/s, w/s, rkB/s, wkB/s: These are the delivered reads, writes, read Kbytes, and write Kbytes per second to the device. Use these for workload characterization. A performance problem may simply be due to an excessive load applied.\
await: The average time for the I/O in milliseconds. This is the time that the application suffers, as it includes both time queued and time being serviced. Larger than expected average times can be an indicator of device saturation, or device problems.\
avgqu-sz: The average number of requests issued to the device. Values greater than 1 can be evidence of saturation (although devices can typically operate on requests in parallel, especially virtual devices which front multiple back-end disks.)\
%util: Device utilization. This is really a busy percent, showing the time each second that the device was doing work. Values greater than 60% typically lead to poor performance (which should be seen in await), although it depends on the device. Values close to 100% usually indicate saturation.
```
# sar -n DEV 1  --  Use this tool to check network interface throughput

# sar -n TCP,ETCP 1
```

# bcc-tools
```
# ./execsnoop  --  cari vaxtda kim sessiyada hansi komandani icra edirse onlari list edir
# ./opensnoop  --  cari vaxtda hansi fayllar istifade olunursa onlari list edir. Misal ucun hal-hazirda nginx.conf faylinda nese nediyisiklikler edirem ve bura dusurki men vim komandasi vasitesile nginx.conf-da edir edirem
# ./xfsslower  --  xfs fayl sisteminin yuklenib-yuklenmemesini gosterir. Meselen hal-hazirda men hdd stress-test qoysam list seklinde gosterecekki bu komanda bu fayla yazir
# ./biolatency  --  diskde olan yuklenmeleri gecikmeleri count seklinde gosterir. Misal ucun eger count sayi coxalir ve azalirsa demekki yuk azalib. count sayi coxdursa demekki gecikmleler var hdd-de
# ./biosnoop  --  ile diskde olan gecikmelere baxa bilersen. Misalcun hansi komanda daha cox yukleyir diski baxmaq olar
# ./cachestat -T  --  ile hal-hazirda sistemde ne qeder cache yuklenir baxmaq olur. Etrafli bu linkde http://www.brendangregg.com/blog/2014-12-31/linux-page-cache-hit-ratio.html
# ./tcpconnect  --  hal-hazirki serverden hansi serverlere hansi portuna connect olursansa onlari list edir
# ./tcpaccept  --  remote serverden local servere gelen port accessleri list edir
# ./runqlat  --  This program summarizes scheduler run queue latency as a histogram, showing how long tasks spent waiting their turn to run on-CPU.
```



# SAR
https://www.computerhope.com/unix/usar.htm 

1.
```
$ sar -u 1 3
Linux 2.6.18-194.el5PAE (dev-db)        03/26/2011      _i686_  (8 CPU)

01:27:32 PM       CPU     %user     %nice   %system   %iowait    %steal     %idle
01:27:33 PM       all      0.00      0.00      0.00      0.00      0.00    100.00
01:27:34 PM       all      0.25      0.00      0.25      0.00      0.00     99.50
01:27:35 PM       all      0.75      0.00      0.25      0.00      0.00     99.00
Average:          all      0.33      0.00      0.17      0.00      0.00     99.50
```
sar -u 1 3  --  CPU Usage of ALL CPUs; 1 saniyeden 1 ve 3 defe. \
%user - user level (application) terefinden olan yuklenmeleri bildirir. Misal ucun hansisa applicationda problem var demekki. \
%nice - eger cpu ve nice ikiside yuklenibse demekki cpu-nu nese yukleyib, cpu yuklenib ve nice yuklenmeyibse yeni arada % ferqi coxdursa demekki nese normal deyil, ya cpu cox stress altinda isleyir ya da ki, hansisa yaranan yeni prosesler gozlemededir.\
%system - system level (kernel) terefinden yuklenmeler var. Daha cox hdd,ram,disk yuklenmeleri bura aid edilir.\
%iowait - diskde olan yuklenmeler bura aid edilir. hdd ni stress test etsek burada %iowait cpu faizinin qalxdigini gore bilerik.\
%steal - eslinde VM ona ayrilan cpu hecminin tam hissesini istifade etmir. Misal ucun %steal-da 25% , %system-de 75% yaziblarsa demekki cpunun %25-i itkilere gedib\
%idle - eger serverde %system 12% ve %idle 88% yuklenibse bu normaldir cunki cpunun 88% i sleep veziyyetindedir ve bu problemli bi sey deyil. Serverde normalda %idle \
100%le isleyir eger serveri yukleyene nese yoxdursa. Bu serverde ola bilerki, hansisa performance meseleleri varki onuda sistem ozu edir server cox yuklu olmayanda.

sar -u  -- ise 1 gun erzinde 10 deq araliginda bize neticeni cixarir. Bu halda serverin evvelki veziyyetini monitorinq etmek olur  \
sar -u -f /var/log/sa/sa10  --  bu komanda ile ise sirf ayin 10u ucun olan sar loguna baxirsan. ayin 10u sa10 seklinde gosterir ve belede davam edir. Misal ucun ayin 
5i ucun olan loga baxmaq ucun ise orada "sa5" kimi qeyd edirik: sar -u -f /var/log/sa/sa5. Burada son 10gunun logu var.

2.
```
$ sar -P ALL

12:30:01 AM     CPU     %user     %nice   %system   %iowait    %steal     %idle
12:40:01 AM     all      0.03      0.06      0.06      0.00      0.00     99.86
12:40:01 AM       0      0.01      0.03      0.02      0.00      0.00     99.94
12:40:01 AM       1      0.05      0.03      0.06      0.00      0.00     99.85
12:40:01 AM       2      0.02      0.07      0.06      0.00      0.00     99.85
12:40:01 AM       3      0.03      0.09      0.08      0.00      0.00     99.80
```
sar -P ALL  --  her cpunu ayri-ayriliqda list edir.\
sar -P ALL 1  --  her 1 saniyaden bir reload edir.\
sar -P ALL -f /var/log/sa/sa10  --  sirf ayin 10u ucun lazim olan datalari list edir.

3.
```
$ sar -r 1 3
Linux 2.6.18-194.el5PAE (dev-db)        03/26/2011      _i686_  (8 CPU)

07:28:06 AM kbmemfree kbmemused  %memused kbbuffers  kbcached  kbcommit   %commit  kbactive   kbinact
07:28:07 AM   6209248   2097432     25.25    189024   1796544    141372      0.85   1921060     88204
07:28:08 AM   6209248   2097432     25.25    189024   1796544    141372      0.85   1921060     88204
07:28:09 AM   6209248   2097432     25.25    189024   1796544    141372      0.85   1921060     88204
Average:      6209248   2097432     25.25    189024   1796544    141372      0.85   1921060     88204
```
sar -r 1 3  --  memory statistikasini report seklinde list edir.\
kbmemfree - hal-hazirda 6209248 KB free ram oldugunu gosterir.\
kbmemused - hal-hazirda 2097432 KB istifade olunan ramdir.\
%memused - hal-hazirda istifade olunan rami %-le gosterir.\
kbcached - hal-hazirda cache-de 1796544 KB ram oldugunu gosterir.\
kbcommit - cari is yuku hecminde lazim olan ram.\
%commit - cari is yuku hecminde lazim olan ram %-le.

4.
```
$ sar -S 1 3
Linux 2.6.18-194.el5PAE (dev-db)        03/26/2011      _i686_  (8 CPU)

07:31:06 AM kbswpfree kbswpused  %swpused  kbswpcad   %swpcad
07:31:07 AM   8385920         0      0.00         0      0.00
07:31:08 AM   8385920         0      0.00         0      0.00
07:31:09 AM   8385920         0      0.00         0      0.00
Average:      8385920         0      0.00         0      0.00
```
sar -S 1 3  --  bu report swapdan statistikasini gosterir\
kbswpfree  - Amount of free swap space in kilobytes.\
kbswpused  - Amount of used swap space in kilobytes.\
%swpused - Percentage of used swap space.

5.
```
$ sar -b 1 3  --  bu misalda gorsenirki serverde hal-hazirda rate var, bu hansisa servis ola bilerki burda calisir ve s. Tam olaraq bu o demekdir ki, serverde hal-hazirda nese calisir ve server bos deyil
Linux 2.6.18-194.el5PAE (dev-db)        03/26/2011      _i686_  (8 CPU)

01:56:28 PM       tps      rtps      wtps   bread/s   bwrtn/s
01:56:29 PM    346.00    264.00     82.00   2208.00    768.00
01:56:30 PM    100.00     36.00     64.00    304.00    816.00
01:56:31 PM    282.83     32.32    250.51    258.59   2537.37
Average:       242.81    111.04    131.77    925.75   1369.90
```
sar -b 1 3  --  Serverde ne qeder input/output transfer olur onu gormek olar. Misal ucun hal-hazirda serverde hecne islemirse buradaki olan tps,rtps ve s. 0 olaraq gorsenecekki buda serverin yuklu olmamasini gosterir.Misal ucun adice servere daxil olub hansisa direktoriyani deyisendeve fayl yaradanda demekki serverde xirdada olsa hansisa isler gorulur. Bu halda bele transfer rate faizi qalxir\
tps – Transactions per second (this includes both read and write)\
rtps – Read transactions per second\
wtps – Write transactions per second\
bread/s – Bytes read per second\
bwrtn/s – Bytes written per second

6.
```
$ sar -p -d 1 1
Linux 2.6.18-194.el5PAE (dev-db)        03/26/2011      _i686_  (8 CPU)

01:59:45 PM       DEV       tps  rd_sec/s  wr_sec/s  avgrq-sz  avgqu-sz     await     svctm     %util
01:59:46 PM       sda      1.01      0.00      0.00      0.00      0.00      4.00      1.00      0.10
01:59:46 PM      sda1      1.01      0.00      0.00      0.00      0.00      4.00      1.00      0.10
01:59:46 PM      sdb1      3.03     64.65      0.00     21.33      0.03      9.33      5.33      1.62
01:59:46 PM      sdc1      3.03     64.65      0.00     21.33      0.03      9.33      5.33      1.62
01:59:46 PM      sde1      8.08      0.00    105.05     13.00      0.00      0.38      0.38      0.30
01:59:46 PM      sdf1      8.08      0.00    105.05     13.00      0.00      0.38      0.38      0.30
01:59:46 PM      sda2      1.01      8.08      0.00      8.00      0.01      9.00      9.00      0.91
01:59:46 PM      sdb2      1.01      8.08      0.00      8.00      0.01      9.00      9.00      0.91
```
sar -p -d 1 1  --  diskin hal-hazirda ne qeder yuklu oldugunu gosterir. Yuxarida da gorunduyu kimi disklerin hamisinin bir-bir outputunu verirki misalcun sdb1 diski bu qeder yuklenib.

7.
```
$ sar -w 1 3
Linux 2.6.18-194.el5PAE (dev-db)        03/26/2011      _i686_  (8 CPU)

08:32:24 AM    proc/s   cswch/s
08:32:25 AM      3.00     53.00
08:32:26 AM      4.00     61.39
08:32:27 AM      2.00     57.00
```
sar -w 1 3  --  1 saniye erzinde ne qeder proses ve context switch-in yuklenmesini gosterir.\
proc/s - 1 saniyede serverde ne qeder proses icra olundugunu gosterir. Yuxaridaki misalda 1 saniyede 3 prosesin icra olundugunu gorurur.\
cswch/s - context swithcing serverde olurki proses1 isleyir. Ardinca proses2 gelir ve CS proses1-i stop edir, proses2 bitenden sonra proses1-i qaldigi yerden start edir. Yeniki 1 saniye erzinde serverde 53 proses gozlemede qalir demekdir.

8.
```
$ sar -q 1 3
Linux 2.6.18-194.el5PAE (dev-db)        03/26/2011      _i686_  (8 CPU)

06:28:53 AM   runq-sz  plist-sz   ldavg-1   ldavg-5  ldavg-15   blocked
06:28:54 AM         0       230      2.00      3.00      5.00         0
06:28:55 AM         2       210      2.01      3.15      5.15         0
06:28:56 AM         2       230      2.12      3.12      5.12         0
Average:            3       230      3.12      3.12      5.12         0
```
sar -q 1 3 -- This reports the run queue size and load average of last 1 minute, 5 minutes, and 15 minutes.\
runq-sz	- Run queue length (number of tasks waiting for run time).\
plist-sz - Number of tasks in the task list.\
ldavg-1 - System load average for the last minute. The load average is calculated as the average number of runnable or running tasks (R state), and the number of tasks in uninterruptible sleep (D state) over the specified interval.






