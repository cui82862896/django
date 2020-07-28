# QQ登录回调处理 {#2--qq登录回调处理}

用户在QQ登录成功后，QQ会将用户重定向回我们配置的回调callback网址，在本项目中，我们申请QQ登录开发资质时配置的回调地址为：

```
http://www.meiduo.site:8080/oauth_callback.html
```

我们在front目录中新建oauth\_callback.html文件，用于接收QQ登录成功的用户回调请求。在该页面中，提供了用于用户首次使用QQ登录时需要绑定用户身份的表单信息。

```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en">
<head>
    <meta http-equiv="Content-Type" content="text/html;charset=UTF-8">
    <title>美多商城-绑定用户</title>
    <link rel="stylesheet" type="text/css" href="css/reset.css">
    <link rel="stylesheet" type="text/css" href="css/main.css">
    <script type="text/javascript" src="js/host.js"></script>
    <script type="text/javascript" src="js/vue-2.5.16.js"></script>
    <script type="text/javascript" src="js/axios-0.18.0.min.js"></script>
</head>
<body>
    <div id="app">
        <div v-if="is_show_waiting" class="pass_change_finish">请稍后...</div>
        <div v-else>
            <div class="register_con">
                <div class="l_con fl">
                    <a class="reg_logo"><img src="images/logo.png"></a>
                    <div class="reg_slogan">商品美 · 种类多 · 欢迎光临</div>
                    <div class="reg_banner"></div>
                </div>

                <div class="r_con fr">
                    <div class="reg_title clearfix">
                        <h1>绑定用户</h1>
                    </div>
                    <div class="reg_form clearfix" id="app" v-cloak>
                        <form id="reg_form" v-on:submit.prevent="on_submit">
                        <ul>
                            <li>
                                <label>手机号:</label>
                                <input type="text" v-model="mobile" v-on:blur="check_phone" name="phone" id="phone">
                                <span v-show="error_phone" class="error_tip">{{ error_phone_message }}</span>
                            </li>
                            <li>
                                <label>密码:</label>
                                <input type="password" v-model="password" v-on:blur="check_pwd" name="pwd" id="pwd">
                                <span v-show="error_password" class="error_tip">密码最少8位，最长20位</span>
                            </li>
                            <li>
                                <label>图形验证码:</label>
                                <input type="text" v-model="image_code" v-on:blur="check_image_code" name="pic_code" id="pic_code" class="msg_input">
                                <img v-bind:src="image_code_url" v-on:click="generate_image_code" alt="图形验证码" class="pic_code">
                                <span v-show="error_image_code" class="error_tip">{{ error_image_code_message }}</span>
                            </li>
                            <li>
                                <label>短信验证码:</label>
                                <input type="text" v-model="sms_code" v-on:blur="check_sms_code" name="msg_code" id="msg_code" class="msg_input">
                                <a v-on:click="send_sms_code" class="get_msg_code">{{ sms_code_tip }}</a>
                                <span v-show="error_sms_code" class="error_tip">{{ error_sms_code_message }}</span>
                            </li>
                            <li class="reg_sub">
                                <input type="submit" value="保 存" name="">
                            </li>
                        </ul>
                        </form>
                    </div>
                </div>
            </div>

            <div class="footer no-mp">
                <div class="foot_link">
                    <a href="#">关于我们</a>
                    <span>|</span>
                    <a href="#">联系我们</a>
                    <span>|</span>
                    <a href="#">招聘人才</a>
                    <span>|</span>
                    <a href="#">友情链接</a>
                </div>
                <p>CopyRight © 2016 北京美多商业股份有限公司 All Rights Reserved</p>
                <p>电话：010-****888    京ICP备*******8号</p>
            </div>
        </div>
    </div>
    <script type="text/javascript" src="js/oauth_callback.js"></script>
</body>
</html>
```

在js目录中新建oauth\_callback.js文件

