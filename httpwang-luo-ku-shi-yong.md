\| 修改者 \| 版本 \| 内容 \| 时间 \|

\| ---- \| ---- \| ---- \| --------- \| 

\| 景天 \| 1.0 \| 初版 \| 2017.12.08

\#\# 说明

以 \[OkGo\]\(https://github.com/jeasonlzy/okhttp-OkGo\) 库为基础，进行一些改动，感谢原作者。这里的文档是精简版，主要写一下常用的方法，详细的方法，可以去OkGo那看看。注意，核心类名“OkGo”已改为"MMCHttp"



\#\#\#\# 接入说明

\`\`\`

compile 'com.mmc.core:mmcHttp:1.0.0-SNAPSHOT@aar'

compile 'com.squareup.okhttp3:okhttp:3.8.1'

compile 'com.google.code.gson:gson:2.8.0'

//如果要用rx，加入以下依赖

compile 'com.mmc.core:mmcHttpRx2:1.0.0-SNAPSHOT@aar'

compile 'io.reactivex.rxjava2:rxjava:2.1.1'

\`\`\`

\#\#\#\# 混淆

\`\`\`

\#okhttp

-dontwarn okhttp3.\*\*

-keep class okhttp3.\*\*{\*;}



\#okio

-dontwarn okio.\*\*

-keep class okio.\*\*{\*;}

\`\`\`

\#\#\#\# 初始化

在Application里面初始化，可以配置全局参数等等

\`\`\`

 @Override

    public void onCreate\(\) {

        super.onCreate\(\);



        initMMCHttp\(\);

    }



    private void initMMCHttp\(\) {

        //---------这里给出的是示例代码,告诉你可以这么传,实际使用的时候,根据需要传,不需要就不传-------------//

        HttpHeaders headers = new HttpHeaders\(\);

        headers.put\("commonHeaderKey1", "commonHeaderValue1"\);    //header不支持中文，不允许有特殊字符

        headers.put\("commonHeaderKey2", "commonHeaderValue2"\);

        HttpParams params = new HttpParams\(\);

        params.put\("commonParamsKey1", "commonParamsValue1"\);     //param支持中文,直接传,不要自己编码

        params.put\("commonParamsKey2", "这里支持中文参数"\);

        //----------------------------------------------------------------------------------------//



        OkHttpClient.Builder builder = new OkHttpClient.Builder\(\);

        HttpLoggingInterceptor loggingInterceptor = new HttpLoggingInterceptor\("MMCHttp"\);

        loggingInterceptor.setPrintLevel\(HttpLoggingInterceptor.Level.BODY\);        //log打印级别，决定了log显示的详细程度

        loggingInterceptor.setColorLevel\(Level.INFO\);                               //log颜色级别，决定了log在控制台显示的颜色

        builder.addInterceptor\(loggingInterceptor\);                                 //添加OkGo默认debug日志



        //超时时间设置，默认60秒

        builder.readTimeout\(MMCHttp.DEFAULT\_MILLISECONDS, TimeUnit.MILLISECONDS\);      //全局的读取超时时间

        builder.writeTimeout\(MMCHttp.DEFAULT\_MILLISECONDS, TimeUnit.MILLISECONDS\);     //全局的写入超时时间

        builder.connectTimeout\(MMCHttp.DEFAULT\_MILLISECONDS, TimeUnit.MILLISECONDS\);   //全局的连接超时时间



        builder.cookieJar\(new CookieJarImpl\(new DBCookieStore\(this\)\)\);              //使用数据库保持cookie，如果cookie不过期，则一直有效



        //信任所有证书,不安全有风险

        HttpsUtils.SSLParams sslParams1 = HttpsUtils.getSslSocketFactory\(\);

        builder.sslSocketFactory\(sslParams1.sSLSocketFactory, sslParams1.trustManager\);



        // 其他统一的配置

        MMCHttp.getInstance\(\).init\(this\)                           //必须调用初始化

                .setOkHttpClient\(builder.build\(\)\)               //建议设置OkHttpClient，不设置会使用默认的

                .setCacheMode\(CacheMode.NO\_CACHE\)               //全局统一缓存模式，默认不使用缓存，可以不传

                .setCacheTime\(CacheEntity.CACHE\_NEVER\_EXPIRE\)   //全局统一缓存时间，默认永不过期，可以不传

                .setRetryCount\(3\)                               //全局统一超时重连次数，默认为三次，那么最差的情况会请求4次\(一次原始请求，三次重连请求\)，不需要可以设置为0

                .addCommonHeaders\(headers\)                      //全局公共头

                .addMMCCommonParams\("lingjimiaosuan", "2000"\)  //MMC必传参数，第一个是应用标识（和后端定义），第二个是产品id

                .addCommonParams\(params\);                       //全局公共参数

    }

    

    必传参数有以下几个，如果有不同，可在具体接口上传字段覆盖掉这里写的值：

      /\*\*

     \* 添加mmc默认公共参数

     \*/

    public MMCHttp addMMCCommonParams\(String mmc\_channel, String mmc\_appid\) {

        if \(mCommonParams == null\) mCommonParams = new HttpParams\(\);

        HttpParams commonParams = new HttpParams\(\);

        commonParams.put\("mmc\_channel", mmc\_channel\);

        commonParams.put\("mmc\_appid", mmc\_appid\);

        if \("CN".equals\(Locale.getDefault\(\).getCountry\(\)\)\) {

            commonParams.put\("mmc\_lang", "zh\_cn"\);

        } else {

            commonParams.put\("mmc\_lang", Locale.getDefault\(\).getCountry\(\)\);

        }

        commonParams.put\("mmc\_platform", "Android"\);

        commonParams.put\("mmc\_devicesn", Util.getUUID\(context\)\);

        commonParams.put\("mmc\_code\_tag", PackageUtil.getAppVersionName\(context\)\);

        commonParams.put\("mmc\_operate\_tag", PackageUtil.getAppVersionName\(context\)\);

        commonParams.put\("mmc\_package", context.getApplicationContext\(\).getPackageName\(\)\);

        mCommonParams.put\(commonParams\);

        return this;

    }

\`\`\`

\#\#\#\# 使用示例

1、get请求和post请求，最简单的使用就是下面这样。支持8种请求方式，替换方法名即可。

\`\`\`

        MMCHttp.&lt;String&gt;get\("https://generalapi.linghit.com/app/parameter"\)

                .tag\(this\)

                .params\("appid", "1234568"\)

                .cacheMode\(CacheMode.IF\_NONE\_CACHE\_REQUEST\)//使用缓存，有五种模式，任由选择

               // .cacheMode\(CacheMode.VALID\_FOR\_TODAY\)// 如果是只想缓存当天，第二天重新请求，就设置这个缓存模式

                .cacheTime\(2 \* 60 \* 60 \* 1000\)//设置缓存时间为2小时

                .execute\(new StringCallback\(\) {

                    @Override

                    public void onSuccess\(Response&lt;String&gt; response\) {

                        Toast.makeText\(MethodActivity.this, response.body\(\), Toast.LENGTH\_SHORT\).show\(\);

                    }

                    

                    @Override

                    public void onCacheSuccess\(Response&lt;String&gt; response\) {

                        Log.i\("---", "缓存数据"\);

                    }



                    @Override

                    public void onError\(Response&lt;String&gt; response\) {

                        Toast.makeText\(MethodActivity.this, response.message\(\), Toast.LENGTH\_SHORT\).show\(\);

                    }

                }\);

\`\`\`

2、json解析回调使用，如果知道固定返回的内容，推荐用这个。  

如果使用缓存，记得实体类都要继承implements Serializable

\`\`\`

        //解析对象示例

        MMCHttp.&lt;GankModel&gt;get\("http://www.test.com"\).tag\(this\).cacheMode\(CacheMode.VALID\_FOR\_TODAY\).execute\(new JsonCallback&lt;GankModel&gt;\(\) {

            @Override

            public void onSuccess\(Response&lt;GankModel&gt; response\) {

                GankModel gankModel = response.body\(\);

            }

        }\);



        //解析对象list示例

        MMCHttp.&lt;List&lt;GankModel&gt;&gt;get\("http://www.test.com"\).tag\(this\).execute\(new JsonCallback&lt;List&lt;GankModel&gt;&gt;\(\) {

            @Override

            public void onSuccess\(Response&lt;List&lt;GankModel&gt;&gt; response\) {

                List&lt;GankModel&gt; gankModels = response.body\(\);

            }

        }\);

\`\`\`

3、下载文件

\`\`\`

        MMCHttp.&lt;File&gt;get\(Urls.URL\_DOWNLOAD\)

                .tag\(this\)

                .headers\("header1", "headerValue1"\)

                .params\("param1", "paramValue1"\)

                .execute\(new FileCallback\(Environment.getExternalStorageDirectory\(\).getPath\(\), "MMCHttp.apk"\) {



                    @Override

                    public void onStart\(Request&lt;File, ? extends Request&gt; request\) {

                        btnFileDownload.setText\("正在下载中"\);

                    }



                    @Override

                    public void onSuccess\(Response&lt;File&gt; response\) {

                        btnFileDownload.setText\("下载完成"\);

                    }



                    @Override

                    public void onError\(Response&lt;File&gt; response\) {

                        btnFileDownload.setText\("下载出错"\);

                    }



                    @Override

                    public void downloadProgress\(Progress progress\) {

                       btnFileDownload.setText\("下载进度：" +\(int\) \(progress.fraction \* 10000\)\);

                    }

                }\);

\`\`\`

4、上传文件

\`\`\`

        ArrayList&lt;File&gt; files = new ArrayList&lt;&gt;\(\);

        MMCHttp.&lt;String&gt;post\(Urls.URL\_FORM\_UPLOAD\)

                .tag\(this\)

                .params\("file1", new File\("文件路径"\)\)   //这种方式为一个key，对应一个文件

                .params\("file2", new File\("文件路径"\)\)

                .params\("file3", new File\("文件路径"\)\)

                //  .addFileParams\("file", files\)           // 这种方式为同一个key，上传多个文件

                .execute\(new StringCallback\(\) {

                    @Override

                    public void onSuccess\(Response&lt;String&gt; response\) {

                        Log.i\("MMCUpload", "上传成功"\);

                    }



                    @Override

                    public void onError\(Response&lt;String&gt; response\) {

                        Log.i\("MMCUpload", "上传失败"\);

                    }



                    @Override

                    public void uploadProgress\(Progress progress\) {

                        Log.i\("MMCUpload", "上传进度：" + \(int\) \(progress.fraction \* 10000\)\);

                    }

                }\);

\`\`\`

5、记得如果有请求，就要在关闭页面的时候取消请求，避免发生灵异事件

\`\`\`

    @Override

    protected void onDestroy\(\) {

        super.onDestroy\(\);

        //Activity销毁时，取消网络请求。

        MMCHttp.getInstance\(\).cancelTag\(this\);

    }

\`\`\`

\#\#\#\# 大家试用过程中，有啥不满的一定要说啊，然后及早更新这个库，慢慢完善。

