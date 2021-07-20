---
title: 使用Telegram接受微信和QQ的消息
date: 2021-07-20 09:21:31
tags: 瞎搞
---
**暂时放弃了吧，似乎微信团队又做了什么奇怪的东西导致又失效了**
虽然现在手机内存够大够用，但对于微信常驻内存这件事也是颇有微词。前几天接触到 [ehForwarderBot](https://github.com/ehForwarderBot) 这个项目，应该是转发微信的消息到TG，并且能够回复。大致浏览了一下，应该是基于网页微信的接口，但是网页微信新的微信号已经不能登录了：
```
<error><ret>1203</ret><message>为了你的帐号安全，此微信号已不允许登录网页微信。你可以使用Windows微信或Mac微信在电脑端登录。Windows微信下载地址：https://pc.weixin.qq.com  Mac微信下载地址：https://mac.weixin.qq.com</message></error>
```
<!-- more -->
微信团队为了适配 [统信UOS](https://www.chinauos.com/) 系统，在 Electron 中嵌套了一个微信网页版，但是这个网页版是能够登录的，好像是添加了一个请求头信息: [wechaty-puppet-wechat:127](https://github.com/wechaty/wechaty-puppet-wechat/issues/127)
```python
UOS_PATCH_CLIENT_VERSION = '2.0.0'
UOS_PATCH_EXTSPAM = 'Gp8ICJkIEpkICggwMDAwMDAwMRAGGoAI1GiJSIpeO1RZTq9QBKsRbPJdi84ropi16EYI10WB6g74sGmRwSNXjPQnYUKYotKkvLGpshucCaeWZMOylnc6o2AgDX9grhQQx7fm2DJRTyuNhUlwmEoWhjoG3F0ySAWUsEbH3bJMsEBwoB//0qmFJob74ffdaslqL+IrSy7LJ76/G5TkvNC+J0VQkpH1u3iJJs0uUYyLDzdBIQ6Ogd8LDQ3VKnJLm4g/uDLe+G7zzzkOPzCjXL+70naaQ9medzqmh+/SmaQ6uFWLDQLcRln++wBwoEibNpG4uOJvqXy+ql50DjlNchSuqLmeadFoo9/mDT0q3G7o/80P15ostktjb7h9bfNc+nZVSnUEJXbCjTeqS5UYuxn+HTS5nZsPVxJA2O5GdKCYK4x8lTTKShRstqPfbQpplfllx2fwXcSljuYi3YipPyS3GCAqf5A7aYYwJ7AvGqUiR2SsVQ9Nbp8MGHET1GxhifC692APj6SJxZD3i1drSYZPMMsS9rKAJTGz2FEupohtpf2tgXm6c16nDk/cw+C7K7me5j5PLHv55DFCS84b06AytZPdkFZLj7FHOkcFGJXitHkX5cgww7vuf6F3p0yM/W73SoXTx6GX4G6Hg2rYx3O/9VU2Uq8lvURB4qIbD9XQpzmyiFMaytMnqxcZJcoXCtfkTJ6pI7a92JpRUvdSitg967VUDUAQnCXCM/m0snRkR9LtoXAO1FUGpwlp1EfIdCZFPKNnXMeqev0j9W9ZrkEs9ZWcUEexSj5z+dKYQBhIICviYUQHVqBTZSNy22PlUIeDeIs11j7q4t8rD8LPvzAKWVqXE+5lS1JPZkjg4y5hfX1Dod3t96clFfwsvDP6xBSe1NBcoKbkyGxYK0UvPGtKQEE0Se2zAymYDv41klYE9s+rxp8e94/H8XhrL9oGm8KWb2RmYnAE7ry9gd6e8ZuBRIsISlJAE/e8y8xFmP031S6Lnaet6YXPsFpuFsdQs535IjcFd75hh6DNMBYhSfjv456cvhsb99+fRw/KVZLC3yzNSCbLSyo9d9BI45Plma6V8akURQA/qsaAzU0VyTIqZJkPDTzhuCl92vD2AD/QOhx6iwRSVPAxcRFZcWjgc2wCKh+uCYkTVbNQpB9B90YlNmI3fWTuUOUjwOzQRxJZj11NsimjOJ50qQwTTFj6qQvQ1a/I+MkTx5UO+yNHl718JWcR3AXGmv/aa9rD1eNP8ioTGlOZwPgmr2sor2iBpKTOrB83QgZXP+xRYkb4zVC+LoAXEoIa1+zArywlgREer7DLePukkU6wHTkuSaF+ge5Of1bXuU4i938WJHj0t3D8uQxkJvoFi/EYN/7u2P1zGRLV4dHVUsZMGCCtnO6BBigFMAA='

headers = {
  'client-version' : UOS_PATCH_CLIENT_VERSION,
  'extspam' : UOS_PATCH_EXTSPAM,
  'referer' : 'https://wx.qq.com/?&lang=zh_CN&target=t'
}
```
这样的话，最初想用微信作为通知机器人的想法似乎也可以实现了 [ItChat](https://github.com/luvletter2333/ItChat)。

继续 EFB 的功能，发现TG需要一直科学上网
* efb服务器是否需要科学上网
* tg客户端是否需要一直科学上网

-- waiting --