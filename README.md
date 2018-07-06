# hivesql_from_qunar
these hive sql are some project from qunar.
'''
this project is create a linux shell for sent emails to some body in the in the right time.
'''

第一步：

#!/bin/bash
#-------------CopyRight--------------
# Name:
# Annotation: 酒店活跃用户及新活跃用户激活渠道分布需求
# Version Number: 1.00
# Language: bash shell
# Date: 2018-06-15
# Author: shaozhong.yu
# Email: shaozhong.yu@qunar.com
# 定时发送邮件附件并抄送
#------------------------------------

if [ $# -lt 1 ];then
        echo "输入一个日期"
        exit 1
fi

this_week=`date -d "$1" +'%Y%m%d'`
last_week=`date -d "$1 7 day ago" +'%Y%m%d'`

a=`date +%w`
if [[ $a != 4 ]]
then
  echo "today is not thursday"
  exit -1
else
  echo "today is thursday and script is running"
fi



第二步：

function sendMail(){

        DIR=$(cd $(dirname $0);pwd)
        if [[ -e $dir/huoyue.xls ]]; then
                if [[ -e $dir/huoyue_new.xls ]]; then
                        if [[ -e $dir/active_new.xls ]]; then
                                mailcontent=$DIR/mailcontent > "$mailcontent"
                                echo -e "您好!\n" >> $mailcontent
                                echo -e "\t附件为 `date +%Y-%m-%d`上一周的酒店活跃用户及新活跃用户激活渠道
分布需求数据" >> $mailcontent
                                cat mailcontent | mail -s "酒店活跃用户及新活跃用户激活渠道分布需求" -a $dir/huoyue.xls -a $dir/huoyue_new.xls -a $dir/active_new.xls shaozhong.yu@qunar.com #jiani.wu@qunar.com hotelgrowth@qunar.com yale.hu@qunar.com linl.chen@qunar.com
                                #-c hotelgrowth@qunar.com -c yale.hu@qunar.com -c linl.chen@qunar.com
                        fi
                fi
        fi
}


第三步：

#定义hive sql

hql_huoyue="set hive.auto.convert.join.noconditionaltask=false;use wirelessdata;select a.channel_name,a.channel,huoyue7,huoyue6,huoyue5,huoyue4,huoyue3,huoyue2,huoyue1,a.this_week_avg,b.last_week_avg,(a.this_week_avg-b.last_week_avg)/b.last_week_avg as two_weeks_huanbi from (select channel_name,channel,huoyue7,huoyue6,huoyue5,huoyue4,huoyue3,huoyue2,huoyue1,(huoyue1+huoyue2+huoyue3+huoyue4+huoyue5+huoyue6+huoyue7)/7 as this_week_avg from t_hotel_huoyue_result where type='huoyue' and dt='${this_week}') a left outer join (select channel_name,channel,(huoyue1+huoyue2+huoyue3+huoyue4+huoyue5+huoyue6+huoyue7)/7 as last_week_avg from t_hotel_huoyue_result where type='huoyue' and dt='${last_week}') b on a.channel_name=b.channel_name and a.channel=b.channel"

hql_huoyue_new="set hive.auto.convert.join.noconditionaltask=false;use wirelessdata;select a.channel_name,a.channel,huoyue7,huoyue6,huoyue5,huoyue4,huoyue3,huoyue2,huoyue1,a.this_week_avg,b.last_week_avg,(a.this_week_avg-b.last_week_avg)/b.last_week_avg as two_week_huanbi from (select channel_name,channel,huoyue7,huoyue6,huoyue5,huoyue4,huoyue3,huoyue2,huoyue1,(huoyue1+huoyue2+huoyue3+huoyue4+huoyue5+huoyue6+huoyue7)/7 as this_week_avg from t_hotel_huoyue_result where type='huoyue_new' and dt='${this_week}') a left outer join (select channel_name,channel,(huoyue1+huoyue2+huoyue3+huoyue4+huoyue5+huoyue6+huoyue7)/7 as last_week_avg from t_hotel_huoyue_result where type='huoyue_new' and dt='${last_week}') b on a.channel_name=b.channel_name and a.channel=b.channel"

hql_active_new="set hive.auto.convert.join.noconditionaltask=false;use wirelessdata;select a.channel_name,a.channel,huoyue7,huoyue6,huoyue5,huoyue4,huoyue3,huoyue2,huoyue1,a.this_week_avg,b.last_week_avg,(a.this_week_avg-b.last_week_avg)/b.last_week_avg as two_weeks_huanbi from (select channel_name,channel,huoyue7,huoyue6,huoyue5,huoyue4,huoyue3,huoyue2,huoyue1,(huoyue1+huoyue2+huoyue3+huoyue4+huoyue5+huoyue6+huoyue7)/7 as this_week_avg from t_hotel_huoyue_result where type='active_new' and dt='${this_week}') a left outer join (select channel_name,channel,(huoyue1+huoyue2+huoyue3+huoyue4+huoyue5+huoyue6+huoyue7)/7 as last_week_avg from t_hotel_huoyue_result where type='active_new' and dt='${last_week}') b on a.channel_name=b.channel_name and a.channel=b.channel"

第四步：
#判断是否有以下文件，如果有责进行csv转xls
function txtToxls(){
  if [[ -e $dir/huoyue.csv ]]; then
        python txtToxls.py $dir/huoyue.csv $dir/huoyue
  fi
  if [[ -e $dir/huoyue_new.csv ]]; then
        python txtToxls.py $dir/huoyue_new.csv $dir/huoyue_new
  fi
  if [[ -e $dir/active_new.csv ]]; then
        python txtToxls.py $dir/active_new.csv $dir/active_new
  fi
}

第五步：
#Python脚本csv 转 xls

#encoding: utf-8
import datetime
import time
import os
import sys
import xlwt
import datetime

d = datetime.datetime.now()

def txt2xls(filename,xlsname):

    f = open(filename,'rb')
    x = 1
    y = 0
    xls=xlwt.Workbook()
    sheet = xls.add_sheet('sheet1',cell_overwrite_ok=True)
    list = [u'渠道名称',u'渠道类型',(d + datetime.timedelta(days=-7)).strftime("%Y%m%d"),(d +  datetime.timedelta(days=-6)).strftime("%Y%m%d"),(d +  datetime.timedelta(days=-5)).strftime("%Y%m%d"),(d + datetime.timedelta(days=-4)).strftime("%Y%m%d"),(d + datetime.timedelta(days=-3)).strftime("%Y%m%d"),(d + datetime.timedelta(days=-2)).strftime("%Y%m%d"),(d + datetime.timedelta(days=-1)).strftime("%Y%m%d"),u'本周平均',u'上>周平均',u'环比']
    for i in range(0,len(list)):
        sheet.write(0,i,list[i])
    while True:
        line = f.readline()
        if not line:
            break
        for i in line.split(b'\t'):
            item = i.strip().decode('utf-8')
            if y == 11:
                if item != "NULL":
                    item_tmp = float(item)
                    item_tmp *= 100
                    item_last = str(item_tmp)
                    sheet.write(x, y, item_last+"%")
              else:
                sheet.write(x, y, item)
            y += 1
        x += 1
        y = 0
    f.close()
    xls.save(xlsname+'.xls')

if __name__ == "__main__":
    filename = sys.argv[1]
    xlsname  = sys.argv[2]
    txt2xls(filename,xlsname)


第六步：
#执行
sudo -u wirelessdev hive -e "$hql_huoyue" > $dir/huoyue.csv
sudo -u wirelessdev hive -e "$hql_huoyue_new" > $dir/huoyue_new.csv
sudo -u wirelessdev hive -e "$hql_active_new" > $dir/active_new.csv
txtToxls
sendMail
