# 首页数据 {#页面静态化}

根据数据表的设计,首页数据分为 分类数据 和 广告,楼层数据两部分

分类数据表

![](/assets/home_category.png)广告楼层数据表

![](/assets/home_ad_content.png)

在goods的应用views中,创建视图,并设置应用和功能的urls

```
from django.views.generic import View
from goods.models import GoodsCategory,GoodsChannel
from contents.models import ContentCategory
from collections import OrderedDict

# Create your views here.
class CategoryView(View):
    """
    获取首页分类数据

    GET /goods/categories/
    """
    def get(self,request):
        pass
```

goods应用的urls为

```
from django.conf.urls import url
from . import views

urlpatterns = [
    #/goods/categories/
    url(r'^categories/$',views.CategoryView.as_view(),name='cagegories'),
]
```

mall工程的urls为

```
urlpatterns = [

    url(r'^goods/',include('goods.urls',namespace='goods')),
]
```

### 首页分类数据 {#在admin中注册模型类}

```
# 使用有序字典保存类别的顺序
        # categories = {
        #     1: { # 组1
        #         'channels': [{'id':, 'name':, 'url':},{}, {}...],
        #         'sub_cats': [{'id':, 'name':, 'sub_cats':[{},{}]}, {}, {}, ..]
        #     },
        #     2: { # 组2
        #
        #     }
        # }
        #初始化存储容器
        categories = OrderedDict()
        #获取一级分类
        channels = GoodsChannel.objects.order_by('group_id','sequence')

        #对一级分类进行遍历
        for channel in channels:
            #获取group_id
            group_id = channel.group_id
            #判断group_id 是否在存储容器,如果不在就初始化
            if group_id not in categories:
                categories[group_id] = {
                    'channels':[],
                    'sub_cats':[]
                }

            one = channel.category
            #为channels填充数据
            categories[group_id]['channels'].append({
                'id':one.id,
                'name':one.name,
                'url':channel.url
            })
            #为sub_cats填充数据
            for two in one.goodscategory_set.all():
                #初始化 容器
                two.sub_cats = []
                #遍历获取
                for three in two.goodscategory_set.all():
                    two.sub_cats.append(three)

                #组织数据
                categories[group_id]['sub_cats'].append(two)
```

### 首页广告内容数据 {#在admin中注册模型类}

```
#广告和首页数据
        contents = {}
        content_categories = ContentCategory.objects.all()
        # content_categories = [{'name':xx , 'key': 'index_new'}, {}, {}]

        # {
        #    'index_new': [] ,
        #    'index_lbt': []
        # }
        for cat in content_categories:
            contents[cat.key] = cat.content_set.filter(status=True).order_by('sequence')
```

### 通过模板进行测试 {#在admin中注册模型类}

在mall工程中设置模板文件夹,并配置模板路径

![](/assets/goods_templates.png)

```
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR,'templates')],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```

在模板文件夹中创建home.html

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en">
<head>
    <meta http-equiv="Content-Type" content="text/html;charset=UTF-8">
    <title>美多商城-首页</title>
    <link rel="stylesheet" type="text/css" href="css/reset.css">
    <link rel="stylesheet" type="text/css" href="css/main.css">
    <script type="text/javascript" src="js/host.js"></script>
    <script type="text/javascript" src="js/vue-2.5.16.js"></script>
    <script type="text/javascript" src="js/axios-0.18.0.min.js"></script>
    <script type="text/javascript" src="js/jquery-1.12.4.min.js"></script>
    <script type="text/javascript" src="js/slide.js"></script>
