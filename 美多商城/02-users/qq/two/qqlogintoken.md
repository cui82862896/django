#### 获取OpenID之后需要判断用户之前否绑定过 {#后端接口设计}

如果用户之前已经登录过,则存在openid,返回登录的信息就可以

如果不存在需要生成一个token,同时跳转到注册页面,注册时将此token传递给后台进行校验

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

        #判断用户是否存在
        try:
            qq_user = OAuthQQUser.objects.get(openid=openid)
        except OAuthQQUser.DoesNotExist:
            #不存在就是第一次授权登录
            token = OAuthQQUser.generate_save_user_token(openid)
            return Response({'access_token':token})
        else:
            #查询用户,生成登录token
            user = qq_user.user
            jwt_payload_handler = api_settings.JWT_PAYLOAD_HANDLER
            jwt_encode_handler = api_settings.JWT_ENCODE_HANDLER

            payload = jwt_payload_handler(user)
            token = jwt_encode_handler(payload)

            response = Response({
                'token': token,
                'user_id': user.id,
                'username': user.username
            })
            return response
```



#### 前端 {#前端}

在oauth\_callback.js 中修改

```
mounted: function(){
        // 从路径中获取qq重定向返回的code
        var code = this.get_query_string('code');
        axios.get(this.host + '/oauth/qq/users/?code=' + code, {
                responseType: 'json',
            })
            .then(response => {
                if (response.data.user_id){
                    // 用户已绑定
                    sessionStorage.clear();
                    localStorage.clear();
                    localStorage.user_id = response.data.user_id;
                    localStorage.username = response.data.username;
                    localStorage.token = response.data.token;

                    // 从路径中取出state,引导用户进入登录成功之后的页面
                    var state = this.get_query_string('state');
                    location.href = state;
                } else {
                    // 用户未绑定
                    this.access_token = response.data.access_token;
                    this.generate_image_code();
                    this.is_show_waiting = false;
                }
            })
            .catch(error => {
                console.log(error.response.data);
                alert('服务器异常');
            })
    },
```



