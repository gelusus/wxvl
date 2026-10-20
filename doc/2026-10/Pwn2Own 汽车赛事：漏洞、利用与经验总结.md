#  Pwn2Own 汽车赛事：漏洞、利用与经验总结  
GRCC
                    GRCC  IoVSecurity   2026-10-20 00:00  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/CQb4KERYG3QA0ezCCjgRONQvXCf3wka7je04trwIyMqsDUWBubpwfiahXImiaoia7NnueGomOO28vicSZ5wEFFTa1Q/640?wx_fmt=gif#imgIndex=0 "")  
  
点击上方  
蓝色字体  
，关注我们  
  
**/**  
**技术交流群****/**  
  
添加微信  
13918880149  
\**15021948198(好友已满），申请会员下载ppt & 加入网络信息安全、测试评价、汽车电子、自动驾驶、智能机器人、智能船舶、超级高铁、飞行器、招聘求职、投融资合作群...**  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/WvsNkg6cHNz6VXR01SzOjyAiaPCe5pjf9SeWJEibsvntLkyHpicViacZHJGB6HibbBKdK2VJicGA5NiaklOJAW2DBia65x73KSWKaCiaIia6eJaTucZcU/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=1 "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/WvsNkg6cHNxics5sF78SUG60ztcVr9M4vibUv3hVoGVGX0BiaSGNyEicVdMFZzbfl1LP4iaA2iaNQibC0Yoib1bahaYYDibS1LyH6LEDNRYt2NFzHNPY/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=2 "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/WvsNkg6cHNzIvWcysvhyFxCsjOpxcf0kThb9F4mMGJicQgyKhw0fTJXaUSGmejoR3HgnzNSnWTNwgdEfZeQfhv40ngicV6zzP51VYCibtvP7fo/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=3 "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/WvsNkg6cHNwdTl1Ogr5ViaOZwicvKCaqLEibwcqoxrbQIQOozu99PXp0BBeFgiaBBibS83vMX5WNdwnIYgcyp16RibkWpIRUYSQicJZFPrIjpMn4ZA/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=4 "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/WvsNkg6cHNwow7gaQtziar4bOeFExqHIYficzzk2UxGV9KEtnG8HNvNGHsF6iagz51KzZUPp6Fxmb423tn8ibTn6XzSSBZ8QYgP1awXflSGGpVw/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=5 "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/WvsNkg6cHNwVkq71M3c8ayj5BXfTG1uN8wHnbDicVZQsKZ9TC2T4UoosLZYuL298u7vIaUa4wicej48GSbNgrh9H6UrMvOSTSumQ6YKSTHIec/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=6 "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/WvsNkg6cHNwWaVUpEiaPxpibjyNxnLz2oUeatk0kC3au8cqLb55SLFzttdjwuicfQOlhgpL1Owg7DtKWNWBKBobVvGQKAq8XyLmBq140usiaacQ/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=7 "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/WvsNkg6cHNycyK9oYrRwpFf0BwA4nicVrfoiaAR6gX4Bicc9fSeRQ0aYbs5uibPxp9zaSoZBAaib6DvHRowckde4SWK23fVwZ3HTxgaLzHOJLG4A/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=8 "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/WvsNkg6cHNzv52ZfSm8lsHOERQ4pkj1iauS5icENmztxNHnibZWuDNnOUUdRNhor5Hic5JISRgFApXBtNAjgDZ71E6bGToWy1H2mLsBy91YU47g/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=9 "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/WvsNkg6cHNzJOL2rkBzNcia24hibha9E2sRman8RbssbESVkNicY5tjM2LfyOiaFTAwLFuwq05icLs3IiaDKia8Fv9d6G8J44BDOichianqtulMaNhGA/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=10 "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/WvsNkg6cHNzwWPwaO2zX4vRN7AeRd2t4qANauugxZuQTYPOBrHnef8wjcEK4wpw3s3Fj5ia2st4jPc4LUjaF5MtLwpe9JhHjgc40iagJcAthM/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=11 "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/WvsNkg6cHNwUsjL3oTBZ7Mfq7lsLUzKZowDoukbZUgtVibwCWiaWtI5rDUFiaicw6JoMxSbhOwvVPicF6UEPHnHfJYRK5Eibqp0AFYyv6MrafhltI/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=12 "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/WvsNkg6cHNzaXWws7s1QfX62zqBjicRvKup0SEiafgI2eJmBuicoalTgLfRDicqeaWY9ThzssIttqsMAtsw33Y7tIxIAqb7icRkrczXibAYBILpIs/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=13 "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/WvsNkg6cHNw7gO7ErKyLT03Qricw9ia95ANIsnIRTZytq3nhZgBgxtdLKUFbKrW8Q2skq4uJJKfxcOS5mw3ygtDjPQ5WpI7W2dNVLRD8huialI/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=14 "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/WvsNkg6cHNx1RWoUebcKBzQiaMiaRPN9Gwh2IvpI9TLkDf7UlnCHTzVKBFMXJDyAEsIWWu6tH5fDdOBic5MffxKXbChCHwd8wObA5CWrP0Ww6g/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=15 "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/WvsNkg6cHNyvfj302XVjTnwTcPAexggcyD6x8l7Wf6eptJJVYibPyXHMX9O5ozdkRfe8QNdMphTeuJ6mIKPk3HrygkoKO6FEDgBaJiavAu3AQ/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=16 "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/WvsNkg6cHNwNW1urwJsWD9OAGCAo6ErCKrDccTkkiciaw9qcOBbzhDd4rUIheX0eaCaaAINvs45bc7bhSx7FTEp3tQpcadB9VMiaYoMTZbBMFs/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=17 "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/WvsNkg6cHNzmmQnn9nDIcWysLyvrbs2r2pdUwzz7s8NHr3PWsbicQIQIbTZMzlw9X7Yibtj3Y1ullITmFhWX9eHScjH43IRGrSEqdZVBKtOjI/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=18 "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/WvsNkg6cHNwL0T9kWp5YEibD6p8fbvIQDzECvrjSuloat2e1icVB8cJCcg9ibhIAydCegWRyhHEY4ibNWTsHhOgdicpx3rriaib1K3ZxkVvz1GUbX0/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=19 "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/WvsNkg6cHNycgiblU6By36n8cVoG9KunWk797NrrGRMK98SzwwVj1YyIXsVgLia0CHHY46IbbgJ8tY2zEXK4McJo7avU4z7zPaNLDHoV27Sbw/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=20 "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/WvsNkg6cHNx4R3NPTicZltcZZnmBj6Smo3qnsNRnV4P6IpKfupXoKiaHcuqfxlDxkzKGb7PbD0Q0icOKwWVShUUHkia6ibzRI5HPZiaTZHSYpCP10/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=21 "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/WvsNkg6cHNzibRrDV4SSkJC7jYHWWe74xgxtgvIH3b4FalxATos35COqjlDoA7RjpEx3Zicp5HTWHIR112yLXDYAuBQKugNRlibGicsiaicokgd8U/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=22 "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/WvsNkg6cHNyZ4dVh2oDgvC8RLPBvH13aOs5ufHhJC2kPnUoB4TAyj4SJGbRIyEfI60Ishzvh88O0cHcViavdSaOpp0nnCIwt30GpUeS02ibs4/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=23 "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/WvsNkg6cHNwWZyH8HPmWCorRpq3UCFJCdb0YUNwuyNouKHiaq0fLe8p0ZvB2VfiagtUBCb58hJLxlPUNgzkXqaxibRMzc55PrE0u47adU0yl7o/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=24 "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/WvsNkg6cHNx6v5J7na3JFg9QcAU7dXMzl1kITgObjUgibIDhEicFW5ZWvdywVP1bbKrYSiaZBHmQGXksctM8Yvtj4Fyt0RfaOtQfNJ2fCoZD90/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=25 "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/WvsNkg6cHNwWU0OujyS9kULXq6qZibql6OxxOPX80BibDtYVPczguOT8FegLlgAJa7Xzc0Hia78mO80kOQNdx8BV5HoslxTnoD4I03b1YpGQ5k/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=26 "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/WvsNkg6cHNyvP1Sz2ZQ6P0wQRiavuICvdT4ev7iacciakrFfibXiaWCp9Sv5ppKlaLuFRvZaNCHMwY6REiaxyZLnE4Szc3N2xaG3MiceyLUOLBsV9Y/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=27 "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/WvsNkg6cHNxjGOweyd4JibUHcwQUicVNQHX2MibycibRZiap79gY7paCGlKCx79YdYV7KHWcSJzFXiaibZnMGqrYB4vH2BfnXaib4tS2p0WNpFFhO8A/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=28 "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/WvsNkg6cHNwUydey92Ckiawia5vrOha17VYVUWvtxlLibfE5cr2GTazJSx1xAH17tySYszCaa8TPelVicAdVmx7v7sR4ic3S8N8X99rf7ibgtOuE4/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=29 "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/WvsNkg6cHNwV0UuXuzSj9q8TlkVWRic45ic79I4MxAxkunVeyvgq5ibQZGrV1iahRiat6XCPU7038VADSv2ibl5nRFvZat42xlYp5JIZQSqGBV2K0/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=30 "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/WvsNkg6cHNzyjeS3hyjPA3gOe9DKEaTvTb1OJ7uCBydNY0EibFgg07TyCpIHadIeibdQETNoJfOiaQ0mZNRIMp7tH7EqDRrYvAb2nHuYCRo8icg/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=31 "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/WvsNkg6cHNy4nKs5JPsm09Z5Z8N0qVzOdswLcdKdsTlHjDaXfQa8OMgvGLobwVlNiaymKZnRYW21onrPT4lhd6qIftmgh4sibKNAIurrJSGjc/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=32 "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/WvsNkg6cHNxLNYKQhgr7eAIU7C745WMkWMmZwLfq5glNWiaTicRZjuB8lFjiapibl9v89Xq4iaJU3A0DuxibNrWXYmurKrpiagZZRBpwic8L3XEJKCo/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=33 "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/WvsNkg6cHNyeEyeUtNs0Z8c0ul3Mb1KJKOyn2rysu6kbtBdfhicMw9HSZBqiaLpbfUS7jNJDjFWicNae3VicbcsGSYzUfiaCaWRb1pFg0MZawwAo/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=34 "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/WvsNkg6cHNxicSxEhjUr3cPERESKWMniccT7OxG1Hsq8LvWfmSJwjtr8aBJibcLIIdfZWhtvNbArcibqZicjBUb2gkxTVZZjrURPb17nN4qxJKMM/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=35 "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/WvsNkg6cHNzCIa88od9wcwsGBgnOBSd0Y6ePITpcKRibYHYS005BiabQzWOfdjQQqaCDqVQCibrSWROCHxhvXpxWKJtJQmnFQKwaMx8oQACcQA/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=36 "")  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/b96CibCt70iabwjyojLhA03PtxUnkNPREnt2F48ywfXLpDdDAjicOTPI8Q94tVLbJ58tbRs12iaXDKhUOW9gd4NlFA/640?wx_fmt=gif#imgIndex=37 "")  
  
相关文章  
  
# 整车信息安全设计对车企的挑战与一汽奔腾的应对  
# 智能网联汽车安全测评的中国方案与实践  
# 国强标下智能网联汽车信息安全检测解决方案  
#   
  
![](https://mmbiz.qpic.cn/mmbiz_gif/MfTd6rd9CyvNRMW8I9cvI1CK5gKiaYqg2veTn9t9dAe1GxYic7pAvgvRIKNFickConFyX8AvW2reAq8GchJI6aBpA/640?wx_fmt=gif&wxfrom=5&wx_lazy=1&tp=webp#imgIndex=38 "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/WvsNkg6cHNzNY8CmicuhgtyPx1PLL1NOwv1ExPIe7z2r0Qo75qPLQkXmLSMoniaTFbdd1mfUbSyImqlCuKEau7tMqvbwsuMdBCf5sQh4EysRE/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=39 "")  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/WvsNkg6cHNw1TS1MbiaAUZUxwVxhXrkDpMIdicxlfFE6olg1ia9BVTy2dVbzKicMicavwib7IMibDqnfZexXMZWibXHJbLA0ouMj4DXQPDtqMQTXzWQ/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=40 "")  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/WvsNkg6cHNwXE2uQjPpYbnUHRotK0guaCRJodAEhl8DkNSPkA6I7HxrP8g8sXICu7EERsRfXnfWIdx6tbfiagx4ibqr7T2IQia3vL7PB8Dn238/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=41 "")  
  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/WvsNkg6cHNwGCGVo8OZnTkf3jBGEWqPuMZuMhbzngn0CzM2y8aA9UqskDa9V25ZFTrxDRjr5TWeMOawWSTicLayNXbdEjuTpMpvgiadia8j3Zk/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=42 "")  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/8Pvibnf7ic0cy77VtN8ibA7XuZgvGQoicjpar7CWkfIEXV4CEjiankS0tjDZEUgxhNHf0HicpBNcO4YuhOm5eIdb7RaA/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=43 "")  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/9yhibG49kQicogTWBZcB6XwgTib9lH6QN57pFdZwoRicFbc3JLM7icu8hadyzRKztBHGZ7eDEVgMiaHYqExfhbbpb5vA/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=44 "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/WvsNkg6cHNxTM4koEqbuI1FWIEqlicNMW0CC4KdeBpXrzETqg3hk0FJ0v9BPDuLsJZMJP8ewqC0WBdMc0pTXhYibSibDnd5nlsE2ZOaheMRtPE/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=45 "")  
  
