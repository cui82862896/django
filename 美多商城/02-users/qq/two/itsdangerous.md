# itsdangerous生成激活token {#itsdangerous生成激活token}

[itsdangerous中文文档](http://itsdangerous.readthedocs.io/en/latest/)

* 1.安装 itsdangerous 模块

  ```
  pip install itsdangerous
  ```

  ![](/assets/itsdangerous_1.png)

* 2.生成用户激活token的方法封装在OAuthQQUser模型类中

  * Serializer\(\)生成序列化器，传入混淆字符串和过期时间
  * dumps\(\)生成openid加密后的token，传入封装openid的字典
  * 返回token字符串
  * loads\(\)解出token字符串，得到用户id明文

    ```
    from itsdangerous import TimedJSONWebSignatureSerializer as Serializer
    from mall import settings

    class OAuthQQUser(BaseModel):
        """
        QQ登录用户数据
        """
       ...

        @staticmethod
        def generate_save_user_token(openid):

            serializer = Serializer(settings.SECRET_KEY, expires_in=3600)

            token = serializer.dumps({'openid': openid})

            return token.decode()
    ```

* 3.生成激活token方法的调用

  ```
  token = OAuthQQUser.generate_save_user_token(openid)
  ```

## 提示: SignatureExpired 异常 {#提示-signatureexpired-异常}

* 签名过期的异常

  ![](/assets/itsdangerous_2.png)



