#  【Web刷题合集】9道Web题目复现与详细分析  
Lovn.🍀
                    Lovn.🍀  三社院信息Sec   2026-11-02 07:12  
  
## 考点汇总表  
  
<table><thead><tr><th style="color: rgb(0, 0, 0);font-size: 15px;line-height: 1.5em;letter-spacing: 0em;text-align: left;font-weight: bold;background: none left top / auto no-repeat scroll padding-box border-box rgb(240, 240, 240);height: auto;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;padding: 5px 10px;min-width: 85px;"><section><span leaf="">题号</span></section></th><th style="color: rgb(0, 0, 0);font-size: 15px;line-height: 1.5em;letter-spacing: 0em;text-align: left;font-weight: bold;background: none left top / auto no-repeat scroll padding-box border-box rgb(240, 240, 240);height: auto;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;padding: 5px 10px;min-width: 85px;"><section><span leaf="">题目名称</span></section></th><th style="color: rgb(0, 0, 0);font-size: 15px;line-height: 1.5em;letter-spacing: 0em;text-align: left;font-weight: bold;background: none left top / auto no-repeat scroll padding-box border-box rgb(240, 240, 240);height: auto;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;padding: 5px 10px;min-width: 85px;"><section><span leaf="">核心考点</span></section></th><th style="color: rgb(0, 0, 0);font-size: 15px;line-height: 1.5em;letter-spacing: 0em;text-align: left;font-weight: bold;background: none left top / auto no-repeat scroll padding-box border-box rgb(240, 240, 240);height: auto;border-style: solid;border-width: 1px;border-color: rgba(204, 204, 204, 0.4);border-radius: 0px;padding: 5px 10px;min-width: 85px;"><section><span leaf="">难点/绕过点</span></section></th></tr></thead><tbody><tr style="color: rgb(0, 0, 0);background-attachment: scroll;background-clip: border-box;background-color: rgb(255, 255, 255);background-image: none;background-origin: padding-box;background-position-x: left;background-position-y: top;background-repeat: no-repeat;background-size: auto;width: auto;height: auto;"><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">1</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">[HCTF 2018]WarmUp</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">PHP文件包含、白名单校验</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">?截断校验，include使用原始参数</span></section></td></tr><tr style="color: rgb(0, 0, 0);background-attachment: scroll;background-clip: border-box;background-color: rgb(248, 248, 248);background-image: none;background-origin: padding-box;background-position-x: left;background-position-y: top;background-repeat: no-repeat;background-size: auto;width: auto;height: auto;"><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">2</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">[极客大挑战 2019]EasySQL</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">字符型SQL注入、万能密码</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">单引号闭合、注释收尾</span></section></td></tr><tr style="color: rgb(0, 0, 0);background-attachment: scroll;background-clip: border-box;background-color: rgb(255, 255, 255);background-image: none;background-origin: padding-box;background-position-x: left;background-position-y: top;background-repeat: no-repeat;background-size: auto;width: auto;height: auto;"><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">3</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">[ACTF2020 新生赛]Include</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">文件包含、php://filter伪协议</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">读取PHP源码，base64解码</span></section></td></tr><tr style="color: rgb(0, 0, 0);background-attachment: scroll;background-clip: border-box;background-color: rgb(248, 248, 248);background-image: none;background-origin: padding-box;background-position-x: left;background-position-y: top;background-repeat: no-repeat;background-size: auto;width: auto;height: auto;"><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">4</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">[ACTF2020 新生赛]Exec</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">命令注入</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">;分隔命令拼接执行</span></section></td></tr><tr style="color: rgb(0, 0, 0);background-attachment: scroll;background-clip: border-box;background-color: rgb(255, 255, 255);background-image: none;background-origin: padding-box;background-position-x: left;background-position-y: top;background-repeat: no-repeat;background-size: auto;width: auto;height: auto;"><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">5</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">[GXYCTF2019]Ping Ping Ping</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">命令注入</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">空格过滤${IFS}、flag关键字过滤、命令替换</span></section></td></tr><tr style="color: rgb(0, 0, 0);background-attachment: scroll;background-clip: border-box;background-color: rgb(248, 248, 248);background-image: none;background-origin: padding-box;background-position-x: left;background-position-y: top;background-repeat: no-repeat;background-size: auto;width: auto;height: auto;"><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">6</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">[强网杯 2019]随便注入</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">MySQL堆叠注入</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">select被过滤，使用handler读取数据</span></section></td></tr><tr style="color: rgb(0, 0, 0);background-attachment: scroll;background-clip: border-box;background-color: rgb(255, 255, 255);background-image: none;background-origin: padding-box;background-position-x: left;background-position-y: top;background-repeat: no-repeat;background-size: auto;width: auto;height: auto;"><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">7</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">[极客大挑战 2019]Knife</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">一句话木马、蚁剑连接</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">识别题目提示Knife=菜刀</span></section></td></tr><tr style="color: rgb(0, 0, 0);background-attachment: scroll;background-clip: border-box;background-color: rgb(248, 248, 248);background-image: none;background-origin: padding-box;background-position-x: left;background-position-y: top;background-repeat: no-repeat;background-size: auto;width: auto;height: auto;"><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">8</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">[极客大挑战 2019]Secret File</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">文件包含、信息泄露</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">抓包寻找隐藏页面secr3t.php</span></section></td></tr><tr style="color: rgb(0, 0, 0);background-attachment: scroll;background-clip: border-box;background-color: rgb(255, 255, 255);background-image: none;background-origin: padding-box;background-position-x: left;background-position-y: top;background-repeat: no-repeat;background-size: auto;width: auto;height: auto;"><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">9</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">[ACTF2020 新生赛]BackupFile</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">备份泄露、PHP弱类型审计</span></section></td><td style="padding-top: 5px;padding-right: 10px;padding-bottom: 5px;padding-left: 10px;min-width: 85px;border-top-style: solid;border-bottom-style: solid;border-left-style: solid;border-right-style: solid;border-top-width: 1px;border-bottom-width: 1px;border-left-width: 1px;border-right-width: 1px;border-top-color: rgba(204, 204, 204, 0.4);border-bottom-color: rgba(204, 204, 204, 0.4);border-left-color: rgba(204, 204, 204, 0.4);border-right-color: rgba(204, 204, 204, 0.4);border-top-left-radius: 0px;border-top-right-radius: 0px;border-bottom-right-radius: 0px;border-bottom-left-radius: 0px;"><section><span leaf="">intval截断、==弱比较</span></section></td></tr></tbody></table>  
# 1、[HCTF 2018]WarmUp  
  
