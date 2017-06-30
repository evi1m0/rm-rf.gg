---
layout: post
title: 使用CloudSight API进行图像识别的Python脚本
---

#### CloudSight: [https://www.cloudsightapi.com/](https://www.cloudsightapi.com/)

\> python cloudsight.py http://url/test.jpg

    #!/usr/bin/env python
    # coding=utf8
    # author=evi1m0@2015<ff0000team>

    import sys
    import time
    import requests

    def _api(url):
        count = 0
        api_url = 'http://api.cloudsightapi.com/image_requests'
        res_url = 'http://api.cloudsightapi.com/image_responses/'
        headers = {
                    'Origin': 'http://cloudsightapi.com',
                    'HOST': 'api.cloudsightapi.com',
                    'Authorization': 'CloudSight amZd_zG32VK-AoSz05JLIA',
                  }
        post_data = {
                    'image_request[remote_image_url]': url,
                    'image_request[locale]': 'zh-CN',
                    'image_request[language]': 'zh-CN',
                    }

        token_req = requests.post(api_url, data=post_data, headers=headers)
        token = token_req.json()['token']
        while count<10:
            try:
                count += 1
                print '[+] Loading...'
                result = requests.get('%s%s'%(res_url, token), headers=headers)
                status = result.json()['status']
                if status == 'completed':
                    print '[+] Pic: %s' % url
                    print '[*] Name: %s' % result.json()[u'name']
                    break
            except Exception, e:
                print '[-] False: %s' % str(e)
                pass    if __name__ == '__main__':
        try:
            url = sys.argv[1]
            _api(url)
        except Exception, e:
            print 'Usage: cloudsightapi.py http://pic_url.com/test.jpg'
            print 'Error: %s' % str(e)
            pass

#### Result

![http://ww4.sinaimg.cn/large/c334041bgw1epqhs1zejnj20nb0c3myq.jpg](http://ww4.sinaimg.cn/large/c334041bgw1epqhs1zejnj20nb0c3myq.jpg)

![http://ww4.sinaimg.cn/large/c334041bgw1epqhsf324ij20vj0j3dib.jpg](http://ww4.sinaimg.cn/large/c334041bgw1epqhsf324ij20vj0j3dib.jpg)> 图像识别因调用接口可能时间较久，可根自身需求修改重试次数变量```count```。
