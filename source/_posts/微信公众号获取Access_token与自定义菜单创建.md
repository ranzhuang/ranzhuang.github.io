---
title: 微信公众号获取Access_token与自定义菜单创建
date: 2018-12-13
tags: 公众号开发
---

### 获取access_token

[获取access_token微信公众号官方文档](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421140183)

1、access_token简单说明

access_token是公众号的全局唯一接口调用凭据，公众号调用各接口时都需使用access_token，获取到access_token之后应将值进行保存，而不是每次都调用接口进行获取；服务器会返回expire_in（有效时间，单位秒），当expire_in过期之后再调用接口刷新access_token的值。

在调用接口获取access_token，需要参数AppID与AppSecret，AppID和AppSecret可在“微信公众平台-开发-基本配置”页中获得（需要已经成为开发者，且帐号没有异常状态）

接口调用请求说明:

    https请求方式: GET
    /**
    *  grant_type 	获取access_token填写client_credential
    *  appid    第三方用户唯一凭证
    *  secret    第三方用户唯一凭证密钥，即appsecret
    */
    https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=APPID&secret=APPSECRET

2、获取access_token 示例

在类WxService中，将appid、appsecret以及获取access_token的URL设置为静态常量；提供一个私有类方法用于调用微信接口获取access_token以及expires_in，并将值进行保存；再提供一个公共类方法，用于获取保存的值，如果保存的值为空或者时间已过期，就调用所提供的私有类方法获取值。

Java示例代码

    //获取ACCESS_TOKEN的URL
    private static final String ACCESS_TOKEN_URL = "https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=APPID&secret=APPSECRET";
    //appID
    private static final String APPID = "wx503297a5bc732744";
    //appsecret
    private static final String APPSECRET = "ab7c6be47ffe47ef4b44d13502afc60a";

示例方法


    /**
     * 获取AccessToken
     */
    private static void getCurrentAccessToken() {
        String url = ACCESS_TOKEN_URL.replace("APPID", APPID).replace("APPSECRET",APPSECRET);
        try {
            //调用GET方法获取数据
            String jsonStr = NetManagerUtil.net(url,null,"GET");
            //将json字符串转为对象
            JSONObject jsonObject = JSON.parseObject(jsonStr);
            AccessToken accessToken = new AccessToken(jsonObject.getString("access_token"), jsonObject.getString("expires_in"));
            cuttentAccessToken = accessToken;
        } catch (Exception e) {
            e.printStackTrace();
        }

    }

    public static String getAccessToken() {
        //如果token不存在或者token过期就去获取token
        if (cuttentAccessToken == null || cuttentAccessToken.isExpires()) {
            getCurrentAccessToken();
        }
        return cuttentAccessToken.getToken();
    }

在"getCurrentAccessToken"方法中用到了"NetManagerUtil"类中的"net"方法进行网络请求
"JSONObject"需要导入"fastjson"的jar包
AccessToken是新建的pojo类用于存放access_token与过期时间
"cuttentAccessToken"是类型为"AccessToken"的静态私有变量

(1)  编写"NetManagerUtil"类中的"net"方法进行网络请求

"NetManagerUtil"类中静态常量

    public static final String DEF_CHATSET = "UTF-8";
    public static final int DEF_CONN_TIMEOUT = 30000;
    public static final int DEF_READ_TIMEOUT = 30000;

