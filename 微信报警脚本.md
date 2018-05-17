##### Shell版本：
```
#!/bin/bash
#

declare -i ToParty=1
declare -i agentid=1000002
declare -i safe=0
content=$@
ToUser=User       # 企业微信通讯录中的有效用户
msgtype=text
CropID=xxxxxxxxxxxxxxxxxx  # ID
Secret=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx # 相应的密钥
ToKenURL=https://qyapi.weixin.qq.com/cgi-bin/gettoken?corpid=${CropID}\&corpsecret=${Secret}
ToKen=`curl -s -G ${ToKenURL} | awk -F"," '{print $3}' | awk -F":" 'gsub(/"/,"",$2){print $2}'`
SendURL=https://qyapi.weixin.qq.com/cgi-bin/message/send?access_token=${ToKen}

output() { 
cat <<EOF
{
   "touser": "$ToUser",
   "toparty": "$ToParty",
   "msgtype": "text",
   "agentid": $agentid,
   "text": {
       "content": "$content"
   },
   "safe":0
}
EOF
}

/usr/bin/curl --data-ascii "$(output $content)" $SendURL
```

##### Python版本：
