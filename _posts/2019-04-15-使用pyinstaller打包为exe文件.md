---

title: "打包py文件为exe文件 + 把以前自己的脚本使用tkinter打包"
date: 2019-04-15 17:43
author: dzt
subtitle:  pyinstaller打包为exe
tags:
  - python
  - exe
  - tkinter
---

## 前言

很久之前就有一个想法，把脚本打包为一个exe文件，就不用每次去修改源码，再跑这么麻烦了，还有就是这种重复性工作，完全可以给其他人使用啊！(关键)

脚本用于 nginx 的日志分析 

仅限于 access.log 和 error.log  

文件名称可修改为 accseexxx.log  errorxxx.log  格式



##### 一、GetIP


* 统计nginx服务器上的access.log日志，输出结果如下：

> **************************************************
> [15/Apr/2019:06:25:04] 至 [15/Apr/2019:09:37:22]
> 访问量:128266
> IP个数:10949 
> 单秒最大访问量:227
> 单秒最大访问量时间:[15/Apr/2019:06:30:53
> 6点至7点，访问量：30303次
> 7点至8点，访问量：51089次
> 8点至9点，访问量：30023次
> [15/Apr/2019:06:30:53，227次
> [15/Apr/2019:06:30:52，200次
> [15/Apr/2019:06:40:53，197次
> [15/Apr/2019:06:50:53，183次
> [15/Apr/2019:06:50:52，178次
> [15/Apr/2019:06:40:52，170次
> [15/Apr/2019:06:30:54，163次
> [15/Apr/2019:06:50:54，132次
> [15/Apr/2019:06:40:54，114次
> [15/Apr/2019:06:30:51，107次
> [15/Apr/2019:06:50:51，96次
> [15/Apr/2019:06:40:51，87次
> [15/Apr/2019:06:30:55，85次
> [15/Apr/2019:06:50:55，82次
> [15/Apr/2019:06:30:12，80次
> [15/Apr/2019:06:30:42，79次
> [15/Apr/2019:06:40:42，79次
> [15/Apr/2019:07:00:12，65次
> [15/Apr/2019:06:50:43，65次
> [15/Apr/2019:06:40:55，64次
> 统计时间: 2019-04-15 09:38:09.351366
>
> **************************************************





* 统计nginx服务器上的error.log日志，输出结果如下:

> connect() to unix:///home/ubuntu/www/cmxsite/cmxsite.sock failed (111: Connection refused) while connecting to upstream,  
> directory index of "/home/ubuntu/www/html/home/" is forbidden,  
> rewrite or internal redirection cycle while processing "/index.php",  
> connect() to unix:///home/ubuntu/www/eleccard/eleccard.sock failed (111: Connection refused) while connecting to upstream,  
>
> 06:56:21 至 20:08:17  
> nginx error出现ip个数:87   
> nginx error类型个数:4   
> nginx error次数:98  
> 统计时间: 2018-10-24 17:16:36.199000  

2018-10-24  error的正则表达式 花费了我很多时间





其实打包成一个exe文件是非常非常简单的  使用PyInstaller这个包就完了！



但是，既然这样我可能要写一个窗口界面来进行操作啊

所以，用到了之前使用过的tkinter









## 效果

```python
pip3 install pyinstaller  # 安装pyinstaller 及其依赖包
```

```python
pyinstaller -F -w -i bitbug_favicon.ico tk_nginx.py  #  打包tk_nginx.py为tk_nginx.exe
```

-F 打包为单个文件

-D 打包为文件夹

-w 去掉命令行终端框 只对Windows有效

-i 添加图标



![](https://raw.githubusercontent.com/yanshigou/yanshigou.github.io/master/img/t/tk_nginx.png)

![](https://raw.githubusercontent.com/yanshigou/yanshigou.github.io/master/img/t/tk_nginx1.png)

![](https://raw.githubusercontent.com/yanshigou/yanshigou.github.io/master/img/t/tk_nginx2.png)

![](https://raw.githubusercontent.com/yanshigou/yanshigou.github.io/master/img/t/tk_nginx3.png)





## 源码

我的github地址 <https://github.com/yanshigou/MyScripts/tree/master/GetIP>


```python
# encoding: utf-8
import re
import numpy as np
from datetime import datetime
import tkinter as tk
import tkinter.filedialog
time1 = datetime.now()


# 提取ip地址，去重，获取不同ip个数
def GetAccessIp(input_file_name, output_file_name):
    sep = '\n'
    sep1 = '*'*50 + '\n'
    sep1 = '*'*50 + '\n'
    sep2 = '\n' + '*'*50 + '\n\n'
    ip_list=[]
    timelist = []
    alltimelist = []
    requestlist = []
    requestlist2 = []

    fLog = open(input_file_name)
    for each in fLog:
        # 匹配ip
        ip = re.findall(r'(?<![.\d])(?:\d{1,3}\.){3}\d{1,3}(?![.\d])', str(each), re.S)
        if ip == []:
            continue
        ip_list.append(ip[0])
        # 分割log中每行空格
        iptime = each.split()[3]
        # print(iptime[-8:])  #为00:00:00格式时间
        timelist.append(iptime)
        # 匹配每个小时
        alltime = re.findall(r':(20|21|22|23|[0-1]\d)', str(iptime), re.S)
        alltimelist.append(int(alltime[0]))

        # 匹配请求类型
        request = re.findall(r'(?<= ").*(?=\?)', str(each), re.S)
        if request == []:
            request = re.findall(r'(?<= ").*(?= HTTP)', str(each), re.S)
            if request == []:
                request = re.findall(r'(?<= ").*', str(each), re.S)
        # print(request)
        requestlist.append(request[0])
    # print(requestlist)

    # 再在requestlist中匹配地址
    for ea in requestlist:
        request2 = re.findall(r'.*(?=\?)', str(ea), re.S)
        if request2 == []:
            request2 = re.findall(r'.*(?= HTTP)', str(ea), re.S)
            if request2 == []:
                request2 = re.findall(r'.*', str(ea), re.S)

        # print(request2)
        requestlist2.append(request2[0])
    # print(requestlist2)


    # 计算每个小时访问次数
    count1 = 1
    countlist1 = []
    alltimelist.sort()
    all = list(set(alltimelist))
    all.sort(key=alltimelist.index)

    for x in range(len(alltimelist)-1):
        if alltimelist[x+1] == alltimelist[x]:
            count1 += 1
        else:
            countlist1.append(count1)
            count1 = 1


    # 计算每种请求地址访问次数
    # count2 = 1
    # countlist2 = []
    # requestlist2.sort()
    # r = list(set(requestlist2))
    # r.sort()
    # # list2.sort(key=list1.index)
    # # print(requestlist2)
    #
    # for x in range(len(requestlist2)-1):
    #     if requestlist2[x+1] == requestlist2[x]:
    #         count2 += 1
    #     else:
    #         countlist2.append(count2)
    #         count2 = 1
    # print(r)
    # print(countlist2)

    # 计算每秒最大访问量
    count = 1
    countlist = []
    tlists = []
    timelist.sort()

    for x in range(len(timelist)-1):
        if timelist[x] == timelist[x+1]:
            count += 1
        else:
            tlists.append(timelist[x])
            countlist.append(count)
            count = 1
    # print(countlist)
    print(max(countlist))
    aa = np.array(countlist)
    # print(a)
    # b为索引的排序的列表 从小到大排列  b[-1]为最大
    ba = np.argsort(aa)
    # print(b)
    # countlist.sort(key=countlist.index)

    #取出最大访问量的时间
    bb = list(ba[-20:])
    print(bb)
    maxtime = []
    maxcountlist = []
    for i in bb:
        maxtime.append(tlists[i])
        maxcountlist.append(countlist[i])
    print(maxtime)
    print(maxcountlist)

    # print(tlist)
    # print(len(tlists))
    # for x in range(len(countlist)-1):
    #     if countlist[x] == max(countlist):
    #         maxtime = tlists[x]
    #         break

    ips = list(set(ip_list))

    print("access不同ip个数:%s " % len(ips))

    # 写 python3中需要encoding='utf-8'
    fout = open(output_file_name, 'a+', encoding='utf-8')
    a = all
    b = countlist1

    # API次数
    # c = r
    # d = countlist2



    # for each in ips:
      # print(each)
      # fout.write(each + sep)
    # ip_count = {}
    # for i in ips:
    #     ip_count[i] = ip_list.count(i)
    # print()

    fout.write(sep1 + timelist[0] + '] 至 ' + timelist[-1] + ']' + sep)
    fout.write("访问量:%s" % len(ip_list) + sep)
    fout.write("IP个数:%s " % len(ips) + sep)
    fout.write("单秒最大访问量:%s" % max(countlist) + sep)
    fout.write("单秒最大访问量时间:%s" % maxtime[-1] + sep)
    for h in range(len(a)-1):
        fout.write("%s点至%s点，访问量：%s次" % (a[h], a[h]+1, b[h])+sep)
    # # API次数
    # for q in range(len(c)-1):
    #     fout.write("%s，%s次"%(c[q], d[q]) + sep)

    for i in range(1, len(maxtime)+1):
        fout.write("%s，%s次" % (maxtime[-i], maxcountlist[-i]) + sep)

    # fout.write('最大5个访问ip:' + sep)
    # temp = sorted(ip_count.items(), key=lambda x: x[1], reverse=True)[:5]
    # for i in temp:
    #     fout.write(str(i) + sep)

    fout.write('统计时间: ' + str(datetime.now()) + sep2)

    fout.close()
    fLog.close()
    print("ip提取完毕")


def GetErrorIP(input_file_name2,output_file_name2):
    sep = '\n'
    sep1 = '*'*50 + '\n'
    sep2 = '\n' + '*'*50 + '\n\n'
    ip_list = []
    err_list = []
    timelist = []
    alert_list = []
    errorL = []

    fLog = open(input_file_name2)
    for each in fLog:
        ip = re.findall(r'(?<![.\d])(?:\d{1,3}\.){3}\d{1,3}(?![.\d])', str(each), re.S)
        # print(ip)

        err = re.findall(r'(?<=: ).*(?=, client)', str(each), re.S)
        if err == []:
            err = re.findall(r'(?<=: ).*', str(each), re.S)
            e = err[0].split()[0:]
        else:
            print(err)
            e = err[0].split()[1:]
        # err_list.append(err[0])
        # print(err[0])
        # print(err)
        error = ' '.join(e)
        err_list.append(error)
        # print(error)

        # alert = re.findall(r'alert', str(each), re.S)
        # alert_list.append(alert[0])
        #
        # errorl = re.findall(r'error', str(each), re.S)
        # errorL.append(errorl[0])

        # err_list.append(each)
        errtime = each.split()[1]
        timelist.append(errtime)

        if ip == []:
            continue
        else:
            ip_list.append(ip[0])

    ips = list(set(ip_list))
    errs = list(set(err_list))

    # print(errs)
    print("error总数：%s" % len(err_list))
    print("error中不同ip个数:%s " % len(ips))
    print("error不同个数:%s " % len(errs))
    # print("alert个数:%s " % len(alert_list))
    # print("error个数:%s " % len(errorL))

    # 写 python3中需要encoding='utf-8'
    fout = open(output_file_name2, "a", encoding='utf-8')
    # for each in ips:
    #   # print(each)
    #   fout.write(each + sep)

    for each in errs:
        fout.write(each + sep)

    fout.write(sep1 + timelist[0] + ' 至 ' + timelist[-1] + sep)
    # fout.write("nginx error出现ip个数:%s "% len(ips) + sep)
    fout.write("error类型个数:%s " % len(errs) + sep)
    fout.write("报错总数:%s " % len(err_list) + sep)

    # fout.write("nginx error总数:%s "% len(errorL) + sep)
    # x = len(err_list)-len(ip_list)
    # fout.write("nginx error次数:%s"% len(ip_list) + sep)
    # fout.write("nginx alert个数:%s "% x + sep)
    fout.write('统计时间: ' + str(datetime.now()) + sep2)
    # print(ips)

    fout.close()
    fLog.close()
    print("error提取完毕")


def TkWindow():
    root.title("---NginxLog分析器---")

    btn = tk.Button(root, text='选择文件', command=files)
    lb.pack()
    # lb.grid(row=0, column=0, pady=10)
    btn.pack()
    # btn.grid(row=20, columnspan=20, pady=20)

    root.geometry('500x200+800+400')
    # root.maxsize(500, 300)
    # root.minsize(500, 300)
    root.mainloop()


def file():
    filename = tk.filedialog.askopenfilename()
    if filename != "":
        lb.config(text="您选择的文件是："+filename)
    else:
        lb.config(text="您没有选择任何文件：" + filename)


def files():
    filenames = tkinter.filedialog.askopenfilenames()
    if len(filenames) != 0:
        string_filename = ""
        for i in range(0, len(filenames)):
            path = str(filenames[i])
            # 传access.log
            # if path[-10:] == "access.log":
            #     output_access = path[:-4] + str(datetime.now())[0:10] + '.txt'
            #     print(path)
            #     print(output_access)
            # elif path[-9:] == "error.log":
            #     output_error = path[:-4] + str(datetime.now())[0:10] + '.txt'
            #     print('error')
            #     print(path)
            #     print(output_error)

            # 传access2019-04-14.log
            if "access" in path and ".log" in path:
                output_access = path[:-4] + '.txt'
                print(path)
                print(output_access)
                GetAccessIp(path, output_access)
                string_filename += str(filenames[i]) + " 分析完成！！" + "\n"
            elif "error" in path and ".log" in path:
                output_error = path[:-4] + '.txt'
                print(path)
                print(output_error)
                GetErrorIP(path, output_error)
                string_filename += str(filenames[i]) + " 分析完成！！" + "\n"
            else:
                string_filename += str(filenames[i]) + " 分析失败！！" + "\n"
            lb.config(text="您选择的文件是：" + string_filename)
    else:
        lb.config(text="您没有选择任何文件")


if __name__ == '__main__':
    # input_file_name = "D:\\work_CMX\\log\\access2019-04-11.log"
    # output_file_name = "D:\\work_CMX\\log\\output2019-04-11.txt"
    # GetAccessIp(input_file_name, output_file_name)
    # input_file_name2 = "D:\\work_CMX\\log\\error2019-04-11.log"
    # # output_file_name2 = "D:\\work_CMX\\log\\output_error2018-11-18.txt"
    # GetErrorIP(input_file_name2, output_file_name)
    # time2 = datetime.now()
    # print('总共耗时：' + str(time2 - time1) + 's')
    root = tk.Tk()
    lb = tk.Label(root, text="您没有选择任何文件")
    TkWindow()


```

