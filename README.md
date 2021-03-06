----------  
Author:   duguowei  
Date:     2022-5-22  
Email:    duguoweisz@gmail.com  
Phone:    13417431640  
WeChat:   dgw_cn  
Kernel:   5.10.66  
----------  
What is it ?  
----------  
fanotifyd is a filesystem monitor. It gives a detailed listing of the list of  
files that running processes are doing IO to (with the pid and task_name of  
read/write to each file). The goal is that using this tool, we can real time  
identify IO paths of all apps.  
  
fanotifyd is implemented as a collection of kernel filesystem noitification,  
and it needs root or CAP_SYS_ADMIN. The running needs to disable SELinux by  
adb shell setenforce 0.  
  
When running fanotifyd, it internal initializes two groups, One group is  
responsible for monitoring the path event such as open/access/modify.  
The other group is responsible for monitoring the dirent event such as  
create/delete...  

format of each line is as below :  
--------------------------  
timestamp  (pid)task_name  event_type	file_path  

note that the timestamp is not the real time of event, but is the monitor  
print time. The event was first enqueued into group notification list by  
the running process, and then, it will be fetched by the fanotifyd.  
  
For open/access/modify/close events, it monitors /system /vendor /product  
/odm /oem /data directory, and for the create/delete events, it only  
monitors the /data.  
  