网络请求方法


    /**
     *
     * @param strUrl 请求地址
     * @param params 请求参数
     * @param method 请求方法
     * @return  网络请求字符串
     * @throws Exception
     */
    public static String net(String strUrl, Map params,String method) throws Exception {
        HttpURLConnection conn = null;
        BufferedReader reader = null;
        String rs = null;
        try {
            StringBuffer sb = new StringBuffer();
            //GET方法链接设置
            if(method==null || method.equals("GET")){
                if (params!= null) {
                    strUrl = strUrl+"?"+urlencode(params);
                }
            }
            URL url = new URL(strUrl);
            //开连接
            conn = (HttpURLConnection) url.openConnection();
            //设置请求方式
            if(method==null || method.equals("GET")){
                conn.setRequestMethod("GET");
            }else{
                conn.setRequestMethod("POST");
                conn.setDoOutput(true);
            }
            //是否缓存
            conn.setUseCaches(false);
            //过期时间
            conn.setConnectTimeout(DEF_CONN_TIMEOUT);
            conn.setReadTimeout(DEF_READ_TIMEOUT);
            //是否重定向
            conn.setInstanceFollowRedirects(false);
            //连接
            conn.connect();
            //POST方法设置
            if (params!= null && method.equals("POST")) {
                try {
                    //输出流
                    DataOutputStream out = new DataOutputStream(conn.getOutputStream());
                    out.writeBytes(urlencode(params));
                } catch (Exception e) {
                    // TODO: handle exception
                }
            }
            //读取输入流
            InputStream is = conn.getInputStream();
            reader = new BufferedReader(new InputStreamReader(is, DEF_CHATSET));
            String strRead = null;
            while ((strRead = reader.readLine()) != null) {
                sb.append(strRead);
            }
            System.out.println(sb.toString());
            rs = sb.toString();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (reader != null) {
                reader.close();
            }
            if (conn != null) {
                conn.disconnect();
            }
        }
        return rs;
    }

    //将map型转为请求参数型
    public static String urlencode(Map<String,Object>data) {
        StringBuilder sb = new StringBuilder();
        for (Map.Entry i : data.entrySet()) {
            try {
                sb.append(i.getKey()).append("=").append(URLEncoder.encode(i.getValue()+"","UTF-8")).append("&");
            } catch (UnsupportedEncodingException e) {
                e.printStackTrace();
            }
        }
        return sb.toString();
    }

(2)  在pom.xml中导入"fastjson"的jar包

    <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>fastjson</artifactId>
      <version>1.1.41</version>
      <scope>compile</scope>
    </dependency>

(3)  新建名为 AccessToken 的pojo类，并添加变量以及构造方法以及判断token是否过期的方法

    /**
     * 成功的token
     */
    private String token;
    /**
     * 过期时间
     */
    private String expiresTimeout;

    public String getToken() {
        return token;
    }

    public void setToken(String token) {
        this.token = token;
    }

    public String getExpiresTimeout() {
        return expiresTimeout;
    }

    public void setExpiresTimeout(String expiresTimeout) {
        this.expiresTimeout = expiresTimeout;
    }

    /**
     * 构造方法
     * @param token     token
     * @param expiresIn 获取到的时间
     */
    public AccessToken(String token, String expiresIn) {
        this.token = token;
        this.expiresTimeout = System.currentTimeMillis() + Integer.parseInt(expiresIn) * 1000 +"";
    }

    /**
     * 是否过期
     * @return
     */
    public boolean isExpires() {
        return System.currentTimeMillis() > Long.parseLong(this.expiresTimeout);
    }

(4)  在WxService类中添加静态私有变量

    private static AccessToken cuttentAccessToken;

3 、获取access_token测试

新建名为WxTest的测试类，编写获取access_token的测试方法进行测试

    /**
     * 获取token
     */
    @Test
    public void getToken() {
        System.out.println(WxService.getAccessToken());
        System.out.println(WxService.getAccessToken());
    }

### 自定义菜单的创建

[自定义菜单创建微信公众号官方文档](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421141013)

1、自定义菜单简单说明

自定义菜单最多包括3个一级菜单，每个一级菜单最多包含5个二级菜单;一级菜单最多4个汉字，二级菜单最多7个汉字;测试时可以尝试取消关注公众账号后再次关注，则可以看到创建后的效果

自定义菜单的按钮类型有很多，比如click、view、scancode_push、scancode_waitmsg等；菜单的创建以click、view两种进行示例，其他按钮方式所传入的json数据格式参考[自定义菜单创建微信公众号官方文档](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421141013)

