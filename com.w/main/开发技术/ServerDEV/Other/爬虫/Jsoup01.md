# Jsoup

# 1.获取文档

```java
String url = "C:\\Users\\ANGLE0\\Desktop\\test.html";
Document doc = null;

try {
    doc = Jsoup.parse(new File(url), "utf-8");
} catch (IOException e) {
    e.printStackTrace();
}
```



# 2.从文档中获取元素

> 操作dom

## 2.1通过值获取元素

```java
private static void getElement() {
    String url = "C:\\Users\\ANGLE0\\Desktop\\test.html";
    Document doc = null;

    try {
        doc = Jsoup.parse(new File(url), "utf-8");
    } catch (IOException e) {
        e.printStackTrace();
    }

    //由ID获取元素
    Element elementByID = doc.getElementById("id1");
    //由属性获取元素
    Element elementByAttribute = doc.getElementsByAttribute("span").first();
    //由class获取元素
    Element elementByClass = doc.getElementsByClass("class_lay class_btn").first();
    //由标签获取元素
    Element elementByTag = doc.getElementsByTag("li > a").first();

    System.out.println("elementByID:"+elementByID);
    System.out.println("elementByAttribute:"+elementByAttribute);
    System.out.println("elementByClass:"+elementByClass);
    System.out.println("elementByTag:"+elementByTag);
}
```



## 2.2通过属性获取元素

> 直接选择器

```java
private static void getElementByVlaue() {
    String url = "C:\\Users\\ANGLE0\\Desktop\\test.html";
    Document doc = null;

    try {
        doc = Jsoup.parse(new File(url), "utf-8");
    } catch (IOException e) {
        e.printStackTrace();
    }

    //标签查找元素
    System.out.println("标签查找元素");
    Elements elements = doc.select("span");
    for(Element ele: elements){
        System.out.println(ele.toString());
    }

    //id查找元素
    Element element = doc.select("#id1").first();
    System.out.println("id查找元素:\n"+element.toString());

    //className查找元素
    Element element1 = doc.select(".class_lay").first();
    System.out.println("className查找元素：\n"+element1.toString());

    //属性查找元素
    Element element2 = doc.select("[btn]").first();
    System.out.println("属性查找元素:\n"+element2);

    //属性值查找元素
    Elements element3 = doc.select("[class=class_btn]");
    for (Element ii : element3){
        System.out.println("属性值查找元素:\n"+ii.toString());
    }
}
```



# 3.从元素中获取属性 

```java
private static void getAttributeFromElement() {
    String url = "C:\\Users\\ANGLE0\\Desktop\\test.html";
    Document doc = null;

    try {
        doc = Jsoup.parse(new File(url), "utf-8");
    } catch (IOException e) {
        e.printStackTrace();
    }

    Element element = doc.getElementById("id1");

    //元素中获取所有属性
    String elementID = element.id();
    //元素中获取class
    String className = element.className();
    Set<String> classNames = element.classNames();
    //元素中获取属性值
    String attrID = element.attr("id");
    String attrValue = element.attr("value");
    String attrClass = element.attr("class");
    //元素中获取文本
    String text = element.text();

    System.out.println("获取到的元素的id为："+elementID);

    System.out.println("获取到的元素的样式名为："+className);

    System.out.println("获取到的样式列表->");
    for(String i : classNames){
        System.out.println(i.toString());
    }

    System.out.println("获取到的元素的属性列表->");
    System.out.println("属性ID为："+attrID);
    System.out.println("属性value为："+attrValue);
    System.out.println("属性class为："+attrClass);

    System.out.println("元素的文本为："+text);
}
```



# 4.组合选择器

> 将多种属性组合后作为检索目标元素的唯一属性，定位目标

- 元素+id：xxx#id
- 元素+className：xxx.class1
- 父元素+子元素组：ul li

> ​	父元素下所有子元素

- 父元素下直接子元素：

  ul > li

> 父元素下第一个子元素

​		ul > *

> 父元素下所有直接子元素，父元素下同级的所有子元素