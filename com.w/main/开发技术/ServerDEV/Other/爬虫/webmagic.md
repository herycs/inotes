# webMagic

# 1.环境部署

- 直接去[Maven仓库](https://mvnrepository.com/search?q=webmagic)查找依赖

- 或者，复制下面的maven仓库坐标

  > 以下使用的是0.7.3，对SSL的支持不完全，对支持SSL1.2的网页会抛出SSL的异常
  >
  > - 可待0.7.4更新
  > - 直接下载最新版（需要去github[下载最新版](https://github.com/code4craft/webmagic/issues/701)到本地仓库），将maven中的核心依赖core改为本地的 

  ```xml
  <!--    webMagic-->
  <!-- https://mvnrepository.com/artifact/us.codecraft/webmagic-core -->
  <dependency>
      <groupId>us.codecraft</groupId>
      <artifactId>webmagic-core</artifactId>
      <version>0.7.3</version>
  </dependency>
  <!-- https://mvnrepository.com/artifact/us.codecraft/webmagic-extension -->
  <dependency>
      <groupId>us.codecraft</groupId>
      <artifactId>webmagic-extension</artifactId>
      <version>0.7.3</version>
  </dependency>
  ```

# 2.webMagic

## 2.1抽取技术

- XPath, 正则表达式， CSS选择器

- 简单使用

  ```java
  public class webMagicTest01 implements PageProcessor {
  
      private Site site = Site.me();
  
      public void process(Page page) {
  
          //        css选择器
          page.putField("div", page.getHtml().css("div.categories a").all());
  
          //      XPath
          page.putField("divByXPath", page.getHtml().xpath("//div[@id=categorys-2014]/div/a"));
  
          //        正则表达式
          page.putField("divByRegx", page.getHtml().css("div#service-2017 ol li").regex(".*多.*|.*全.*" +
                                                                                        "").all());
          
          //爬取链接
          page.addTargetRequests(page.getHtml().css("div#dd-inner").links().all());
      }
  
      public Site getSite() {
          return site;
      }
  
      public static void main(String[] args) {
          Spider.create(new webMagicTest01()).addUrl("https://www.jd.com/allSort.aspx").run();
  
          Spider.create(new webMagicTest01())
              .addUrl("https://www.jd.com/allSort.aspx")
              .addPipeline(new FilePipeline("C:\\Users\\tese\\Desktop\\result"))//爬取到的信息保存到文件
              .thread(5)//配置处理线程数量
              .run();
      }
  }
  ```

  ## 3.去重

  ## 3.1URL去重

  - HashSet
  
    > 优：使用方便
  >
    > 缺：内存大，性能低

  - Redis
  
    > 优：速度快，不占爬虫服务器资源
  >
    > 缺：提供Redis服务器，成本提高

  - BloomFilter
  
    > 使用位数组实现数据的存放
  >
    > 当由连接计算出的数字都没有被占用时认为不重复，否则重复
  
    > 优：内存占用（约HashSet0.1倍）
    >
    > 缺：有误判的可能

## 3.2内容去重

- 指纹密码对比

  > 生成文档的指纹密码，一致则认为重复

- BloomFilter

  > 对文章内容计算的数字，有少量不重复认为是不一样的内容

- KMP算法

  > 改进的字符串比较算法，利用匹配失败后的信息，减少模式串与主串的匹配次数

  > 缺：时空复杂度高，不适合大数据量对比

- 最长公共子串

- 后缀数组

- DFA

- simhash(Google)算法

  > 1.分词，对文章分词
  >
  > 2.hash，将算为hash值
  >
  > 3.加权，对2的结果进行加权处理
  >
  > 4.合并，对3中的序列值累加
  >
  > 5.降为，对4中计算出来的结果转换为01串，得出签名

# 4.代理

- [米扑代理](https://proxy.mimvp.com/)
- [西刺赖代理](https://www.xicidaili.com/)

# 5.ElasticSearch

- 下载解压后，启动执行bat，访问地址：127.0.0.1:9200

- 若需图形化界面

  > 安装：elasticSearch-head
  >
  > 安装：node.js
  >
  > ​	使用node.js将grunt安装为全局命令
  >
  > ​	npm install -g crunt-cli
  >
  > 修改elasticsearch配置文件.yml文件
  >
  > ```
  > http.cors.enabled:true
  > 
  > http.cors.allow-origin:"*"
  > 
  > network.host:127.0.0.1
  > ```
  >
  > 进入head目录使用 grunt server
  >
  > ```sql
  > -- 成功场景：
  > Started connected web server on http://localhost:9100
  > -- 失败:
  > npm i grunt
  > cnpm i grunt-contrib-clean grunt-contrib-concat grunt-contrib-watch grunt-contrib-connect grunt-contrib-copy grunt-contrib-jasmine -S
  > ```

- IK分词器

  - 下载elasticsearch-analysis，解压后的elasticsearch文件夹，更名为ik后复制到elasticsearch的plugins文件夹下