What does the output from fanotifyd look like ?  
-------------------------------------------  
...  
...  
05-22 00:41:55.076  (  3544)tycenter.remote  OPEN|ACCESS|CLOSE_NOWRITE        /data/user/0/com.miui.securitycenter/databases/IdProvider-journal  
05-22 00:41:55.076  (  3544)tycenter.remote  ACCESS                           /data/user/0/com.miui.securitycenter/databases/IdProvider  
05-22 00:41:55.078  (  2233)iptables-restor  OPEN                             /system/etc/xtables.lock  
05-22 00:41:55.078  (  2233)iptables-restor  CLOSE_NOWRITE                    /system/etc/xtables.lock  
05-22 00:41:55.079  (  2235)ip6tables-resto  OPEN                             /system/etc/xtables.lock  
05-22 00:41:55.079  (  3544)tycenter.remote  OPEN|ACCESS|CLOSE_NOWRITE        /data/user/0/com.miui.securitycenter/databases/IdProvider-journal  
05-22 00:41:55.079  (  3544)tycenter.remote  ACCESS                           /data/user/0/com.miui.securitycenter/databases/IdProvider  
05-22 00:41:55.080  (  2235)ip6tables-resto  CLOSE_NOWRITE                    /system/etc/xtables.lock  
05-22 00:41:55.084  (  3022)tcpdump-vendor   MODIFY                           /data/vendor/wlan_logs/tcpdump.pcap2  
05-22 00:41:55.085  (  3022)tcpdump-vendor   MODIFY                           /data/vendor/wlan_logs/tcpdump.pcap2  
05-22 00:41:55.330  (  1315)connsyslogger    MODIFY                           /data/connsyslog/bootupLog/WIFI_FW_2022_0521_225547.clog.curf  
05-22 00:41:55.450  (  4477)iui.powerkeeper  OPEN|ACCESS|CLOSE_NOWRITE        /data/user/0/com.miui.powerkeeper/databases/user_configure.db-journal  
05-22 00:41:55.450  (  4477)iui.powerkeeper  ACCESS                           /data/user/0/com.miui.powerkeeper/databases/user_configure.db  
05-22 00:41:57.133  (  3022)tcpdump-vendor   MODIFY                           /data/vendor/wlan_logs/tcpdump.pcap2  
05-22 00:41:57.486  (  2234)rkstack.process  CLOSE_WRITE                      /data/user_de/0/com.android.networkstack/databases/IpMemoryStore.db  
05-22 00:41:57.555  ( 19242)om.miui.gallery  CREATE                           /data/data/com.miui.gallery/shared_prefs/mi_stat_pref.xml  
05-22 00:41:57.555  (  3392)re.connectivity  CREATE                           /data/data/com.miui.mishare.connectivity/shared_prefs/mi_stat_pref.xml  
05-22 00:41:57.555  (  3392)re.connectivity  OPEN|MODIFY                      /data/data/com.miui.mishare.connectivity/shared_prefs/mi_stat_pref.xml  
05-22 00:41:57.556  ( 19242)om.miui.gallery  OPEN|MODIFY                      /data/data/com.miui.gallery/shared_prefs/mi_stat_pref.xml  
05-22 00:41:57.561  ( 19242)om.miui.gallery  CLOSE_WRITE                      /data/data/com.miui.gallery/shared_prefs/mi_stat_pref.xml  
05-22 00:41:57.561  (  3392)re.connectivity  CLOSE_WRITE                      /data/data/com.miui.mishare.connectivity/shared_prefs/mi_stat_pref.xml  
05-22 00:41:57.561  (  3392)re.connectivity  DELETE                           /data/data/com.miui.mishare.connectivity/shared_prefs/mi_stat_pref.xml.bak  
...  
...  
05-22 00:41:55.076  (  3544)tycenter.remote  OPEN|ACCESS|CLOSE_NOWRITE        /data/user/0/com.miui.securitycenter/databases/IdProvider-journal  
05-22 00:41:55.076  (  3544)tycenter.remote  ACCESS                           /data/user/0/com.miui.securitycenter/databases/IdProvider  
05-22 00:41:55.078  (  2233)iptables-restor  OPEN                             /system/etc/xtables.lock  
05-22 00:41:55.078  (  2233)iptables-restor  CLOSE_NOWRITE                    /system/etc/xtables.lock  
05-22 00:41:55.079  (  2235)ip6tables-resto  OPEN                             /system/etc/xtables.lock  
05-22 00:41:55.079  (  3544)tycenter.remote  OPEN|ACCESS|CLOSE_NOWRITE        /data/user/0/com.miui.securitycenter/databases/IdProvider-journal  
05-22 00:41:55.079  (  3544)tycenter.remote  ACCESS                           /data/user/0/com.miui.securitycenter/databases/IdProvider  
05-22 00:41:55.080  (  2235)ip6tables-resto  CLOSE_NOWRITE                    /system/etc/xtables.lock  
05-22 00:41:55.084  (  3022)tcpdump-vendor   MODIFY                           /data/vendor/wlan_logs/tcpdump.pcap2  
05-22 00:41:55.085  (  3022)tcpdump-vendor   MODIFY                           /data/vendor/wlan_logs/tcpdump.pcap2  
05-22 00:41:55.330  (  1315)connsyslogger    MODIFY                           /data/connsyslog/bootupLog/WIFI_FW_2022_0521_225547.clog.curf  
05-22 00:41:55.450  (  4477)iui.powerkeeper  OPEN|ACCESS|CLOSE_NOWRITE        /data/user/0/com.miui.powerkeeper/databases/user_configure.db-journal  
05-22 00:41:55.450  (  4477)iui.powerkeeper  ACCESS                           /data/user/0/com.miui.powerkeeper/databases/user_configure.db  
05-22 00:41:57.133  (  3022)tcpdump-vendor   MODIFY                           /data/vendor/wlan_logs/tcpdump.pcap2  
05-22 00:41:57.486  (  2234)rkstack.process  CLOSE_WRITE                      /data/user_de/0/com.android.networkstack/databases/IpMemoryStore.db  
05-22 00:41:57.555  ( 19242)om.miui.gallery  CREATE                           /data/data/com.miui.gallery/shared_prefs/mi_stat_pref.xml  
05-22 00:41:57.555  (  3392)re.connectivity  CREATE                           /data/data/com.miui.mishare.connectivity/shared_prefs/mi_stat_pref.xml  
05-22 00:41:57.555  (  3392)re.connectivity  OPEN|MODIFY                      /data/data/com.miui.mishare.connectivity/shared_prefs/mi_stat_pref.xml  
05-22 00:41:57.556  ( 19242)om.miui.gallery  OPEN|MODIFY                      /data/data/com.miui.gallery/shared_prefs/mi_stat_pref.xml  
05-22 00:41:57.561  ( 19242)om.miui.gallery  CLOSE_WRITE                      /data/data/com.miui.gallery/shared_prefs/mi_stat_pref.xml  
05-22 00:41:57.561  (  3392)re.connectivity  CLOSE_WRITE                      /data/data/com.miui.mishare.connectivity/shared_prefs/mi_stat_pref.xml  
05-22 00:41:57.561  (  3392)re.connectivity  DELETE                           /data/data/com.miui.mishare.connectivity/shared_prefs/mi_stat_pref.xml.bak  
...  
...  
  
Kernel Patches Required :  
-----------------------  
fanotfyd with feature FAN_REPORT_FID needs the kernel version after 5.9,  
and needs to open kenrel config as below:  
CONFIG_FSNOTIFY=y  
CONFIG_FANOTIFY=y  
CONFIG_FHANDLE=y  
  
How to test :  
-----------  
$m fanotifyd  
$adb root  
$adb remount  
$adb push fanotifyd /system/bin/  
$adb shell fanotifyd  
