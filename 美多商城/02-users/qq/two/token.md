# 获取Access\_Token {#2--qq登录回调处理}

#### 后端接口设计 {#后端接口设计}

请求方式 ： GET /oauth/qq/users/?code=xxx

请求参数：

| 参数 | 类型 | 是否必传 | 说明 |
| :--- | :--- | :--- | :--- |
| code | str | 是 | qq返回的授权凭证code |

返回数据：

| 返回值 | 类型 | 是否必须 | 说明 |
| :--- | :--- | :--- | :--- |
| access\_token | str | 否 | 用户是第一次使用QQ登录时返回，其中包含openid，用于绑定身份使用 |
| token | str | 否 | 用户不是第一次使用QQ登录时返回，登录成功的JWT token |
| username | str | 否 | 用户不是第一次使用QQ登录时返回，用户名 |
| user\_id | int | 否 | 用户不是第一次使用QQ登录时返回，用户id |

![](/assets/qq_login_two_1.png)

获取access\_token的接口文档

![](/assets/qq_login_two_api.png)

定义视图

```
from rest_framework.views import APIView
from rest_framework.response import Response
from .utils import OauthQQ
from rest_framework import status
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
        except Exception as e:
            logger.error(e)

        return Response()
```

在OAuthQQ辅助类中添加方法并修改init方法：

```
from mall import settings
from urllib.parse import urlencode,parse_qs
from urllib.request import urlopen
class OauthQQ(object):
    """
    QQ授权工具类
    """
    def __init__(self,client_id=None,redirect_uri=None,client_secret=None):

        self.client_id = client_id or settings.QQ_APP_ID
        self.redirect_uri = redirect_uri or settings.QQ_REDIRECT_URL
        self.client_secret = client_secret or settings.QQ_APP_KEY

    def get_access_token(self,code):
        # PC网站：https://graph.qq.com/oauth2.0/token
        # GET
        # grant_type      必须      授权类型，在本步骤中，此值为“authorization_code”。
        # client_id       必须      申请QQ登录成功后，分配给网站的appid。
        # client_secret   必须      申请QQ登录成功后，分配给网站的appkey。
        # code            必须      上一步返回的authorization
        # redirect_uri    必须      与上面一步中传入的redirect_uri保持一致。

        # 准备url,注意添加?
        base_url = 'https://graph.qq.com/oauth2.0/token?'
        params = {
            'grant_type': 'authorization_code',
            'client_id': self.client_id,
            'client_secret': self.client_secret,
            'code': code,
            'redirect_uri': self.redirect_uri
        }

        url = base_url + urlencode(params)

        #发送请求,获取响应
        response = urlopen(url)

        #读取数据
        data = response.read().decode()

        query_params = parse_qs(data)

        #获取token,并进行判断
        access_token = query_params.get('access_token')
        # print(access_token)
        # ['5F5893DBC5339A54B26AFD0A1312276F']
        if access_token is None:
            raise Exception('获取token失败')

        return access_token[0]
```

> 注意: 通过query\_params.get\('access\_token'\)获取的token是一个列表
>
> \['5F5893DBC5339A54B26AFD0A1312276F'\]

设置url

```
from django.conf.urls import url
from . import views

urlpatterns = [
    #   /oauth/qq/statues/
    url(r'^qq/statues/$',views.QQAuthURLView.as_view(),name='statues'),
    # /oauth/qq/users/
    url(r'^qq/users/$',views.QQTokenView.as_view(),name='qqtoken'),

]
```

####  {#后端接口设计}



