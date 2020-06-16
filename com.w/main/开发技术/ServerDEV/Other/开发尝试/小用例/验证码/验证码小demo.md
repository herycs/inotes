# 验证码小demo

## 1.自己实现

- 验证码程序：

  ```java
  package com.it.w.test;
  
  import javax.imageio.ImageIO;
  import javax.servlet.ServletException;
  import javax.servlet.http.HttpServlet;
  import javax.servlet.http.HttpServletRequest;
  import javax.servlet.http.HttpServletResponse;
  import java.awt.*;
  import java.awt.image.BufferedImage;
  import java.io.IOException;
  import java.util.Random;
  
  public class servlet_ConfigCode extends HttpServlet {
  
      private int LINE_NUB_MAX = 9;//干扰线数量
      @Override
      protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
  
          int width = 110;
          int height = 25;
          //内存中创建图像
          BufferedImage image = new BufferedImage(width, height, BufferedImage.TYPE_INT_BGR);
  
          //创建画笔
          Graphics g = image.createGraphics();
          
          //加背景
          g.setColor(Color.WHITE);
          
          //填充顺序，上右下左
          g.fillRect(1, 1, width-2, height-2);
          
          //给边框颜色
          g.setColor(Color.RED);
          g.drawRect(0, 0, width-1, height-1);
  
          //设置文本样式
          g.setColor(Color.BLACK);
          //字体 类型 样式 大小
          g.setFont(new Font("宋体", Font.BOLD|Font.ITALIC, 23));
  
          //生成随机数
          Random random = new Random();
          //图片添加文字
          int position = 20;
          for (int i = 0; i < 4; i++) {
              g.drawString(random.nextInt(10) + "",position,20);
                  position+=20;
          }
  
          //添加干扰线
          for (int i = 0; i < LINE_NUB_MAX; i++) {
              g.drawLine(random.nextInt(width), random.nextInt(height), random.nextInt(width), random.nextInt(height));
          }
  
          //图片以流的形式输出到客户端
          ImageIO.write(image, "jpg", resp.getOutputStream());
      }
  
      @Override
      protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          super.doPost(req, resp);
      }
  }
  
  ```

  相应的代码调用代码的设置
  
  ```js
  <script type="text/javascript">
  function changeCode() {
              var img = document.getElementsByTagName("img")[0];
              // img.src = "com/it/w/test/servlet_ConfigCodeDemo1";
      //保证页面刷新
              img.src = "*.req?time="+new Date().getTime();
      
          }
  </script>
  ```
  
  
  
  ```html
  <tr>
                  <td>请输入验证码：<input type="text"><img src="*.req" alt="这是验证码" onclick=changeCode()><a href="javascript:changeCode()">看不清，换一张</a></td>
   </tr>
  ```

## 2.使用架包

- 架包为：ValidateCode.jar

- 代码：

  ```java
  package com.it.w.test;
  
  
  import cn.dsna.util.images.ValidateCode;
  
  import javax.servlet.ServletException;
  import javax.servlet.http.HttpServlet;
  import javax.servlet.http.HttpServletRequest;
  import javax.servlet.http.HttpServletResponse;
  
  import java.io.IOException;
  
  public class servlet_ConfigCode extends HttpServlet {
      @Override
      protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          //设置浏览器不缓存
          resp.setHeader("prama", "no-cache");
          resp.setHeader("cache-control", "no-cache");
          resp.setHeader("expires","0");
  
          ValidateCode validCode = new ValidateCode(110,25,4,9);
          String code = validCode.getCode();
          validCode.write(resp.getOutputStream());
  
      }
  
      @Override
      protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          doGet(req, resp);
      }
  }
  
  ```

  