```
var vm = new Vue({
    el: '#app',
    data: {
        host: host,
        is_show_waiting: true,

        error_password: false,
        error_phone: false,
        error_image_code: false,
        error_sms_code: false,
        error_image_code_message: '',
        error_phone_message: '',
        error_sms_code_message: '',

        image_code_id: '', // 图片验证码id
        image_code_url: '',

        sms_code_tip: '获取短信验证码',
        sending_flag: false, // 正在发送短信标志

        password: '',
        mobile: '',
        image_code: '',
        sms_code: '',
        access_token: ''
    },
    mounted: function(){
         // 从路径中获取qq重定向返回的code
        var code = this.get_query_string('code');
        axios.get(this.host + '/oauth/qq/users/?code=' + code, {
                responseType: 'json',
            })
            .then(response => {

            })
            .catch(error => {
                console.log(error.response.data);
                alert('服务器异常');
            })
    },
    methods: {
        // 获取url路径参数
        get_query_string: function(name){
            var reg = new RegExp('(^|&)' + name + '=([^&]*)(&|$)', 'i');
            var r = window.location.search.substr(1).match(reg);
            if (r != null) {
                return decodeURI(r[2]);
            }
            return null;
        },
        // 生成uuid
        generate_uuid: function(){
            var d = new Date().getTime();
            if(window.performance && typeof window.performance.now === "function"){
                d += performance.now(); //use high-precision timer if available
            }
            var uuid = 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, function(c) {
                var r = (d + Math.random()*16)%16 | 0;
                d = Math.floor(d/16);
                return (c =='x' ? r : (r&0x3|0x8)).toString(16);
            });
            return uuid;
        },
        // 生成一个图片验证码的编号，并设置页面中图片验证码img标签的src属性
        generate_image_code: function(){
            // 生成一个编号
            // 严格一点的使用uuid保证编号唯一， 不是很严谨的情况下，也可以使用时间戳
            this.image_code_id = this.generate_uuid();

            // 设置页面中图片验证码img标签的src属性
            this.image_code_url = this.host + "/verifications/imagecodes/" + this.image_code_id + "/";
        },
        check_pwd: function (){
            var len = this.password.length;
            if(len<8||len>20){
                this.error_password = true;
            } else {
                this.error_password = false;
            }
        },
        check_phone: function (){
            var re = /^1[345789]\d{9}$/;
            if(re.test(this.mobile)) {
                this.error_phone = false;
            } else {
                this.error_phone_message = '您输入的手机号格式不正确';
                this.error_phone = true;
            }
        },
        check_image_code: function (){
            if(!this.image_code) {
                this.error_image_code_message = '请填写图片验证码';
                this.error_image_code = true;
            } else {
                this.error_image_code = false;
            }
        },
        check_sms_code: function(){
            if(!this.sms_code){
                this.error_sms_code_message = '请填写短信验证码';
                this.error_sms_code = true;
            } else {
                this.error_sms_code = false;
            }
        },
        // 发送手机短信验证码
        send_sms_code: function(){
            if (this.sending_flag == true) {
                return;
            }
            this.sending_flag = true;

            // 校验参数，保证输入框有数据填写
            this.check_phone();
            this.check_image_code();

            if (this.error_phone == true || this.error_image_code == true) {
                this.sending_flag = false;
                return;
            }

            // 向后端接口发送请求，让后端发送短信验证码
            axios.get(this.host + '/verifications/smscodes/' + this.mobile + '/?text=' + this.image_code+'&image_code_id='+ this.image_code_id, {
                    responseType: 'json'
                })
                .then(response => {
                    // 表示后端发送短信成功
                    // 倒计时60秒，60秒后允许用户再次点击发送短信验证码的按钮
                    var num = 60;
                    // 设置一个计时器
                    var t = setInterval(() => {
                        if (num == 1) {
                            // 如果计时器到最后, 清除计时器对象
                            clearInterval(t);
                            // 将点击获取验证码的按钮展示的文本回复成原始文本
                            this.sms_code_tip = '获取短信验证码';
                            // 将点击按钮的onclick事件函数恢复回去
                            this.sending_flag = false;
                        } else {
                            num -= 1;
                            // 展示倒计时信息
                            this.sms_code_tip = num + '秒';
                        }
                    }, 1000, 60)
                })
                .catch(error => {
                    if (error.response.status == 400) {
                        this.error_image_code_message = '图片验证码有误';
                        this.error_image_code = true;
                    } else {
                        console.log(error.response.data);
                    }
                    this.sending_flag = false;
                })
        },
        // 保存
        on_submit: function(){
            this.check_pwd();
            this.check_phone();
            this.check_sms_code();

        }
    }
});
```

使用QQ扫描登录

![](/assets/qq_login_code.png)在QQ将用户重定向到此网页的时候，重定向的网址会携带QQ提供的code参数，用于获取用户信息使用，我们需要将这个code参数发送给后端，在后端中使用code参数向QQ请求用户的身份信息，并查询与该QQ用户绑定的用户。

####  {#后端接口设计}



