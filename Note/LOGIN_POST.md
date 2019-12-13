如何建立自己的网站    
====

## 登录案例  
请注意, 这里的教程是至上而下的学习; 就是说, 先看实际生产代码, 然后在学习底层逻辑    
- [跳转--Ajax讲解]()    
- [跳转--Cookies讲解]()   
- [跳转--Session讲解]()    
  
> 请先看下面的一张图:  
>> ![lo-1](https://github.com/KissMyLady/Django/blob/master/Img/Ajax_Session/lo-1.jpg)  
  
1、当浏览器输入127.0.0.1:8000/login/时, 最先调主文件夹下的urls.py(当然没有这个url地址), 然后再去子项目(APP)  文件里去找url，就是上图  
  
2、找到后，调用views.py里面的函数, 函数判断，调用html文件， 此时我们来到登录页面    
> 当login函数调用时，调用html文件    
> 文件里面有相应表单，会收集信息，判断，然后返回给下一个函数login_check     
>> ![lo-2](https://github.com/KissMyLady/Django/blob/master/Img/Ajax_Session/lo-1.jpg)    
>>  
>>> 语法都很简单, 理解整个过程后， 接下来我将介绍Ajax、Cookies和Session     
  
- [Ajax讲解]()     
- [Cookies讲解]()    
- [Session讲解]()  


## 关于学习的一点建议    
光说不练假把式, 不用例子学编程那是在扯蛋    
什么是学习？   
> 学习就是从有限的例子中找出问题和答案之间的规律的一个过程     
 
为什么学习?    
> 学习是所有生命的支持    
 
学进去的标准-费曼标准:    
> "如果你没有办法用简单的语言表达你所学的知识，你就没有真正学会它"  
  
如何学？    
* 第一步，明确什么是问题，什么是答案;    
* 第二步，学例子;    
* 第三步，验证(重推)。  

