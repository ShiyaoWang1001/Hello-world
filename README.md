# Hello-world
#第一个库
#西建大图书馆预约脚本
import requests
import json
import time
import datetime
import os
import sys
url = "http://seat.lib.xauat.edu.cn/reservations/futureReserve"   # 预约明天座位POST的网址（当日预约POST的网址不是这个）
urlone = input('''Hi！我是追风蟹，生活愉快
我可以定时帮您预约我们学校图书馆的座位哦 
请在这里复制粘贴您的座位预约网址：''')
#定义基本变量
a = input('''请输入您要预约的日期
例如 02-07  ： ''')
b = input('''
雁塔三楼围廊E区：1
雁塔一楼大厅A区：2
雁塔二楼文库B区：3
雁塔三楼上网D区：4
雁塔三楼自修F区：5
雁塔四楼自习G区：6
雁塔四楼上网H区：8
请输入您要预约区域的代号：'''
              )

hour = input('请输入开始预约时间的小时数：')
minute = input('请输入开始预约时间的分钟数：')

timing = input('请问是否需要预约结束后自动关机（关机扣1敲回车，不关机扣0敲回车）：')

print('好的，已经开始工作，时间到了我会告诉您预约结果，请别关闭电脑、程序或者断网...')
time.sleep(2)
print('请等待...')
try:
    while (True):
        now = datetime.datetime.now()  # 获取当前系统时间
        if (now.hour == eval(hour) and now.minute == eval(minute)):
            break
    time.sleep(0.5)
    v = time.perf_counter()     #开始计时
#获取cookie
    m = requests.session()
    m = m.get(urlone)
    cookies = requests.utils.dict_from_cookiejar(m.cookies)     #将cookie的cookiejar格式转换为dict格式
#提取不同键cookie的值
    g = cookies['XSRF-TOKEN']
    h = cookies['laravel_session']
    p = g.strip('%3D')                 #本cookie的URL编码最后部分可能会有=号，在获取时会被编码成%3D，所以要消除掉
    l = h.strip('%3D')
#修改请求头
    kv = {'Host': 'seat.lib.xauat.edu.cn',
        'User-Agent': 'Mozilla/5.0 (Windows NT 6.3; Win64; x64; rv:63.0) Gecko/20100101 Firefox/63.0',
        'Accept': 'application/json, text/plain, */*',
        'Accept-Language': 'zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2',
        'Accept-Encoding': 'gzip, deflate',
        'Referer': 'http://seat.lib.xauat.edu.cn/',
        'Content-Type': 'application/json;charset=utf-8',
        'X-XSRF-TOKEN': p,
        'Content-Length': '43',
        'Connection': 'keep-alive',
        'Cookie':'XSRF-TOKEN='+p+'; laravel_session='+l
          }

    for x in range(21):                                #高峰期可能会请求失败，所以循环提交21次请求
#预约上午座位
        kt = {'date': '2019-' + a,
              'period': 0,
              'roomId': eval(b)
              }
        kt1 = json.dumps(kt)
        requests.post(url, headers=kv, data=kt1)
#预约下午座位
        kt = {'date': '2019-' + a,
              'period': 1,
              'roomId': eval(b)
              }
        kt1 = json.dumps(kt)
        requests.post(url, headers=kv, data=kt1)

        if (x == 0):
            print('恭喜啦！成功预约到全天座位')
            o = time.perf_counter() - v
            print('预约用时：{:.3f}秒'.format(o))
        else:
            print('保险重复预约{0}次'.format(x))
    time.sleep(30)
    if (eval(timing) == 1):          #如果开始输入的关机指令为1，计算机到时间自动关闭
        os.system('shutdown -s -f -t 1')
    else:
        pass
except:
    print("爬取失败")
    time.sleep(30)
    if (eval(timing) == 1):
        os.system('shutdown -s -f -t 1')
    else:
        pass
