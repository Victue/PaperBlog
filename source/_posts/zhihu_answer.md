---
title: 知乎，爬虫，x-zse-86 2.0
author: Li Pie
categories: 摸鱼
tags:
- requests
- zhihu
- spider
date: 2021-03-30
---

上一次使用selenium爬知乎，太慢。并且不知道为啥，上次使用的方法已经失效，用户回答无法显示，可能是由于知乎升级了反爬机制。

这一次使用requests，正经的爬一次。

显然可知，用户回答api为

```
https://www.zhihu.com/api/v4/members/people/answers?include=data[*].is_normal,admin_closed_comment,reward_info,is_collapsed,annotation_action,annotation_detail,collapse_reason,collapsed_by,suggest_edit,comment_count,can_comment,content,editable_content,attachment,voteup_count,reshipment_settings,comment_permission,mark_infos,created_time,updated_time,review_info,excerpt,is_labeled,label_info,relationship.is_authorized,voting,is_author,is_thanked,is_nothelp,is_recognized;data[*].vessay_info;data[*].author.badge[?(type=best_answerer)].topics;data[*].question.has_publishing_draft,relationship&offset=0&limit=20&sort_by=created
```

json格式

![zhihu_answer_1](https://cdn.jsdelivr.net/gh/Victue/PaperBlog@main/source/images/zhihu_answer/zhihu_answer_1.png)

三个变量，people，offset，limit

直接获取这个json即可

requests(api)出现403

![zhihu_answer_2](https://cdn.jsdelivr.net/gh/Victue/PaperBlog@main/source/images/zhihu_answer/zhihu_answer_2.png)

好家伙，header改了很多都是403

分析分析

关键在于这个x-zse-86

![zhihu_answer_3](https://cdn.jsdelivr.net/gh/Victue/PaperBlog@main/source/images/zhihu_answer/zhihu_answer_3.png)

![zhihu_answer_4](https://cdn.jsdelivr.net/gh/Victue/PaperBlog@main/source/images/zhihu_answer/zhihu_answer_4.png)

y.set('x-zse-86', '2.0_' + j)

![zhihu_answer_5](https://cdn.jsdelivr.net/gh/Victue/PaperBlog@main/source/images/zhihu_answer/zhihu_answer_5.png)

b.set('x-zse-86', '2.0_' + E)

![zhihu_answer_6](https://cdn.jsdelivr.net/gh/Victue/PaperBlog@main/source/images/zhihu_answer/zhihu_answer_6.png)

![zhihu_answer_7](https://cdn.jsdelivr.net/gh/Victue/PaperBlog@main/source/images/zhihu_answer/zhihu_answer_7.png)

signature: (0, o.default) ((0, r.default) (p))

signature: (0, o.default) ((0, r.default) (d))

打个断点看看d是啥

![zhihu_answer_8](https://cdn.jsdelivr.net/gh/Victue/PaperBlog@main/source/images/zhihu_answer/zhihu_answer_8.png)

![zhihu_answer_9](https://cdn.jsdelivr.net/gh/Victue/PaperBlog@main/source/images/zhihu_answer/zhihu_answer_9.png)

d值为

"3_2.0+

/api/v4/members/MarryMea/answers?include=data%5B*%5D.is_normal%2Cadmin_closed_comment%2Creward_info%2Cis_collapsed%2Cannotation_action%2Cannotation_detail%2Ccollapse_reason%2Ccollapsed_by%2Csuggest_edit%2Ccomment_count%2Ccan_comment%2Ccontent%2Ceditable_content%2Cattachment%2Cvoteup_count%2Creshipment_settings%2Ccomment_permission%2Cmark_infos%2Ccreated_time%2Cupdated_time%2Creview_info%2Cexcerpt%2Cis_labeled%2Clabel_info%2Crelationship.is_authorized%2Cvoting%2Cis_author%2Cis_thanked%2Cis_nothelp%2Cis_recognized%3Bdata%5B*%5D.vessay_info%3Bdata%5B*%5D.author.badge%5B%3F(type%3Dbest_answerer)%5D.topics%3Bdata%5B*%5D.question.has_publishing_draft%2Crelationship&offset=0&limit=20&sort_by=created+

"ALAd3ohjFRKPTjNZeG5_W8KUwFVpuCzndOk=|1603453023""

最后字符串为cookie中d_c0的值

简单思索了一下，使用md5加密这一长串试试，巧了，正好一样

![zhihu_answer_10](https://cdn.jsdelivr.net/gh/Victue/PaperBlog@main/source/images/zhihu_answer/zhihu_answer_10.png)

第一步加密

![zhihu_answer_11](https://cdn.jsdelivr.net/gh/Victue/PaperBlog@main/source/images/zhihu_answer/zhihu_answer_11.png)



第二步知乎js加密

https://github.com/hellokuls/python-course/blob/master/zhihu/g_encrypt.js



大功告成，可以去写爬虫收集你心爱用户的回答了。



```python
    # 加密
    def encrypt(self, d):
        d_md5 = hashlib.new('md5', d.encode()).hexdigest()
        with open('zhihu1.js', 'r') as f:
            ctx = execjs.compile(f.read(), 'C:/Users/hasee/node_modules')
        encrypt_d = ctx.call('b', d_md5)
        return encrypt_d
```

```python
    # api地址以及未加密文本
    def address(self):
        hash_url = self.hash_url.format(self.offset)
        url = ''.join([self.main_url, hash_url])
        self.d = '+'.join(['3_2.0', hash_url, self.d_c0])
        return url, self.d
```

```python
    # 获取内容,answer包含20个item
    def get(self):
        url, d = self.address()
        encrypt_d = self.encrypt(d)
        headers = {
            'user-agent': self.User_Agent,
            'cookie': self.cookie,
            'x-zse-83': '3_2.0',
            'x-zse-86': '2.0_%s' % encrypt_d
        }

        r = requests.get(url=url, headers=headers)
        answer = json.loads(r.text)
        return answer
```

answer就是一个json文件，从中可以很简单找到需要的内容

```python
# 问题内容
answer['data'][i]['question']['title']
# 问题id
answer['data'][i]['question']['id']

# 回答内容
answer['data'][i]['content']
# 回答id
answer['data'][i]['id']

# 创建时间
answer['data'][i]['created_time']
# 修改时间
answer['data'][i]['updated_time']

# 点赞数
answer['data'][i]['voteup_count']
# 评论数
answer['data'][i]['comment_count']
```

将json文件存起来，这里使用mongodb

```python
    # save json
    def write_database(self):
        answer = self.get()
        data = {
            "name": "zhihu_a",
            "content": answer
        }
        self.col.insert_one(data)
```



结束。拿着数据去分析吧