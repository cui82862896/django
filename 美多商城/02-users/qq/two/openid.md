#### 获取token之后需要获取OpenID {#后端接口设计}

OpenID是此网站上或应用中唯一对应用户身份的标识，网站或应用可将此ID进行存储，便于用户下次登录时辨识其身份，或将其与用户在网站上或应用中的原有账号进行绑定。

![](blob:file:///177e912a-e535-44b7-9f2e-bfdde249805e)

获取OpenID的API接口文档

![](/assets/qq_login_two_openid_api.png)

在OAuthQQ辅助类中添加方法

```
def get_openid(self,access_token):
        # https://graph.qq.com/oauth2.0/me
        # GET
        # access_token        必须      在Step1中获取到的accesstoken。

        # 返回数据PC网站接入时，获取到用户OpenID，返回包如下：
        # callback( {"client_id":"YOUR_APPID","openid":"YOUR_OPENID"} );
        # openid是此网站上唯一对应用户身份的标识，网站可将此ID进行存储便于用户下次登录时辨识其身份，
        # 或将其与用户在网站上的原有账号进行绑定

        base_url = 'https://graph.qq.com/oauth2.0/me?'

        params = {
            'access_token': access_token
        }

        url = base_url + urlencode(params)

        # 请求来获取响应
        response = urlopen(url)

        response_data = response.read().decode()

        try:
            # 返回的数据  callback({"client_id": "YOUR_APPID", "openid": "YOUR_OPENID"})\n;
            data = json.loads(response_data[10:-4])
        except Exception as e:
            raise Exception('获取用户错误')

        openid = data.get('openid',None)

        return openid
```

完善视图方法

```
from rest_framework.views import APIView
from rest_framework.response import Response
from .utils import OauthQQ
from rest_framework import status
from .models import OAuthQQUser
from rest_framework_jwt.settings import api_settings

import logging

logger = logging.getLogger('meiduo')
# Create your views here.

class QQTokenView(APIView):
    """
    获取access_token
    GET /oauth/qq/users/?code=xxx
    """

    def get(self,request):

        #获取code,并进行判断
        code = request.query_params.get('code')
        if code is None:
            return Response({'message':'缺少参数'},status=status.HTTP_400_BAD_REQUEST)

        qq = OauthQQ()
        try:
            access_token = qq.get_access_token(code)
            openid = qq.get_openid(access_token)
        except Exception :
            return Response({'message':'服务异常'},status=status.HTTP_503_SERVICE_UNAVAILABLE)

       
            return response
```