![image](https://mmbiz.qpic.cn/sz_mmbiz_jpg/IjQoEYG92cwTZqicPuOE5awvAOU9P4vEwx82R1nIEXuYkcxKxLKBhqtylaUjAtw3H30a8q4Yh7SKBsIb9SRg5H2ryUKHY0nbkOHAaiaOYHYGU/640?wx_fmt=webp&from=appmsg "")  
首先我们进行查看原代码   发现  
  
![image](https://mmbiz.qpic.cn/sz_mmbiz_jpg/IjQoEYG92cxcEVaIBHeNseunMth4FwjlziaXtIDrtGS6qpGo0HhCjBHMtUdDyPzJDJLG24uPQh2liaEiaIo30AXbHaITr8KfqtAVl9JnaeSWiao/640?wx_fmt=webp&from=appmsg "")  
然后进行访问   source.php  
  
![image](https://mmbiz.qpic.cn/sz_mmbiz_jpg/IjQoEYG92cz5xKXuuVDz0t2nX7pV80aZFPoeH0n3YxgMx9PiaxsZhZ4kwE1ZibuRbQyT7wugwQFxXPOdCHVc6icLsgKcDGCcfDIoian7enAbltg/640?wx_fmt=webp&from=appmsg "")  
接下来进行访问  hint.php   发现ffffllllaaaagggg  
  
![image](https://mmbiz.qpic.cn/mmbiz_jpg/IjQoEYG92cySKbRGd8zib5PQZcBnFAWXgkOlD7x65LLuT8ngpYiboNWwCj3RMysIpy5H6JWhgtJz389hmj5XXWrtKW6cM3ECNIHIZTOib5ibyhU/640?wx_fmt=webp&from=appmsg "")  
校验里面 $_page  而 include 里面执行的是 file 字符串 所以这个时候我们就进行构造payload  
```
source.php？file=hint.php？/../../../../../fffflllllaaagggg
```  
  
![image](https://mmbiz.qpic.cn/mmbiz_jpg/IjQoEYG92cy2vwibklETQzAADKyzrjLyKl2jJZVK1G8j0rgRoNVicak85d8G7ciakiaicCala8CBM58EjN9KlrLrBicTMYQcSibrBHyRIKHqHBWKtY/640?wx_fmt=webp&from=appmsg "")  
  
image  
  
CTF2{bc759c51-e6c3-4617-b3ee-1266f80c9ded}  
# 2、[极客大挑战 2019]EasySQL  
  
进行测试 我们就尝试进行单引号闭合  
  
![image](https://mmbiz.qpic.cn/sz_mmbiz_jpg/IjQoEYG92czInHe0lROvp0KhA6GI8sTkT8j1mBGCnOic5yzozYn3yJl41ybkVxhoJ8kOpWBv6a2vSDvkdE8t421ibQNLK27RW9dYktMichYwc8/640?wx_fmt=webp&from=appmsg "")  
判断为单引号闭合的字符型 SQL 注入看到登录框，参数为 GET 传参。，结合题目为sql注入，直接尝试万能密码：  
```
 admin' or '1'='1'#
```  
  
密码随便填  
  
![image](https://mmbiz.qpic.cn/mmbiz_jpg/IjQoEYG92cwAJLlfM3juGjwWhRic2Rouic6e0J3scIcM4OaBJibPN2ic3d71qMxicwiapJjGHHv0x24EoetHqYXm6AoxT1qGMYuGwjxLPkic6icYrvM/640?wx_fmt=webp&from=appmsg "")  
  
image  
# 3、[ACTF2020 新生赛]Include  
  
F12可以看到源代码里面有?file=flag.php 进行访问  
  
![image](https://mmbiz.qpic.cn/mmbiz_jpg/IjQoEYG92cyBZiaKEH7S6nSEZZwCpzt4B6DibHH8BFRPpob0optkudtSf8SKtHqq2DMKNc5J0aE5rwh2wiaN5QeS6RYQUTdLW622E2Nk8J0Rlk/640?wx_fmt=webp&from=appmsg "")  
说明了这个文件被执行了，我们怎样才能只读取文件，在里面找到flag 所以说：PHP 提供了一种内置的流协议：  作用：在读取文件内容的时候，先经过一个过滤器处理，再交给 includephp://filter  
  
简单点来说，就是这个内置流协议 会先把里面的内容，进行转化为我们所想要的base64编码，我们是可以进行读取的  ，最后才会执行  
  
onvert.base64-encode   是php://filter里面的过滤器，这个比较常用  
  
![image](https://mmbiz.qpic.cn/sz_mmbiz_jpg/IjQoEYG92cwX4PJwza76Zicu3ib3HiasutASsUtsuCAZggPOQurODN4rpGTEZNiaBYufmiccUvXUniaepzJJQrCbt3qulJ0JGT27gcRwLlFYzya7o/640?wx_fmt=webp&from=appmsg "")  
base64 进行解码  
  
![image](https://mmbiz.qpic.cn/mmbiz_jpg/IjQoEYG92cw5RcvGtvXqZkALBZ5DUtaZDnZO42oEYIuYibUVQYGz4lJ8TGt4xJG3NZj1zdGleyIFX2Jsfsmvak0yab7nxxmFmYQPcFGepFxc/640?wx_fmt=webp&from=appmsg "")  
CTF2{7f3cc302-7c71-47c6-afec-61f6938efd14}  
# 4、[ACTF2020 新生赛]Exec  
  
输入正常 IP，ping 功能正常执行，说明用户输入直接拼接进系统命令，存在**命令注入漏洞**  
  
![image](https://mmbiz.qpic.cn/sz_mmbiz_jpg/IjQoEYG92cwfq6sjna9ZBFDCoEv4h09cc2nRDUoXF5h4nt8Bh5icN44W3h5ibaVGNYzicps7ryHZDB3aB8HO0PjfM8XvFEhLM5rKSSiaD1yrwo0/640?wx_fmt=webp&from=appmsg "")  
  
image  
  
Linux 下支持多种命令分隔符，可以在原有 ping 命令结束后拼接新命令  
```
127.0.0.1;cat /flag
```  
  
![image](https://mmbiz.qpic.cn/mmbiz_jpg/IjQoEYG92cx8N9c8motEXxF2H3bMdQm7njicialGDMjy8k0kZjc9lm6CsbxtqMsKBIoWU7l0XwxZ0jWViblhibcF2tJjibIYy0DicagEAibEqEO860/640?wx_fmt=webp&from=appmsg "")  
CTF2{c2cf3e6e-a177-4009-8956-cbe5f0ba616a}  
# 5、[GXYCTF2019]Ping Ping Ping  
  
首先进行测试看看ping回显是否正常  
  
![image](https://mmbiz.qpic.cn/sz_mmbiz_jpg/IjQoEYG92cyulZVylxlXYhdxI4Ru65xXWWzVulM0I2oeyEfpUuiaZibqhFOMG97hmTStA2uWUycTDzFjpsKGHVPicudpbpRmzsSEqUt844QfQY/640?wx_fmt=webp&from=appmsg "")  
注入分隔符，先 ls 看目录  发现了  
  
![image](https://mmbiz.qpic.cn/mmbiz_jpg/IjQoEYG92cxenP422EgAMWJPlibyvGAZohfXNkI4HgcWLMJUTf0boibccV1C5g3Jib9ucias0hAiaWIRlTr1MurGMyBwR8YADhFCjnC5NwlaFj5A/640?wx_fmt=webp&from=appmsg "")  
尝试进行payload  
  
![image](https://mmbiz.qpic.cn/sz_mmbiz_jpg/IjQoEYG92cxYos6y16C26JzpyVFqOdiboj3XmXnENZib0ibhgOibibdDqK3nE0ZRGOaRbx5BhdpQuibsdOBibTdooBZktdgJDJnvAv0LWfJ7zvWFF4/640?wx_fmt=webp&from=appmsg "")  
这里说明空格被过滤了    空格换成**${IFS}**继续尝试  
  
![image](https://mmbiz.qpic.cn/mmbiz_jpg/IjQoEYG92cxgLXL0EAyu8358wPSy53iaslY2rpa78XTTOK4j2L6qCJCH60bD1G5JBYeSsqpgENqjZp7J6oERlRib6MhcYujU2zokHznMho964/640?wx_fmt=webp&from=appmsg "")  
说明：这条字符串命中了 /.\*f.\*l.\*a.\*g.\*/ 这条正则     说明flag被过滤了 我们想 ls 里面有 flag.php  我们从 ls 下手进行绕过核心就是这句话：ls 能自己把 flag.php 这个名字吐出来，我们输入的时候不写 flag，以此绕过那个 f.*l.*a.*g.* 的正则 反引号  是 shell 的「命令替换」语法。   shell 先处理反引号里面的     index.php flag.phpxxxls  
```
;cat$IFS`ls`
```  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/IjQoEYG92cwlRcBsR2XDkJYdicaiczxU7FGEKCpSIczWO3VTVTJgrWRRXZONN7nHNw3mxSCbKK5YYGO7kKOlTuyfuVKUkSaxAbib53oVAtjkYY/640?wx_fmt=png&from=appmsg "")  
flag = CTF2{d2cfcc1a-0641-4429-8b82-c7ae8b6ae4e3}  
# 6、[强网杯 2019] 随便注入  
  
输入单引号  测试 页面报错，确认是单引号闭合的字符型注入点。'  
  
![image](https://mmbiz.qpic.cn/mmbiz_jpg/IjQoEYG92cw1jnYVWZHQ1mT7llxweicxJO4rxxbWgu5gQw7He8Ogu0WHbVxtx8ibymiaI6fd9aZc1YZBymjanxRlMQCYAKniaicEgRFTOGnljADM/640?wx_fmt=webp&from=appmsg "")  
1' and 1=1# 测试，页面正常；  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/IjQoEYG92cyzR7Tya7uj1JpQHm2CqSlBsSGgojn75ykxCwhVDkmRnpfpN2dibUsLNpYkK4O2ud6f3EtDJ4BoIicOpgDrfnKk8ThpAkNLNmdNc/640?wx_fmt=png&from=appmsg "")  
使用判断列数： 报错，确定查询为 2 列。order by   
```
-1' union select 1,2 --+
```  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/IjQoEYG92cy0Hpw7n4N7OgDeGtq0Fe9eSQLCjFoOkVN2tpAgicXgk54suZPL25OV5t70XctbYEKJVJ88Mdq9HZ5gjfr3BNK4Kv8Q57vibJYmQ/640?wx_fmt=png&from=appmsg "")  
  
image  
  
发现不行，我们就应该考虑，这有没有可能被ban了那，试试分割符  
  
爆所有数据库  
```
1';show databases;--+
```  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/IjQoEYG92cwulrJx36DAoCAUicuxAuZ3mEahoibwricbjiacxX9k1icccx9uibdhjRplgyL5O5JLbn441WYACmv3sPhBOHrt8t28VkJZEtRrwS3d4/640?wx_fmt=png&from=appmsg "")  
  
image  
  
爆出这个库里面所有表  
```
1';use supersqli;show tables --+
```  
  
![image](https://mmbiz.qpic.cn/sz_mmbiz_png/IjQoEYG92czOuicPdibdWIRLUSXSyORcb6VH1k7SrWgLbckSKv1GTh3aDdPKuQQFA3xeBmKqxork3FYgEaX4iaHQl7LK7Fm0JiameRCyKPKJjJY/640?wx_fmt=png&from=appmsg "")  
  
image  
  
返回两张表       表名是数字，必须加 反引号  
  
查看表里面的字段  
```
1';use supersqli;show columns from `1919810931114514`;#
```  
  
![image](https://mmbiz.qpic.cn/mmbiz_png/IjQoEYG92cxiceAcEqksk0n7b2v7FNw2sqc2ayF3QtVibrRC8l40GaFp2guoibFceJGh6Muk4QVTAtmia2rKfCDQGfp5AtTpYIz7c9psONhrNFc/640?wx_fmt=png&from=appmsg "")  
  
image  
  
show columns 确认字段名叫 flag  
```
1';use supersqli;handler `1919810931114514` open;handler `1919810931114514` read first;#
```  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/IjQoEYG92cybxZ78clX0dPu5uQIFapgd5pcTg5o2zG5n5qPg8e4rQnL9Lk5ltWicSlX0VHLibv0J5JhCia5CqoNKljNkzMP8HXAnllcJibVPDrU/640?wx_fmt=png&from=appmsg "")  
CTF2{7dabf855-1511-43f3-ad6b-c09e468b2e4f}  
# 7[极客大挑战 2019]刀  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/IjQoEYG92cxA1qicqzd2rEJt0rH7zRIHJEiarN9JDqwq6cdUlqJB0HQLFyeSnOZewv9tbTHejAoQEVrKMtucFed1AfPWhqiclalcl3lXgKpRic0/640?wx_fmt=png&from=appmsg "")  
菜刀、knife（英文刀） 根据一句话木马传参Syc 可以让我们想到用蚁剑进行看看里面的内容  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/IjQoEYG92cwRfwkWPfK3WRFkTf5icLljxpdXNt8Gyic6vH63RHgPvCeKZJYesq7Hy9Uvl1EAHr5v90RGgpxoAUAGN2xRAYBy2CkiaQDlXwdpvM/640?wx_fmt=png&from=appmsg "")  
  
图片  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/IjQoEYG92cyjNFEYOvaOL5GibsLqR4P57HMHQ8mDognuiakWCtKiaGRztiaTEh6LN8IxxrVjPgwm2SaCEmznGX8yOPMfdsXZ3uolRd6KloHfCOA/640?wx_fmt=png&from=appmsg "")  
在根目录下就可以找到flag  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/IjQoEYG92cxpJkH0ibwdf5uuhnnK2F2txianibzQ6wCmvM4oygssCIdLFtRH82rMibJf5Jz2Qz7p0GaRYYhKxXPI08TjibhINBQpibCjibgulPKylA/640?wx_fmt=png&from=appmsg "")  
  
图片  
# 8、极客大挑战 2019]秘密档案  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/IjQoEYG92cxAL4RFV5LicRvMlt79vJiaDqRay6ft2zQY3wicml17bzdvGFiaxlwUAYGJND9icfOGbXpibfU7r2YO6IY4kiaCUW3BP5nzGz71nIjx7c/640?wx_fmt=png&from=appmsg "")  
根据图片里面的内容 我们可以知道 页面藏了东西，不在当前可视区域 所以我们就进行看看原代码  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/IjQoEYG92cz46eFTGM06icjfLIPymFJrFWQYmSW46FvyYt9E6PKiaO2oru7hbdezr9bnoFemV6mj27sU850V8uQ9xJbdf84B245l8d90jxWFo/640?wx_fmt=png&from=appmsg "")  
现这个东西 是页面里面没有的东西 我们试着进行点进行去 发现  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/IjQoEYG92cw6AIx6EVJiawgr7KQo7eeu5mrcxIn5bAEiaaibIF061G2u7bURXwrribRXsAjDJNDW19d2PhickFeIJb5CzOQ7xRko6Q4bZknWJLcE/640?wx_fmt=png&from=appmsg "")  
在进行点进去 发现  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/IjQoEYG92cxv8BficqkH4krQiaEVYiaRic2TIu1S0mHJhnvpoSiaO9JGdWjVt4QpwWiblKFc5NKBNcO5u385ibckx2Kiaqic1qiakog4G7LlOlicY1N3Vs/640?wx_fmt=png&from=appmsg "")  
  
图片  
  
据它的提示  我们不难发现中间肯定有东西是我们在action.php 到end.php里面的东西   所以说我们在actio.phpj进行抓包 看看里面有什么隐藏的东西  
  
找到action.php之后进行放行  到http历史记录里面查看action.php进行查看  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/IjQoEYG92cyPxSJbeQ4dQibTDc8ZnUyUepOhxosxAHibl8oskmiaDrKgcJo6vDftOoXKDQQeXXcbq25wfnRZBHRKia1qomhxhJIfUUSZy2B0p3U/640?wx_fmt=webp&from=appmsg "")  
结果真有所发现 进行继续访问  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/IjQoEYG92cx3M4Hj4W147SFYficuQaKOFJfATU2jySk9E6HZjdoK3ib1FELYn59pqBLhMsWeVK9zwwN7vqxKBBWe7cS8R5pQyUlLDxvL6NhpA/640?wx_fmt=webp&from=appmsg "")  
过滤黑名单：  
  
../  —— 禁止目录穿越  
  
TP —— 过滤 php://（因为里面包含 TP）  
  
input —— 过滤 php://input  
  
data —— 过滤 data:// 伪协议  
```
secr3t.php?file=php://filter/read=convert.base64-encode/resource=flag.php
```  
  
![图片](https://mmbiz.qpic.cn/mmbiz_jpg/IjQoEYG92czrdQicdQjNBE62zx4YjrWSAuJe8c0AicFZ74UcNiaHNneFfkAo62NUnbtchMLYMNewmfTTN4sHGvqjylrtjtJAGPJLszeEIBgcAg/640?wx_fmt=webp&from=appmsg "")  
进行解码  
  
![图片](https://mmbiz.qpic.cn/mmbiz_jpg/IjQoEYG92cwlghsogkMRyoTibP4bREKQVxCTvdxJavA0mKdNn3UVqCbRnIYQRpfynYCC7zLibTP0O03jicjE5G3a787szvgVsVhgyuaVdkHTt8/640?wx_fmt=webp&from=appmsg "")  
CTF2{b92e831e-8f9c-4dfc-adc6-2be48318bee7}  
# 9、[ACTF2020 新生赛]备份文件  
  
备份文件：优先试 .bak、.swp、.zip、.tar  
```
index.php.bak
```  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/IjQoEYG92cxDS8OEMIPFkcmF6tEU7BO7fc6IBIEIWtYkUYxqyOtN9ibjDgY2iblRHL7Pf8oAHFw4a9Nxa2aduWibTzzJeNEQ3OoTGibUbZ0bNy4/640?wx_fmt=png&from=appmsg "")  
  
图片  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/IjQoEYG92czmPApagNvDTctf59yHrheQ2oKhJCqq6eqX5d8E1g8GrDra7SjX9s5jKQvuBIv64uRrVEAcyrb426cplG5JZEfaP7pju62HBick/640?wx_fmt=png&from=appmsg "")  
看源码 ： PHP 弱类型比较 + intval 特性    分析之后我们进行传payload  
```
?key=123
```  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/IjQoEYG92cxqVrMDvFSVdsNR4Tnl7fADcQiaXSHjg6bYVN1PZrPSZ25YNpFftPh7r0mVBvWnRBzBx3LjpfcMgIke3ATNyR8LW9MVXbmoCvT8/640?wx_fmt=png&from=appmsg "")  
CTF2{48191ea1-149f-40e1-a7fe-354cb9476efe}  
  
  
