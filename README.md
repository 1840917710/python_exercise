import pandas as pd
import numpy as np
import datetime
import xlrd
import os

filepath = "/root/autodl-tmp/data"
file_names = os.listdir(filepath)
list_data = [] 
for name in file_names:
    if name.startswith("."): continue
    data = pd.read_csv(filepath + "/" + name)
    list_data.append(data)
# 首先对所有数据按时间降序排列
new_col = ['Car_id','Line','lng','Lat','Time']
data_lenth = len(list_data)
# type(list_data[1].iloc[0][-1])
for i in range(data_lenth):
    list_data[i].columns = new_col
    list_data[i].sort_values(by='Time',inplace=True)
    
deal_data = list_data[:]
data_number = len(deal_data) 

#  - deal_data[0].iloc[0][4]
# def timeSecond(time):#此函数将得到的datetime.time格式转换成秒
#     return time.hour * 3600 + time.minute * 60 + time.second
def timeSecond(time):
    #将时间转化为字符串,hour,minute,seconds
    
    x_time = str(time).split(":")
    # 转化为总秒数
    total_seconds = int(x_time[0])*3600 + int(x_time[1])*60 + int(x_time[2])
    #返回总秒数
    return total_seconds

# 此函数将得到的秒数转换成datetime.time格式
def getTime(time):
    gethour = time//3600
    getminute = (time%3600)//60
    getsecond = time%60
    return datetime.time(gethour, getminute, getsecond)
    
 #  - deal_data[0].iloc[0][4]
# def timeSecond(time):#此函数将得到的datetime.time格式转换成秒
#     return time.hour * 3600 + time.minute * 60 + time.second
def timeSecond(time):
    #将时间转化为字符串,hour,minute,seconds
    
    x_time = str(time).split(":")
    # 转化为总秒数
    total_seconds = int(x_time[0])*3600 + int(x_time[1])*60 + int(x_time[2])
    #返回总秒数
    return total_seconds

# 此函数将得到的秒数转换成datetime.time格式
def getTime(time):
    gethour = time//3600
    getminute = (time%3600)//60
    getsecond = time%60
    return datetime.time(gethour, getminute, getsecond)
    
# 插值
for i in range(data_number):
    middle = pd.DataFrame(columns=new_col)
    for index, row in deal_data[i].iterrows():
        if index == 0:
            continue
        sindex = index - 1
        # deal_data[i].iloc[index][4] = pd.to_datetime(deal_data[i].iloc[index][4])
        # deal_data[i].iloc[sindex][4] = pd.to_datetime(deal_data[i].iloc[sindex][4])
        delta = timeSecond(deal_data[i].iloc[index][4])-timeSecond(deal_data[i].iloc[sindex][4])#相邻两行数据时间差
        deal_data[i].iloc[index][4] = getTime(timeSecond(deal_data[i].iloc[index][4]))
        deal_data[i].iloc[sindex][4] = getTime(timeSecond(deal_data[i].iloc[sindex][4]))
        # deal_data[i].iloc[index][4] = getTime(deal_data[i].iloc[index][4]) 
        # deal_data[i].iloc[sindex][4] = getTime(deal_data[i].iloc[sindex][4])
        for second in range(1,delta):
            car_id = row[0]
            line = row[1]
            lng = deal_data[i].iloc[sindex][2] + i * (row[2] - deal_data[i].iloc[sindex][2]) / delta
            lat = deal_data[i].iloc[sindex][3] + i * (row[3] - deal_data[i].iloc[sindex][3]) / delta
            ctime = getTime(timeSecond(deal_data[i].iloc[sindex][4])+second)
            middle1 = pd.DataFrame([[car_id,line,lng,lat,ctime]],columns=new_col)
            middle = middle.append(middle1,ignore_index=True)
    deal_data[i] = deal_data[i].append(middle,ignore_index=True)
    print(deal_data[i].iloc[:,:])
    # deal_data[i].iloc[:,4] = getTime(deal_data[i].iloc[:,4])
    deal_data[i].sort_values(by='Time',inplace=True)
    deal_data[i].reset_index(drop=True,inplace=True)
    print(del_data[i].iloc[:,:])
    
# 去重，相邻两行数据经纬度相同认为为重复数据，则删除第二行
for i in range(data_number):
    del_list = []
    for index, row in deal_data[i].iterrows():
        if index == 0:
            continue
        if (row[2] ==deal_data[i].iloc[index-1][2]) and (row[3] ==deal_data[i].iloc[index-1][3]):
            del_list.append(index)
    deal_data[i] = deal_data[i].drop(del_list)
    deal_data[i].reset_index(drop=True,inplace=True)