</head>
<body>
    <div id="app" v-cloak>
    <div class="header_con">
        <div class="header">
            <div class="welcome fl">欢迎来到美多商城!</div>
            <div class="fr">
                <div v-if="username" class="login_btn fl">
                    欢迎您：<em>[[ username ]]</em>
                    <span>|</span>
                    <a @click="logout">退出</a>
                </div>
                <div v-else class="login_btn fl">
                    <a href="login.html">登录</a>
                    <span>|</span>
                    <a href="register.html">注册</a>
                </div>
                <div class="user_link fl">
                    <span>|</span>
                    <a href="user_center_info.html">用户中心</a>
                    <span>|</span>
                    <a href="cart.html">我的购物车</a>
                    <span>|</span>
                    <a href="user_center_order.html">我的订单</a>
                </div>
            </div>
        </div>
    </div>

    <div class="search_bar clearfix">
        <a href="index.html" class="logo fl"><img src="images/logo.png"></a>
        <div class="search_wrap fl">
            <form method="get" action="/search.html" class="search_con">
                <input type="text" class="input_text fl" name="q" placeholder="搜索商品">
                <input type="submit" class="input_btn fr" name="" value="搜索">
            </form>
            <ul class="search_suggest fl">
                <li><a href="#">索尼微单</a></li>
                <li><a href="#">优惠15元</a></li>
                <li><a href="#">美妆个护</a></li>
                <li><a href="#">买2免1</a></li>
            </ul>
        </div>

        <div class="guest_cart fr">
            <a href="#" class="cart_name fl">我的购物车</a>
            <div class="goods_count fl" id="show_count">15</div>

            <ul class="cart_goods_show">
                <li>
                    <img src="images/goods/goods001.jpg" alt="商品图片">
                    <h4>商品名称手机</h4>
                    <div>4</div>
                </li>
                <li>
                    <img src="images/goods/goods002.jpg" alt="商品图片">
                    <h4>商品名称手机</h4>
                    <div>5</div>
                </li>
                <li>
                    <img src="images/goods/goods003.jpg" alt="商品图片">
                    <h4>商品名称手机</h4>
                    <div>6</div>
                </li>
                <li>
                    <img src="images/goods/goods003.jpg" alt="商品图片">
                    <h4>商品名称手机</h4>
                    <div>6</div>
                </li>
            </ul>
        </div>
    </div>

    <div class="navbar_con">
        <div class="navbar">
            <h1 class="fl">商品分类</h1>
            <ul class="navlist fl">
                <li><a href="">首页</a></li>
                <li class="interval">|</li>
                <li><a href="">真划算</a></li>
                <li class="interval">|</li>
                <li><a href="">抽奖</a></li>
            </ul>
        </div>
    </div>

    <div class="pos_center_con clearfix">
        <ul class="slide">
            {% for content in contents.index_lbt %}
            <li><a href="{{ content.url }}"><img src="{{ content.image.url }}" alt="{{ content.title }}"></a></li>
            {% endfor %}
        </ul>
        <div class="prev"></div>
        <div class="next"></div>
        <ul class="points">
            <!-- <li class="active"></li>
            <li></li>
            <li></li>
            <li></li> -->
        </ul>
        <ul class="sub_menu">
            {% for group in categories.values %}
            <li>
                <div class="level1">
                    {% for channel in group.channels %}
                    <a href="{{ channel.url }}">{{ channel.name }}</a>
                    {% endfor %}
                </div>
                <div class="level2">
                    {% for cat2 in group.sub_cats %}
                    <div class="list_group">
                        <div class="group_name fl">{{cat2.name}} &gt;</div>
                        <div class="group_detail fl">
                            {% for cat3 in cat2.sub_cats %}
                            <a href="/list.html?cat={{cat3.id}}">{{cat3.name}}</a>
                            {% endfor %}
                        </div>
                    </div>
                    {% endfor %}
                </div>
            </li>
            {% endfor %}
        </ul>

        <div class="news">
            <div class="news_title">
                <h3>快讯</h3>
                <a href="#">更多 &gt;</a>
            </div>
            <ul class="news_list">
                {% for content in contents.index_kx %}
                <li><a href="{{ content.url }}">{{ content.title }}</a></li>
                {% endfor %}
            </ul>
            {% for content in contents.index_ytgg %}
            <a href="{{ content.url }}" class="advs"><img src="{{ content.image.url }}"></a>
            {% endfor %}
        </div>
    </div>

    <div class="list_model">
        <div class="list_title clearfix">
            <h3 class="fl" id="model01">1F 手机通讯</h3>
            <div class="subtitle fr">
                <a @mouseenter="f1_tab=1" :class="f1_tab===1?'active':''">时尚新品</a>
                <a @mouseenter="f1_tab=2" :class="f1_tab===2?'active':''">畅想低价</a>
                <a @mouseenter="f1_tab=3" :class="f1_tab===3?'active':''">手机配件</a>
            </div>
        </div>
        <div class="goods_con clearfix">
            <div class="goods_banner fl">
                <img src="{{ contents.index_1f_logo.0.image.url }}">
                <div class="channel">
                    {% for content in contents.index_1f_pd %}
                    <a href="{{ content.url }}">{{ content.title }}</a>
                    {% endfor %}
                </div>
                <div class="key_words">
                    {% for content in contents.index_1f_bq %}
                    <a href="{{ content.url }}">{{ content.title }}</a>
                    {% endfor %}
                </div>

            </div>
            <ul v-show="f1_tab===1" class="goods_list fl">
                {% for content in contents.index_1f_ssxp %}
                <li>
                    <a href="{{ content.url }}" class="goods_pic"><img src="{{ content.image.url }}"></a>
                    <h4><a href="{{ content.url }}" title="{{ content.title }}">{{ content.title }}</a></h4>
                    <div class="prize">{{ content.text }}</div>
                </li>
                {% endfor %}
            </ul>
            <ul v-show="f1_tab===2" class="goods_list fl">
                {% for content in contents.index_1f_cxdj %}
                <li>
                    <a href="{{ content.url }}" class="goods_pic"><img src="{{ content.image.url }}"></a>
                    <h4><a href="{{ content.url }}" title="{{ content.title }}">{{ content.title }}</a></h4>
                    <div class="prize">{{ content.text }}</div>
                </li>
                {% endfor %}
            </ul>
            <ul v-show="f1_tab===3" class="goods_list fl">
                {% for content in contents.index_1f_sjpj %}
                <li>
                    <a href="{{ content.url }}" class="goods_pic"><img src="{{ content.image.url }}"></a>
                    <h4><a href="{{ content.url }}" title="{{ content.title }}">{{ content.title }}</a></h4>
                    <div class="prize">{{ content.text }}</div>
                </li>
                {% endfor %}
            </ul>
        </div>
    </div>

    <div class="list_model model02">
            <div class="list_title clearfix">
                <h3 class="fl" id="model01">2F 电脑数码</h3>
                <div class="subtitle fr">
                    <a @mouseenter="f2_tab=1" :class="f2_tab===1?'active':''">加价换购</a>
                    <a @mouseenter="f2_tab=2" :class="f2_tab===2?'active':''">畅享低价</a>
                </div>
            </div>
            <div class="goods_con clearfix">
                <div class="goods_banner fl">
                    <img src="{{ contents.index_2f_logo.0.image.url}}">
                    <div class="channel">
                        {% for content in contents.index_2f_pd %}
                        <a href="{{ content.url }}">{{ content.title }}</a>
                        {% endfor %}
                    </div>
                    <div class="key_words">
                        {% for content in contents.index_2f_bq %}
                        <a href="{{ content.url }}">{{ content.title }}</a>
                        {% endfor %}
                    </div>
                </div>
                <ul v-show="f2_tab===1" class="goods_list fl">
                    {% for content in contents.index_2f_jjhg %}
                    <li>
                        <a href="{{ content.url }}" class="goods_pic"><img src="{{ content.image.url }}"></a>
                        <h4><a href="{{ content.url }}" title="{{ content.title }}">{{ content.title }}</a></h4>
                        <div class="prize">{{ content.text }}</div>
                    </li>
                    {% endfor %}
                </ul>
                <ul v-show="f2_tab===2" class="goods_list fl">
                    {% for content in contents.index_2f_cxdj %}
                    <li>
                        <a href="{{ content.url }}" class="goods_pic"><img src="{{ content.image.url }}"></a>
                        <h4><a href="{{ content.url }}" title="{{ content.title }}">{{ content.title }}</a></h4>
                        <div class="prize">{{ content.text }}</div>
                    </li>
                    {% endfor %}
                </ul>
            </div>
        </div>

        <div class="list_model model03">
            <div class="list_title clearfix">
                <h3 class="fl" id="model01">3F 家居家装</h3>
                <div class="subtitle fr">
                    <a @mouseenter="f3_tab=1" :class="f3_tab===1?'active':''">生活用品</a>
                    <a @mouseenter="f3_tab=2" :class="f3_tab===2?'active':''">厨房用品</a>
                </div>
            </div>
            <div class="goods_con clearfix">
                <div class="goods_banner fl">
                    <img src="{{ contents.index_3f_logo.0.image.url }}">
                    <div class="channel">
                        {% for content in contents.index_3f_pd %}
                        <a href="{{ content.url }}">{{ content.title }}</a>
                        {% endfor %}
                    </div>
                    <div class="key_words">
                        {% for content in contents.index_3f_bq %}
                        <a href="{{ content.url }}">{{ content.title }}</a>
                        {% endfor %}
                    </div>
                </div>
                <ul v-show="f3_tab===1" class="goods_list fl">
                    {% for content in contents.index_3f_shyp %}
                    <li>
                        <a href="{{ content.url }}" class="goods_pic"><img src="{{ content.image.url }}"></a>
                        <h4><a href="{{ content.url }}" title="{{ content.title }}">{{ content.title }}</a></h4>
                        <div class="prize">{{ content.text }}</div>
                    </li>
                    {% endfor %}
                </ul>
                <ul v-show="f3_tab===2" class="goods_list fl">
                    {% for content in contents.index_3f_cfyp %}
                    <li>
                        <a href="{{ content.url }}" class="goods_pic"><img src="{{ content.image.url }}"></a>
                        <h4><a href="{{ content.url }}" title="{{ content.title }}">{{ content.title }}</a></h4>
                        <div class="prize">{{ content.text }}</div>
                    </li>
                    {% endfor %}
                </ul>
            </div>
        </div>

    <div class="footer">
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
    <script type="text/javascript" src="js/index.js"></script>
</body>
</html>
```

测试:

![](/assets/goods_home.png)