扫描二维码**申请**  
会员**下载ppt**  
 &   
  
加入行业交流群  
  
获取合作机会  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/kuhNyShuqyAGSIk680L6OHthYzkwuUDkKqfw3icohb1JLrEvjicKgfaiatIDP1L7RN7zPQkzbrksWzTMmgh5LKjzA/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=46 "")  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/uTSIm9RGwm0ibSggKRaicPibLl2nXk3lGdgeoXo0P9Xy8e2aNHPm3LOhKjicHk2zhB5V1ar3CwUTs258UkiaTPYq4gw/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=47 "")  
  
扫描二维码获取  
  
更多精彩  
  
![](https://mmbiz.qpic.cn/mmbiz_png/XiacM3aibSNia0qvdL1PUiaZugASarnXx5wAxT5ic13sgRB49E67AsdWeZpHnibUEW2oibToqEWRjHmImztgv33MaknnQ/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=48 "")  
  
  
关注“**IoVSecurity**  
”  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/fBQwicMRtG3qyicHcTibNaG9RMs2E8knzWpfH0gnibzKsciaBTYdnW8mFyNgvEAqBNoib29iasxMgwh2gWRSIkINyHVLA/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=49 "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/D7nIuxbSmauhlzDVRGHTibAGyGcFvY5qFSPyZdMCxTSXwjhzFTotRe6rciaIxatoAHF0MPI73MMPAbf0UUMIMSvw/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=50 "")  
  
点个在看  
你最好看  
  
  
  
