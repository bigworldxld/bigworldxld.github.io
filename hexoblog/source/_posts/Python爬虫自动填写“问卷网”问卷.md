---
title: Python爬虫自动填写“问卷网”问卷
date: 2022-03-02 13:58:28
tags: python
categories: python
img: /images/python1.jpg
---

# 前言

征集问卷，但很难找到渠道迅速收集大量样本，如果是自己通过“问卷网”设计的问卷可以在设置不锁IP（默认情况）下用本方法快速刷取大量样本

## 1.下载驱动

首先你要去 这里: [chome驱动下载传送门](http://chromedriver.storage.googleapis.com/index.html). 下载好你对应chome浏览器版本的chome驱动.
如果没有谷歌浏览器也可以下载ie/火狐等其它驱动，驱动必须与本地浏览器版本匹配，再对程序稍作修改即可。

## 2.先附上我改之后的完整代码

```python
from selenium import webdriver
import random
import time

option = webdriver.ChromeOptions()
option.add_argument('headless')
url = 'https://www.wenjuan.com/s/UZBZJvWVObd/'
t = 200
# 设置提交问卷次数
path = r"G:\\chromedriver.exe"

for times in range(t):
    # 不自动关闭浏览器
    option = webdriver.ChromeOptions()
    option.add_experimental_option("detach", True)
    driver = webdriver.Chrome(executable_path=path, options=option)
    driver.get(url)
    # 定位所有的问卷问题
    questions = driver.find_elements_by_css_selector('.matrix')
    for index, answers in enumerate(questions):

        # 定位所有问卷问题选项
        answer = answers.find_elements_by_css_selector('.init_option')
        # 定位需要填写文字的选项，并填入相关内容
        # if not answer:
        #     blank_potion = answers.find_element_by_css_selector('.blank.option')
        #     blank_potion.send_keys('很好')
        #     continue
        # 专项选择题处理
        if index == 0:
            choose_ans = answer[random.randint(0, 1)]
            choose_ans.click()
            time.sleep(random.randint(0, 3))

        elif index == 1:
            choose_ans = answer[1]
            choose_ans.click()
            time.sleep(random.randint(0, 3))
        elif index == 3:
            choose_ans = answer[2]
            choose_ans.click()
            time.sleep(random.randint(0, 3))
        elif index == 4:
            choose_ans = answer[0]
            choose_ans.click()
            time.sleep(random.randint(0, 3))
        elif index == 5:
            choose_ans = answer[0]
            choose_ans.click()
            time.sleep(random.randint(0, 3))
        elif index == 9:
            choose_ans = answer[0]
            choose_ans.click()
            time.sleep(random.randint(0, 3))

        elif index == 6 :
            choose_ans = answer[1]
            choose_ans.click()
            time.sleep(random.randint(0, 4))
        elif index == 7 :
            choose_ans = answer[0]
            choose_ans.click()
            time.sleep(random.randint(0, 4))
        elif index == 8:
            choose_ans = answer[0]
            choose_ans.click()
            time.sleep(random.randint(0, 4))
      # 多选题处理
        elif index == 2 or index == 10:
            for i in range(1, random.randint(2, 4)):
                choose_ans = answer[random.randint(0, 3)]
                choose_ans.click()
                time.sleep(random.randint(0, 5))

        # 双选题处理
        # else:
        #     choose_ans = random.choice(answer)
        #     choose_ans.click()
        #     time.sleep(random.randint(0, 1))
    subumit_button = driver.find_element_by_css_selector('#next_button')
    subumit_button.click()
    print('已经成功提交了{}次问卷'.format(int(times) + int(1)))
    # 延迟问卷结果提交时间，以免间隔时间太短而无法提交
    time.sleep(4)
    driver.quit()
```

## 3.代码解释

driver = webdriver.Chrome(executable_path=path,options=option)
这里去除options可以在运行程序时弹出谷歌浏览器界面，方便查看爬虫运行的各个动作，确认无误后再加上可以方便后台自动刷问
path=r"G:\\chromedriver.exe"
path填写的是你下载的浏览器驱动的位置
url填写自己的问卷界面的链接
程序中for index,answers in enumerate(questions):即开始遍历问卷中的每道题目，if index=...即限制对应序号的题目进行特殊操作，例如限制选项范围和数量等
random.randint(0, 3)即生成0-3的随机数，原来随机的选择选项，范围可以自己调节
个人建议，可以在程序中按照自己的需求直接预先选好每道题的答案，再设置刷问卷的次数即开始运行
如要丰富数据，则修改预设答案和提交次数，程序开始运行代码

## 4.最后要注意：

1. 可能会出现自动填漏题的现象，正常。大部分都不会漏，放心。
   如果想保证它不漏题，可以每个问题都设置一个else if确保万无一失。

2. 可能会出现一开始把代码已经运行到可以自动答题了，然后突然又不可以了，可能是因为你的chome浏览器自动升级导致和chome驱动不匹配，最直接的方法就是再去下载一个对应的驱动
   