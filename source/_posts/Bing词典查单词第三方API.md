---
title: Bing词典查单词第三方API
categories:
  - 计算机
abbrlink: 57344
date: 2018-03-10 01:01:48
tags:
author: 徐梓涵
---

利用简单的请求，获得详细的中英释义及例句

因为自己建立了一个电子生词本，经常需要在Excel表格中使用到查单词的功能。为了方便，开发了这样的一个简单的API。

本文即介绍这个API的调用方式以及在Excel中如何具体调用。

<!-- more -->

## API调用方式（若没有技术背景可直接看后文在Excel中直接使用）
为便于调用，API采用了非常简便的Get方式调用。只需要一个参数`word`即可。

如下是一个请求示例：
```null
https://www.fenzhengrou.wang/engtools/dic?word=hello
```

返回的数据也非常简单，是一个Json字串：
```json
{
"headword": "hello",
"meanings": {
"int.": "你好；喂；您好；哈喽",
"网络": "哈罗；哈啰；大家好"
},
"pronunce": ["美[heˈləʊ] ", "英[həˈləʊ] "],
"sample": {
"H is for hello , whichis something nice to say , it means I 'm glad to see you , any time of a day .": "H代表你好，一句彬彬有礼的问好，意思是我在一天的任何时候都很高兴见到你。",
"He saidto say hello and he hopes we 'll seeyoufor dinner soon , but he 's got too much work togo out pub crawlingwith the boys .": "他说让我问好，还说希望我们三个最近能在一起吃顿午饭，但是他要干的活儿实在太多了，所以没时间和咱们在酒吧里鬼混了。",
"Hello ! How much should I pay ?": "你好！我要交多少钱？",
"Hello , Sergio . What'sthematter ? - - I 'm feeling terrible . --Oh , dear ! Why don't you see adoctor ? -- Perhaps I will .": "你好，塞吉尔。你怎麽了？--我感觉很难受。--哎呀！你怎麽不去看医生？--也许我就去。",
"It was OK . Thank you for coming out to meet me eventhough you are busy . Oh yeah , Andyasked me tosay hello .": "没关系。你很忙还来接我真是谢谢。对了，还有安迪向你问好。",
"Or if oneof our cats comes uptosay hello , rubs itself on my leg and settles down for a nap near me . . . yep : great day!": "再或者，如果我们的某只猫走过来跟我打招呼，再拿身子在我腿上蹭，然后靠近我躺下来打个盹儿…没错：奇妙无比的又一天！",
"Thankful fora familiar face in the crowd I mademywaytoward her to say hello .": "感谢能在人群里见到熟悉的脸庞，我走过去，跟她打招呼。",
"The thing that annoys me about him is theway he never says\" Hello !\" .": "他使我讨厌的是，从来不跟别人打呼。",
"Though I was a bit underdressed in apair of khaki pants and a damp leather jacket I decided to step in and say hello to her .": "虽然我当时穿着有些寒酸——黄褐色的裤子和潮湿的皮夹克，但我还是决定要走进去和她打个招呼。",
"When she greetedmewithher hello , I replied \" Guess what I need from you today ? \"": "她向我打招呼，我说：你说我想弄点什么呢？"
},
"tabs": {
"反义词": ["int.", "ciao", ",", "good afternoon", ",", "good day", ",", "good morning", ",", "hi"],
"同义词": ["int.", "goodbye"],
"搭配": ["v.+n.", "say Hello"]
}
}
```

剩下的，就是直接解析出来使用了。

## 在Excel中使用
在2013版本之后，Excel中加入了`WEBSERVICE`函数。若不想使用这里提到的自定义函数，可以直接使用WEBSERVICE函数。

最终效果如下：

