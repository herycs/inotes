# Android

# 1.基础学习

- 学习基础：第一行代码
- 参照文档：Android开发官网

# 2.商城开发流程

## 2.1启动页

- 应用第一个界面（启动界面）
- 延时后进入
- 判断是否第一次进入（是否引导）
- 动画
- 销毁

### 2.1.1建立Activity

- 新建EmptyActivity并命名为StartActivity

- 为StartActivity设置背景，并修改values/style.xml中

  - <style name="AppTheme" parent="Theme.AppCompat.Light.NoActionBar">

- 设置启动页StartActivity为主Activity

### 2.1.2延迟进入

- StartActivity文件中添加

  ```java
  Handler handler = new Handler();
  
  @Override
  protected void onCreate(Bundle savedInstanceState) {
      //。。。
      //        延时
      handler.postDelayed(new Runnable() {
          @Override
          public void run() {
  
  
          }
      }, 3000);
  }
  ```

### 2.1.3多图片展示

- 使用ViewPager
- 图片的全屏铺满：使用setBackgroundResource()

### 2.1.4最后一页出现跳转按钮

- 使用监听器，当当前posttion是图片数组中的最后一个时显示按钮

### 2.1.5点击按钮跳转到主页

- Activity中设置OnClick...

## 2.2主页布局

### 2.2.1基础布局

- 使用FragmentTabHost
  - 布局内采用线性布局

### 2.2.2底部按钮

- 建立多个Fragment
- 创建tab_item布局文件
- 创建图标文件
- 代码中将布局文件组合起来