接口调用请求说明：

    http请求方式：POST（请使用https协议）
    [https://api.weixin.qq.com/cgi-bin/menu/create?access_token=ACCESS_TOKEN](https://api.weixin.qq.com/cgi-bin/menu/create?access_token=ACCESS_TOKEN)

2、json字符串所对应的对象封装

click和view的请求示例:

     {
     "button":[
     {    
          "type":"click",
          "name":"今日歌曲",
          "key":"V1001_TODAY_MUSIC"
      },
      {
           "name":"菜单",
           "sub_button":[
           {    
               "type":"view",
               "name":"搜索",
               "url":"http://www.soso.com/"
            },
            {
                 "type":"miniprogram",
                 "name":"wxa",
                 "url":"http://mp.weixin.qq.com",
                 "appid":"wx286b93c14bbf93aa",
                 "pagepath":"pages/lunar/index"
             },
            {
               "type":"click",
               "name":"赞一下我们",
               "key":"V1001_GOOD"
            }]
       }]
     }

通过这段示例可以看出创建按钮需传入一段json字符串，而且一个button键对应按钮所形成的数组（一级菜单），在按钮数组中还存在sub_button键所对应的子按钮数组（二级菜单）；所以最外层可以创建为WxButton类，在类中只有一个属性，属性名为button，类型为List。观察发现每个按钮都有name字段，所以创建一个BaseButton抽象类，再类中只提供一个属性，属性名为name，类型为String。每个不同类型的按钮都继承自BaseButton类去构建不同类型的pojo类；如果菜单中包含二级菜单，就创建类继承自BaseButton并提供一个名为sub_button的List<BaseButton>类型的属性，来代表一级菜单并包含子菜单这种情形。

创建抽象类"BaseButton"

    /**
     * 菜单名字
     */
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public BaseButton(String name) {
        this.name = name;
    }

创建类型为Click的ClickButton类，并继承自BaseButton

    /**
     * 菜单类型
     */
    private String type = "click";
    /**
     * 唯一键
     */
    private String key;

    public String getType() {
        return type;
    }

    public void setType(String type) {
        this.type = type;
    }

    public String getKey() {
        return key;
    }

    public void setKey(String key) {
        this.key = key;
    }

    public ClickButton(String name, String key) {
        super(name);
        this.key = key;
    }

创建按钮类型为View的ViewButton类，并继承自BaseButton

    /**
     * 菜单类型
     */
    private String type = "view";
    /**
     * 跳转地址
     */
    private String url;

    public String getType() {
        return type;
    }

    public void setType(String type) {
        this.type = type;
    }

    public String getUrl() {
        return url;
    }

    public void setUrl(String url) {
        this.url = url;
    }

    public ViewButton(String name, String url) {
        super(name);
        this.url = url;
    }

创建有子菜单的菜单，类名为SubButton，继承自BaseButton

    private List<BaseButton> sub_button = new ArrayList<>();

    public List<BaseButton> getSub_button() {
        return sub_button;
    }

    public void setSub_button(List<BaseButton> sub_button) {
        this.sub_button = sub_button;
    }

    public SubButton(String name) {
        super(name);
    }

创建名为WxButton的类用于生成最终json字符串的类

    private List<BaseButton> button = new ArrayList<>();

    public List<BaseButton> getButton() {
        return button;
    }

    public void setButton(List<BaseButton> button) {
        this.button = button;
    }

3、利用封装的pojo创建自定义菜单

常见名为"CreatButtonUtil"的类，在main方法中调用creatWxButton私有类方法进行菜单的创建

(1) 将所需要用到的文字定义为静态常量


    private static final String CREAT_MENU_URL = "https://api.weixin.qq.com/cgi-bin/menu/create?access_token=ACCESS_TOKEN";

    private static final String ONE_MENU_TITLE = "查找路线";
    public static final String ONE_MENU_TITLEKEY = "查找路线KEY";

    private static final String TWO_MENU_TITLE = "产品服务";
    private static final String TWO_MENU_SUBTITLE_ONE = "服务范围";
    private static final String TWO_MENU_SUBTITLE_ONEURL = "http://www.epochyoosure.com/fangan/";
    private static final String TWO_MENU_SUBOTITLE_TWO = "服务项目";
    private static final String TWO_MENU_SUBTITLE_TWOURL = "http://www.epochyoosure.com/a/youxuechanpin/";
    private static final String TWO_MENU_SUBTITLE_THREE = "新闻资讯";
    private static final String TWO_MENU_SUBTITLE_THREEURL = "http://www.epochyoosure.com/zixun/";

    private static final String THREE_MENU_TITLE = "艾派格";
    private static final String THREE_SUBTITLE_ONE = "私人订制";
    private static final String THREE_SUBTITLE_ONEKEY = "私人订制KEY";
    private static final String THREE_SUBTITLE_TWO = "关于我们";
    private static final String THREE_MENU_SUBTITLE_THREEURL = "http://www.epochyoosure.com/index.html";

(2)  创建"creatWxButton"私有类方法，在方法中创建菜单对象，并将对象转为json字符串， 调用"NetManagerUtil"所提供的post方法请求网络，传入所需要的参数，创建菜单

    private static void creatWxButton() {
        //第一个菜单
        ClickButton oneButton = new ClickButton(ONE_MENU_TITLE,ONE_MENU_TITLEKEY);
        //第二个菜单
        SubButton twoButton = new SubButton(TWO_MENU_TITLE);
        //第二个菜单的子菜单
        ViewButton oneSubButton = new ViewButton(TWO_MENU_SUBTITLE_ONE,TWO_MENU_SUBTITLE_ONEURL);
        ViewButton twoSubButton = new ViewButton(TWO_MENU_SUBOTITLE_TWO,TWO_MENU_SUBTITLE_TWOURL);
        ViewButton threeSubButton = new ViewButton(TWO_MENU_SUBTITLE_THREE,TWO_MENU_SUBTITLE_THREEURL);
        twoButton.getSub_button().add(threeSubButton);
        twoButton.getSub_button().add(twoSubButton);
        twoButton.getSub_button().add(oneSubButton);
        //第三个菜单
        SubButton threeButton = new SubButton(THREE_MENU_TITLE);
        //第三个菜单的子菜单
        ClickButton subButtonOne = new ClickButton(THREE_SUBTITLE_ONE,THREE_SUBTITLE_ONEKEY);
        ViewButton subButtonTwo = new ViewButton(THREE_SUBTITLE_TWO,THREE_MENU_SUBTITLE_THREEURL);
        threeButton.getSub_button().add(subButtonOne);
        threeButton.getSub_button().add(subButtonTwo);
        //添加到button对象中
        WxButton wxButton = new WxButton();
        wxButton.getButton().add(oneButton);
        wxButton.getButton().add(twoButton);
        wxButton.getButton().add(threeButton);
        //转json
        String jsonStr = JSONObject.toJSONString(wxButton);
        //替换token
        String url = CREAT_MENU_URL.replace("ACCESS_TOKEN", WxService.getAccessToken());
        //调用网络请求
        NetManagerUtil.post(url,jsonStr);
    }

(3)  类"NetManagerUtil"类中添加post方法

    /**
     * pust 请求
     * @param url  请求的url
     * @param data 请求的data
     * @return 返回json字符串
     */
    public static String post(String url,String data){
        try {
            URL urlObj = new URL(url);
            //开链接
            URLConnection connection = urlObj.openConnection();
            //要发送数据出去，需要设置为可发送数据状态
            connection.setDoOutput(true);
            //是否缓存
            connection.setUseCaches(false);
            //过期时间
            connection.setConnectTimeout(DEF_CONN_TIMEOUT);
            connection.setReadTimeout(DEF_READ_TIMEOUT);
            //获取输出流
            OutputStream outputStream = connection.getOutputStream();
            //写出数据
            if (data != null) {
                outputStream.write(data.getBytes());
            }
            outputStream.close();
            //获取输入流
            InputStream is = connection.getInputStream();
            BufferedReader reader = new BufferedReader(new InputStreamReader(is, DEF_CHATSET));
            String strRead = null;
            StringBuffer sb = new StringBuffer();

            while ((strRead = reader.readLine()) != null) {
                sb.append(strRead);
            }
            System.out.println(sb.toString());
            return sb.toString();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

(4)  在此类的main方法中调用"creatWxButton"方法

    public static void main(String[] args) {
        creatWxButton();
    }

(5)  运行main方法

控制台打印"{"errcode":0,"errmsg":"ok"}"，就创建成功

>提醒
在公众号中点击自定义菜单会收到自定义菜单事件，部分按钮类型需要对事件进行处理以对用户的操作进行响应，后续将持续更新消息事件的接收与回复
