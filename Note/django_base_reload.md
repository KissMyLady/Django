重定向  
=====

## 图文说明      
![reload-1](https://github.com/KissMyLady/Django/blob/master/Img/reload-1.jpg)    
```Pythno
from django.shortcuts import render
from django.shortcuts import render, redirect
from book_app.models import BookInfo, HeroInfo
from datetime import date


def index(request):
    list = BookInfo.objects.all()
    return render(request, 'html/index.html', {'list':list})


def create(request):
    book = BookInfo()
    print('到达create')
    c =  ['秘密', '嫌疑人x的献身', '白夜行',
          '红手指', '小王子', '三体', '未来的冲击',
          '人类三部曲', '平凡世界', '北京折叠']
    import random
    book.btitle = random.choice(c)
    import time, random
    a1 = (1976, 1, 1, 0, 0, 0, 0, 0, 0)
    a2 = (2018, 12, 31, 23, 59, 59, 0, 0, 0)
    start = time.mktime(a1)
    end = time.mktime(a2)
    for i in range(1):
        t = random.randint(start, end)
        date_touple = time.localtime(t)
        date = time.strftime("%Y-%m-%d", date_touple)
        print(date)
    book.bpub_date = date
    book.save()
    return redirect('/index/')


def delete(request, id):
    book= BookInfo.objects.get(id=int(id))
    book.delete()
    return redirect('/index/')



def detail(request, bid):
    book = BookInfo.objects.get(id=int(bid))
    heros = book.heroinfo_set.all()
    return render(request, 'html/detail.html', {'book':book, 'heros':heros})

```