![](https://i.loli.net/2018/02/20/5a8bf47280c01.png)

### 1.文档设置
首先我们需要把我们的Excel文档另存为xlsm格式，因为这样的格式可以加入自定义的函数。具体可以直接在另存为中选择`Excel启用宏的工作簿 *.xlsm`

然后，我们在`文件-&gt;选项-&gt;自定义功能区`的中找到**开发工具**并勾选。最终效果如图所示：

![](https://i.loli.net/2018/02/20/5a8bf59f91908.png)

### 2.启用正则表达式
我们点击`查看代码`

![](https://i.loli.net/2018/02/20/5a8bf64c6441e.png)

在弹出的界面中选择`菜单-&gt;工具-&gt;引用`并勾选`Microsoft VBScript Regular Expressions 5.5`(若没有该选项，可以尝试勾选其他的最新版，切记勿勾选多个版本）然后单击`确定`

### 3.建立自定义函数
看到窗口的左面，有类似的如下界面（可能有些许不同，但不影响使用）

![](https://i.loli.net/2018/02/20/5a8bf760d15f6.png)

在`Microsoft Excel对象`处单击右键选择`插入-&gt;模块`，双击创建出来的该模块，并在代码框中复制上以下内容：

```vb

Function LookupWord(word)
If Len(word) < 1 Then
LookupWord = ""
Exit Function
End If
On Error Resume Next
With CreateObject("Microsoft.XMLHTTP")
.Open "GET", "https://www.fenzhengrou.wang/engtools/dic?word=" & word, True
.Send
Do Until .ReadyState = 4
DoEvents
Loop
LookupWord = .responsetext
End With
End Function

Function regular_find(str, pat)
Dim mRegExp As RegExp
Dim mMatches As MatchCollection '匹配字符串集合对象
Dim mMatch As Match '匹配字符串

Set mRegExp = New RegExp
With mRegExp
.Global = True
.IgnoreCase = False
.Pattern = pat
Set mMatches = .Execute(str)
End With

Dim ret
ret = ""

For Each mMatch In mMatches
ret = ret & mMatch.Value
Next
regular_find = ret
End Function

Function regular_find_all(str, pat)
Dim mRegExp As RegExp
Dim mMatches As MatchCollection '匹配字符串集合对象
Dim mMatch As Match '匹配字符串

Set mRegExp = New RegExp
With mRegExp
.Global = True
.IgnoreCase = False
.Pattern = pat
Set mMatches = .Execute(str)
End With

Dim ret
ret = ""

For Each mMatch In mMatches
ret = ret & "||||" & mMatch.Value
Next
regular_find_all = Split(ret, "||||")
End Function

Function GetPronunciation(ori_data)
Dim pronunce_part
Dim pronunce
pronunce_part = regular_find(ori_data, "(pronunce[\s\S]*?\],)")
pronunce = regular_find(pronunce_part, "(\[""[\s\S]*?""\])")
pronunce = Replace(pronunce, "[""", "")
pronunce = Replace(pronunce, """]", "")
pronunce = Replace(pronunce, """", "")
pronunce = Replace(pronunce, " ,", Chr(10))
GetPronunciation = pronunce
End Function

Function GetMeanings(ori_data)
Dim meanings_part
meanings_part = regular_find(ori_data, "(meanings"":\{[\s\S]*?\})")
meanings_part = regular_find(meanings_part, "(\{[\s\S]*?\})")
meanings = regular_find_all(meanings_part, "(""[\s\S]*?"")")

Dim ret
ret = ""
For i = 1 To UBound(meanings) Step 1
If ((i - 1) Mod 2) = 0 And (i - 1) <> 0 Then
ret = ret & Chr(10)
End If
ret = ret & Mid(meanings(i), 2, Len(meanings(i)) - 2)
If ((i - 1) Mod 2) = 0 Then
ret = ret & ":"
End If
Next

GetMeanings = ret

End Function

Function GetSamples(ori_data, max_sample)
Dim samples_part
samples_part = regular_find(ori_data, "(sample"":\{[\s\S]*?\})")
samples_part = regular_find(samples_part, "(\{[\s\S]*?\})")
samples_part = Mid(samples_part, 3, Len(samples_part) - 2)
samples_part_arr = Split(samples_part, """,""")
If UBound(samples_part_arr) + 1 < max_sample Then
max_sample = UBound(samples_part_arr) + 1
End If
ret = ""
For i = 0 To max_sample - 1 Step 1
samples_zip = Split(samples_part_arr(i), """:""")
If UBound(samples_zip) = 1 Then
ret = ret & (i + 1) & ". " & samples_zip(0) & Chr(10) & samples_zip(1) & Chr(10)
End If
Next
GetSamples = ret
End Function

Function GetSyn(ori_data)

End Function

```
保存，然后回到Excel的主窗口。

### 4.使用自定义函数
现在，我们就可以使用刚刚设置好的函数了。
例如如下是获得一个单词的释义到当前单元格。这里的A1是存放单词的单元格。
```excel
=GetMeanings(LookupWord(A1))
```
例如如下是获得一个单词的例句到当前单元格。这里的A1是存放单词的单元格，2为获得例句的最大数量。
```excel
=GetSamples(LookupWord(A1),2)
```
例如如下是获得一个单词的读音到当前单元格。这里的A1是存放单词的单元格。
```excel
=GetPronunciation(LookupWord(A1))
```
当前，在设置好以后，单元格默认是不会自动换行的，我们需要将其设置为自动换行，才能获得更好的显示效果

![](https://i.loli.net/2018/02/20/5a8bfafdbf861.png)

如此，便大功告成。只需要使用如上示例中的三个函数，就能获得单词的发音、释义、例句信息。

### 5.特别注意
如果在打开Excel后，出现了如下提示，请务必选择`启用内容`。否则函数将无法计算。

![](https://i.loli.net/2018/02/20/5a8bfb5415b8a.png)

**由于Excel会在每次打开的时候尝试重新计算函数结果，所以建议在获得完释义以后将函数运行的结果单独复制成文本，而非保留公式。**

## 其他调用方式
如果希望使用其他的程序，例如通过编写程序来实现调用。只需要参考第一节来使用Get请求获得结果即可。

## 声明
本API的数据均来自微软Bing词典，仅限于个人非商业性质地使用，若用户将该API用于不正当用途产生侵权或其他法律纠纷，与笔者无关。