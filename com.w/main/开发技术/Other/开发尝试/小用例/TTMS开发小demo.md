# 开发小demo

## 1.前端总页面(index.jsp)

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<html>

<head>
    <title>有点大雪影院管理系统</title>
    <meta content="text/html" charset="UTF-8">
    <link rel="stylesheet" type="text/css" href="themes/bootstrap/easyui.css">
    <link rel="stylesheet" type="text/css" href="themes/icon.css">
    <link rel="stylesheet" type="text/css" href="themes/color.css">
    <link rel="stylesheet" type="text/css" href="themes/local_css/index.css">
    <script type="text/javascript" src="js/jquery.min.js"></script>
    <script type="text/javascript" src="js/jquery.easyui.min.js"></script>
    <script type="text/javascript" src="locale/easyui-lang-zh_CN.js"></script>
</head>

<body class="easyui-layout">
<div data-options="region:'north',hideCollapsedContent:false, title:'<center>有点大雪影院管理系统</center>', border:false, " style="height:100px;">
    <div id="m_top" style="display:inline-block;padding-top: 8px">
        <span style="text-align: center;color:royalblue;margin-left: 400px;">
            <img src="themes/icons/kodinger_logo.png" alt="有点大雪影院管理系统" style="width: 50px; height: 50px;">
            <span style="font-size: 33px;">欢迎来到有点大雪影院管理系统</span>
            <span style="font-size: 20px; margin-left: 400px;">
                欢迎您，${u.u_name}!
                <a href="logout">注销</a>
            </span>
        </span>

    </div>
</div>
<div data-options="region:'south',hideCollapsedContent:false,collapsed:true,title:'版权信息',border:'false',split:true" style="height:70px;">
    <div id="belongs" style=" font-family: 'Lucida Sans'; text-align: center;">
        <p>Copyright &copy; 2019.5.15 XQRJ</p>
    </div>
</div>
<div data-options="region:'east',split:true,hideCollapsedContent:false,collapsed:true,title:'系统通知'"
     style="width:100px;"></div>
<div id="west_menu" data-options="region:'west',hideCollapsedContent:false,collapsed:false,title:'菜单',split:true" style="width:180px;">
        <div class="easyui-accordion" data-options="fit:true,border:false" style="width:248px; height: 80px">

            <div title="管理剧目" data-options="iconCls:'icon-film'" style="overflow:auto;padding:10px;">
                <ul data-options="iconCls:'icon-staff'" >
                    <li><a href="#" title="剧目管理" onclick="addTab('剧目管理','m_film.jsp')">剧目管理</a></li>
             
                </ul>
            </div>
         </div>
</div>
<div data-options="region:'center',title:'center title'" style="padding:5px;background:#eee;">
    <div id="tt" class="easyui-tabs" data-options="fit:true">
        <div title="系统欢迎页面" style="padding:10px;">
            欢迎来到 <span style="font-size: 23px;">有点大雪影院</span><br>
        </div>
    </div>
</div>

<%--自定义js,用于页面处理--%>
<script type="text/javascript" src="/js/local_js/index.js"></script>

</body>
</html>
```

## 2.总页面js文件(index.js)

```javascript
$(function () {
    $("a[title]").click(function () {
        var text = $(this).text();
        var href = $(this).attr("title");
        //判断当前右边是否已有相应的tab
        if ($("#tt").tabs("exists", text)) {
            $("#tt").tabs("select", text);
        } else {
            //如果没有则创建一个新的tab，否则切换到当前tag
            $("#tt").tabs("add", {
                title: text,
                closable: true,
                content: '<iframe title=' + text + 'src=' + href + ' frameborder="0" width="100%" height="100%" />'
                //href:默认通过url地址加载远程的页面，但是仅仅是body部分
                //href:'send_category_query.action'
            });
        }

    });
});

function addTab(title, url) {
    if ($('#tt').tabs('exists', title)) {
        $('#tt').tabs('select', title);
    } else {
        var content = '<iframe scrolling="auto" frameborder="0"  src="' + url + '" style="width:100%;height:100%;"></iframe>';
        $('#tt').tabs('add', {
            title: title,
            content: content,
            closable: true
        });
    }
}
```

## 3.分页面之一（m_film.jsp）

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>M_FILM</title>
    <link rel="stylesheet" type="text/css" href="themes/bootstrap/easyui.css">
    <link rel="stylesheet" type="text/css" href="themes/icon.css">
    <link rel="stylesheet" type="text/css" href="themes/color.css">
    <script type="text/javascript" src="js/jquery.min.js"></script>
    <script type="text/javascript" src="js/jquery.easyui.min.js"></script>
    <script type="text/javascript" src="locale/easyui-lang-zh_CN.js"></script>
</head>
<body>
<div id="c1">
    <table id="m_film" class="easyui-datagrid" style="width: 100%;height:600px;" data-options="
                            title:'剧目管理',
                            iconCls: 'icon-film',
                            url: 'search_all_plays',
                            method: 'get',
                            pagination:true,
                            toolbar: '#c1_tool'
                        ">
    </table>
    <div id="c1_tool" style="padding: 5px;">
        <a id="n1" href="javascript:void(0)" class="easyui-linkbutton"
           data-options="iconCls:'icon-add',plain:true" onclick="nappend()">增加</a>
        <a id="n2" href="javascript:void(0)" class="easyui-linkbutton"
           data-options="iconCls:'icon-edit',plain:true" onclick="nedit()">编辑</a>
        <a id="n3" href="javascript:void(0)" class="easyui-linkbutton"
           data-options="iconCls:'icon-remove',plain:true" onclick="nremoveit()">删除</a>
        <a id="n4" href="javascript:void(0)" class="easyui-linkbutton"
           data-options="iconCls:'icon-save',plain:true" onclick="naccept()">保存</a>
        <a id="n5" href="javascript:void(0)" class="easyui-linkbutton"
           data-options="iconCls:'icon-undo',plain:true" onclick="nrejectChanges()">放弃修改</a>
        <input id="n6" class="easyui-searchbox" style="margin-left: 100px; width:250px" data-options="prompt:'请输入查询信息',menu:'#c1_tool_search',
                            searcher: function (value, name) {
                                javascript:nsearch(value, name);
                            }" />
    </div>
    <div id="c1_tool_search" style="width:120px">
        <div data-options="name:'play_id'">剧目类型ID</div>
        <div data-options="name:'play_type_id'">剧目语言ID</div>
        <div data-options="name:'play_lang_id'">剧目名称</div>
        <div data-options="name:'play_name'">剧情简介</div>
        <div data-options="name:'play_status'">剧目状态</div>
    </div>
</div>
<script type="text/javascript" src="/js/local_js/m_film.js"></script>
</body>
</html>

```

## 4.分页面的js文件(m_film.js)

```js
$('#m_film').datagrid({
    // title: '座位管理',             //标题
    // iconCls: 'icon-mans',        //图标
    // width: 1303,                 //表格宽度
    // height: 586,                 //表格高度
    // url: 'find_all_users',       //表格数据源
    // method: 'get',               //表格数据请求方式
    // pagination:true,             //表格分页信息
    // align: 'center',             //列数据对齐方式
    // halign: 'center',            //列标题对齐方式
    // frozenColumns: [u_id],       //设置冻结列
    pageList: [10, 15, 20, 30, 40, 50],
    fitColumns: true,               //自适应宽
    autoRowHeight: true,            //自适应高
    nowrap: true,                   //内容超宽时不许换行
    rownumbers: true,               //显示行号
    striped: true,                  //显示斑马线效果
    ctrlSelect: true,              // 选择限制
    columns: [[
        { field: 'ck', title: '选中状态', checkbox: true },
        {
            field: 'play_id', title: '剧目ID', width: 30, align: 'center', halign: 'center',
            sortable: true,
            sorter: function (a, b) {
                return (parseFloat(a) > parseFloat(b) ? 1 : -1);
            }
        },
        {
            field: 'play_type_id', title: '剧目类型ID', width: 70, align: 'center', halign: 'center',
            formatter: function (val, row) {
                if (val == 1) {
                    return '<span>' + '科幻' + '</span>';
                } else if (val == 2) {
                    return '<span>' + '喜剧' + '</span>';
                } else if (val == 3) {
                    return '<span>' + '冒险' + '</span>';
                }else if (val == 4) {
                    return '<span>' + '幻想' + '</span>';
                }else if (val == 5) {
                    return '<span>' + '悬念' + '</span>';
                }else if (val == 6) {
                    return '<span>' + '惊悚' + '</span>';
                }else if (val == 7) {
                    return '<span>' + '记录' + '</span>';
                }else if (val == 8) {
                    return '<span>' + '战争' + '</span>';
                }else if (val == 9) {
                    return '<span>' + '西部' + '</span>';
                }else if (val == 10) {
                    return '<span>' + '爱情' + '</span>';
                }else if (val == 11) {
                    return '<span>' + '剧情' + '</span>';
                }else if (val == 12) {
                    return '<span>' + '恐怖' + '</span>';
                }else if (val == 13) {
                    return '<span>' + '动作' + '</span>';
                }else if (val == 14) {
                    return '<span>' + '音乐' + '</span>';
                }else if (val == 15) {
                    return '<span>' + '家庭' + '</span>';
                }else if (val == 16) {
                    return '<span>' + '犯罪' + '</span>';
                }else {
                    return '<span>' + '其它类型' + '</span>';
                }
            },
            editor: {
                type: 'combobox',
                options: {
                    required: true,
                    valueField: "value", textField: "text",
                    data: [
                        { value: '1', text: '科幻' },
                        { value: '2', text: '喜剧' },
                        { value: '3', text: '冒险' },
                        { value: '4', text: '幻想' },
                        { value: '5', text: '悬念' },
                        { value: '6', text: '惊悚' },
                        { value: '7', text: '记录' },
                        { value: '8', text: '战争' },
                        { value: '9', text: '西部' },
                        { value: '10', text: '爱情' },
                        { value: '11', text: '剧情' },
                        { value: '12', text: '恐怖' },
                        { value: '13', text: '动作' },
                        { value: '14', text: '音乐' },
                        { value: '15', text: '家庭' },
                        { value: '16', text: '其它类型' },
                    ],

                }
            }
        },
        {
            field: 'play_lang_id', title: '剧目语言ID', width: 70, align: 'center', halign: 'center',
            formatter: function (val, row) {
                if (val == 1) {
                    return '<span style=color:black;>' + '中文' + '</span>';
                } else if (val == 2) {
                    return '<span style=color:black;>' + '英语' + '</span>';
                } else if(val == 3){
                    return '<span style=color:black;>' + '其它语言' + '</span>';
                }
            },
            editor: {
                type: 'combobox',
                options: {
                    required: true,
                    valueField: "value", textField: "text",
                    data: [
                        { value: '1', text: '中文/国语' },
                        { value: '2', text: '英语' },
                        { value: '3', text: '其它语种' },
                    ],

                }
            }
        },
        {
            field: 'play_name', title: '剧目名称', width: 100, align: 'center', halign: 'center',
            editor: {
                type: 'text',
                options: {
                    required: true,
                    validType: 'isEmpty'
                }

            }
        },
        {
            field: 'play_introduction', title: '剧情简介', width: 200, align: 'center', halign: 'center',
            editor: {
                type: 'text',
            }
        },
        {
            field: 'play_image', title: '剧目海报', width: 100, align: 'center', halign: 'center',
            editor: {
                type: 'filebox',
            }
        },
        {
            field: 'play_length', title: '演出时长(min)', width: 100, align: 'center', halign: 'center',
            editor:{
                type: 'text',//文本框为数字
            }
        },
        {
            field: 'play_ticket_price', title: '基准票价', width: 100, align: 'center', halign: 'center',
            formatter: function(val){
              return "<span>￥"+val+"</span>"
            },
            editor: {
                type: 'text',
                options: {
                    required: true,
                    validType: 'isEmpty'
                }
            }
        },
        {
            field: 'play_status', title: '剧目状态', width: 100, align: 'center', halign: 'center',
            formatter: function (val, row) {
                if (val == 1) {
                    return '<span style=color:red;>' + '已安排演出' + '</span>';
                } else if (val == 0) {
                    return '<span style=color:green;>' + '未安排演出' + '</span>';
                } else if(val == -1){
                    return '<span style=color:grey;>' + '已下线' + '</span>';
                }else{
                    return '<span style=color:grey;>' + '剧目状态待确定' + '</span>';
                }
            },
            editor: {
                type: 'combobox',
                options: {
                    required: true,
                    valueField: "value", textField: "text",
                    data: [
                        { value: '1', text: '安排演出' },
                        { value: '0', text: '待安排' },
                        { value: '-1', text: '下线' },
                    ],

                }
            },
        }
    ]],
    onBeforeEdit: function (index, row) {
        //编辑前触发
        objn.editIndex = undefined;
    },
    onBeginEdit: function (index, row) {
        //进入编辑状态触发
        objn.enable();
        objn.editIndex = index;
    },
    onAfterEdit: function (index, row, changes) {
        //编辑结束触发
        objn.disable();
        //判断修改类型
        var insert = $('#m_film').datagrid('getChanges', 'inserted');
        var updated = $('#m_film').datagrid('getChanges', 'updated');
        var type = insert.length > 0 ? 'add' : 'update';
        //将判断的修改类型及结果交给ajax处理
        objn.manger(type, row);
        objn.editIndex = undefined;
    },
    onCancelEdit: function (index, row, changes) {
        //取消编辑
        objn.disable();
        objn.editIndex = undefined;
    },
    onDblClickRow: function (index, field) {
        if (objn.editIndex != index) {
            if (objn.endEditing()) {             //判断原来的行，通过验证则进入新行
                $(this).datagrid('beginEdit', index);
                var ed = $(this).datagrid('getEditor', { index: index, field: field });
                if (ed !== null) {
                    $(ed.target).focus();
                    objn.editIndex = index;
                }
            } else {
                $(this).datagrid('unselectRow', index);         //否则取消选择行
            }
        }
    },
    onLoadSuccess: function (data) {
        objn.disable();          //编辑状态，刷新数据，保存，放弃按钮应不可用
    },


});

//-----------------------------------------

//表格按钮控制
var objn = {

    editIndex: undefined,          //行编辑

    endEditing: function () {         //结束编辑
        if (this.editIndex == undefined) return false;
        if ($('#m_film').datagrid('validateRow', this.editIndex)) {
            $('#m_film').datagrid('endEdit', this.editIndex);
            $('#m_film').datagrid('unselectRow', this.editIndex);
            this.editIndex = undefined;
            return true;
        } else {
            return false;
        }
    },
    manger: function (type, row) {   //将前台处理交个后台
        // alert("修改的行的信息为："+JSON.stringify(row));
        $.ajax({
            type: 'get',
            url: 'change_play',
            data: {
                type: type,
                row: JSON.stringify(row),
            },
            beforeSend: function () {
                $('#m_film').datagrid('loading');
            },
            success: function (data) {
                $('#m_film').datagrid('reload');            //重新加载当前页
                $('#m_film').datagrid('loaded');            //显示加载完成
                $.messager.show({
                    title: '信息提示',
                    msg: data,
                })
            }
        })
    },
    mangerSearch: function (value, name) {                  //将前台处理交个后台
        $.ajax({
            method: 'get',
            url: 'search_play',
            data: {
                type: 'schplay',
                value: value,
                name: name,
            },
            beforeSend: function () {
                $('#m_film').datagrid('loading');
            },
            success: function (data) {
                $('#m_film').datagrid('reload');            //重新加载当前页
                $('#m_film').datagrid('loaded');            //显示加载完成
                $.messager.show({
                    title: '信息提示',
                    msg: data,
                });
            }
        })
    },
    enable: function () {
        $('#n1').linkbutton('disable');
        $('#n2').linkbutton('disable');
        $('#n3').linkbutton('disable');
        $('#n4').linkbutton('enable');
        $('#n5').linkbutton('enable');
    },
    disable: function () {
        $('#n1').linkbutton('enable');
        $('#n2').linkbutton('enable');
        $('#n3').linkbutton('enable');
        $('#n4').linkbutton('disable');
        $('#n5').linkbutton('disable');
        this.editIndex = undefined;         //放弃修改后，能双击进入下一个编辑
    }
}

//-----------------------------------------

//放弃修改
function nrejectChanges() {
    $.messager.confirm('确认提示', '您确认放弃修改吗？', function (r) {
        if (r) $('#m_film').datagrid('rejectChanges');
    })
}
//保存
function naccept() {
    if (!$('#m_film').datagrid('validateRow', objn.editIndex)) {
        $.messager.alert('警告', '当前行没有通过验证，不能保存数据！', 'warning');
    } else {
        $('#m_film').datagrid('endEdit', objn.editIndex);
    }
}
//添加
function nappend() {
    objn.editIndex = $('#m_film').datagrid('getRows').length;       //最后一条记录的位置
    $('#m_film').datagrid('appendRow', {}).datagrid('beginEdit', objn.editIndex);
}
//修改
function nedit() {
    var rows = $('#m_film').datagrid('getSelections');          //获取选择的所有记录
    if (rows.length == 0) {
        $.messager.alert('信息提示', '请选择要编辑的行！', 'warning');
    } else if (rows.length == 1) {
        objn.editIndex = $('#m_film').datagrid('getRowIndex', rows[0]);
        $('#m_film').datagrid('beginEdit', objn.editIndex);
    } else {
        $.messager.alert('信息提示', '编辑记录时请不要选择多行！', 'warning');
    }
}
//删除
function nremoveit() {
    var rows = $('#m_film').datagrid('getSelections');
    if (rows.length == 0) {
        $.messager.alert('信息提示', '请选择要删除的行', 'info');
    } else {
        $.messager.confirm('确认提示', '您确定要删除所选记录吗？', function (r) {
            if (r) {
                //由于删除操作并不会触发onAfterEdit事件，so,换个方法
                // $.each(rows, function (i) {
                //     var index = $('#m_mans').datagrid('cancelEdit', obj.editIndex);
                //     $('#m_mans').datagrid('deleteRow', obj.editIndex);
                // })
                var ids = [];
                for (i = 0; i < rows.length; i++) {
                    ids.push(rows[i].play_id);
                }
                objn.manger('delete', ids.join(','));
            }
        });
    }
}
//查找
function nsearch(value, name) {
    if (value == '' || value == null) {
        $.messager.alert('信息提示', '请输入您的搜索信息', 'info');
    } else {
        objn.mangerSearch(value, name);//传给manger后，实际name为查找类型，value为值，操作为select
    }
}
```

## 5.后台：

> 总述：后台结构为
>
> - dao数据交互——直接和数据库打交道
> - daoImpl
> - service调用有关的dao 
> - serviceImpl
> - daomain使用到的类定义
> - test测试程序数学位置
> - util工具包
> - servlet具体提供给前台的端口

### 5.1daomain

```java
package com.w.ttms.domian;

public class Play {

    private int play_id;                          //剧目ID
    private int play_type_id;                     //剧目类型ID
    private int play_lang_id;                     //剧目语言ID
    private String play_name;                        //剧目名称
    private String play_introduction;                //剧目简介
    private String play_image;                       //剧目海报
    private int play_length;                      //演出时长
    private double play_ticket_price;                //基准票价
    private String play_status;                      //剧目状态

    public int getPlay_id() {
        return play_id;
    }

    public void setPlay_id(int play_id) {
        this.play_id = play_id;
    }

    public int getPlay_type_id() {
        return play_type_id;
    }

    public void setPlay_type_id(int play_type_id) {
        this.play_type_id = play_type_id;
    }

    public int getPlay_lang_id() {
        return play_lang_id;
    }

    public void setPlay_lang_id(int play_lang_id) {
        this.play_lang_id = play_lang_id;
    }

    public String getPlay_name() {
        return play_name;
    }

    public void setPlay_name(String play_name) {
        this.play_name = play_name;
    }

    public String getPlay_introduction() {
        return play_introduction;
    }

    public void setPlay_introduction(String play_introduction) {
        this.play_introduction = play_introduction;
    }

    public String getPlay_image() {
        return play_image;
    }

    public void setPlay_image(String play_image) {
        this.play_image = play_image;
    }

    public int getPlay_length() {
        return play_length;
    }

    public void setPlay_length(int play_length) {
        this.play_length = play_length;
    }

    public double getPlay_ticket_price() {
        return play_ticket_price;
    }

    public void setPlay_ticket_price(double play_ticket_price) {
        this.play_ticket_price = play_ticket_price;
    }

    public String getPlay_status() {
        return play_status;
    }

    public void setPlay_status(String play_status) {
        this.play_status = play_status;
    }
}

```



### 5.2daoImpl

```java
package com.w.ttms.dao.daoImpl;

import com.w.ttms.dao.playDao;
import com.w.ttms.domian.Play;
import com.w.ttms.util.DBUtils;
import org.apache.commons.dbutils.QueryRunner;
import org.apache.commons.dbutils.handlers.BeanHandler;
import org.apache.commons.dbutils.handlers.BeanListHandler;

import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.List;

public class playDaoImpl implements playDao {
    @Override
    public List<Play> findAllPlay() throws SQLException {
        QueryRunner qr = new QueryRunner(DBUtils.getDataSource());
        return qr.query("select * from play", new BeanListHandler<>(Play.class));
    }

    @Override
    public List<Play> searchPlayUnderCondition(String name, String value) throws SQLException {
        QueryRunner qr = new QueryRunner(DBUtils.getDataSource());
        String sql = "select * from play where ? = ?";
        return qr.query(sql, new BeanListHandler<>(Play.class), name, value);
    }

    @Override
    public List<Play> searchPlayByID(int play_id) throws SQLException {
        QueryRunner qr = new QueryRunner(DBUtils.getDataSource());
        String sql = "select * from play where play_id = ?";
        return qr.query(sql, new BeanListHandler<>(Play.class), play_id);
    }

    @Override
    public List<Play> findPlayUnderLimit(int first, int rows) throws SQLException {
        QueryRunner qr = new QueryRunner(DBUtils.getDataSource());
        String sql = "select * from play limit ? , ?";
        return qr.query(sql, new BeanListHandler<>(Play.class), first, rows);
    }

    @Override
    public int updatePlayInfo(Play play) throws SQLException {
        QueryRunner qr = new QueryRunner(DBUtils.getDataSource());
        int answer = checkPlay(qr, play);
        if (answer > 0){
            String sql = "update play set play_type_id = ?, play_lang_id = ?, play_name = ?, play_introduction = ?, play_image = ?, play_length = ?, play_ticket_price= ?,play_status = ? where play_id = ?";
            return qr.update(sql, play.getPlay_type_id(), play.getPlay_lang_id(), play.getPlay_name(), play.getPlay_introduction(), play.getPlay_image(), play.getPlay_length(), play.getPlay_ticket_price(), play.getPlay_status(), play.getPlay_id());
        }
        return answer;
    }

    @Override
    public int addPlay(Play play) throws SQLException {
        QueryRunner qr = new QueryRunner(DBUtils.getDataSource());
        int answer = checkPlayWhenAdd(qr, play);
        if(answer > 0){
            String status;
            if(play.getPlay_status() == null || play.getPlay_status() ==""){
                status = "0";
            }else{
                status = play.getPlay_status();
            }
            String sql = "insert into play( play_lang_id, play_type_id, play_name, play_introduction, play_image, play_length, play_ticket_price, play_status) values(?, ?, ?, ?, ?, ?, ?, '"+status+"')";
            return qr.update(sql, play.getPlay_lang_id(), play.getPlay_type_id(), play.getPlay_name(), play.getPlay_introduction(), play.getPlay_image(), play.getPlay_length(), play.getPlay_ticket_price());
        }
        return answer;
    }

    @Override
    public int deletePlay(int play_id) throws SQLException {
        QueryRunner qr = new QueryRunner(DBUtils.getDataSource());
        int answer = checkScheduleWhenDelete(qr, play_id);
        if (answer > 0) {
            String sql = "delete from play where play_id = ?";
            return qr.update(sql, play_id);
        }
        return answer;
    }

    public int checkPlay(QueryRunner qr, Play play) throws SQLException {
        /**
          * @Author ANGLE0
          * @Description 检查剧目是否存在
          * @Date 20:05 2019/6/9
          * @Param [qr, play]
          * @return int -80264 ：剧目不存在
          **/
        List list = qr.query("select * from play where play_id = ?", new BeanListHandler<>(Play.class), play.getPlay_id());
        if (list.size() > 0){
            return 1;
        }else{
            return -80264;
        }
    }

    public int checkScheduleWhenDelete(QueryRunner qr, int play_id) throws SQLException {
        /**
          * @Author ANGLE0
          * @Description 删除检查
          * @Date 22:34 2019/6/12
          * @Param [qr, play_id]
          * @return int -547618 : 有计划已安排了剧目
          **/
        List list = qr.query("select * from schedule where play_id = ?", new BeanListHandler<>(Play.class), play_id);
        if(list.size() > 0){
            return -547618;
        }
        return 1;
    }

    public int checkPlayWhenAdd(QueryRunner qr, Play play) throws SQLException{
        /**
          * @Author ANGLE0
          * @Description
          * @Date 15:21 2019/6/10
          * @Param [qr, play]
          * @return int -797752: 剧目类型不正确    -797763： 剧目语言不存在
          **/
        List list = (List) qr.query("select * from play_language where lang_id = ?", new BeanListHandler<>(Object.class), play.getPlay_lang_id());
        if( list.size() > 0){
            list = qr.query("select * from play_type where type_id = ?", new BeanListHandler<>(Object.class), play.getPlay_type_id());
            if(list.size() > 0){
                return 1;
            }
            return -797752;
        }
        return -797763;
    }

}

```



### 5.3service

```java
package com.w.ttms.service;

import com.w.ttms.domian.Play;

import java.sql.SQLException;
import java.util.List;

public interface PlayService {

    List<Play> searchAllPlay() throws SQLException;

    List<Play> searchPlayLimit(int page, int rows) throws SQLException;

    List<Play> findPlayUnderCondition(String name, String value) throws SQLException;

    List<Play> findPlayByID(int play_id) throws SQLException;

    int updatePlay(Play play) throws SQLException;

    int addPlay(Play play) throws SQLException ;

    int deletePlay(int play_id) throws SQLException;
}

```



### 5.4serviceImpl

```java
package com.w.ttms.service.serviceImpl;

import com.w.ttms.dao.daoImpl.playDaoImpl;
import com.w.ttms.dao.playDao;
import com.w.ttms.domian.Play;
import com.w.ttms.service.PlayService;

import java.sql.SQLException;
import java.util.List;

public class PlayServiceImpl implements PlayService {
    playDao playDao = new playDaoImpl();
    @Override
    public List<Play> searchAllPlay() throws SQLException {
        return playDao.findAllPlay();
    }

    @Override
    public List<Play> searchPlayLimit(int page, int rows) throws SQLException {
        if(page > 0){
            page = 1;
        }
        if (rows > 0){
            rows = 10;
        }
        return playDao.findPlayUnderLimit((page-1)*rows, rows);
    }

    @Override
    public List<Play> findPlayUnderCondition(String name, String value) throws SQLException {
        return playDao.searchPlayUnderCondition(name, value);
    }

    @Override
    public List<Play> findPlayByID(int play_id) throws SQLException {
        return playDao.searchPlayByID(play_id);
    }

    @Override
    public int updatePlay(Play play) throws SQLException {
        return playDao.updatePlayInfo(play);
    }

    @Override
    public int addPlay(Play play) throws SQLException {
        return playDao.addPlay(play);
    }

    @Override
    public int deletePlay(int play_id) throws SQLException {
        return playDao.deletePlay(play_id);
    }


}

```



### 5.5servlet

- 处理最初加载

```java
package com.w.ttms.web.servlet.m_play;

import com.w.ttms.domian.Play;
import com.w.ttms.service.serviceImpl.PlayServiceImpl;
import net.sf.json.JSONArray;
import net.sf.json.JSONObject;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;
import java.sql.SQLException;
import java.util.List;

/**
 * ClassName: searchAllSaleItemServlet
 * Desrciption: 查询所有用户
 * Author: ANGLE0
 * CreateDate: 2019/5/22 17:49
 * Version: V1.0
 **/
@WebServlet("/search_all_plays")
public class searchAllPlayServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

        resp.setCharacterEncoding("UTF-8");
        PrintWriter out = resp.getWriter();
        //设置头属性
        resp.setHeader("content-type", "text/html;charset=UTF-8");
        //用于输出数据

        String sort = req.getParameter("sort");
        String order = req.getParameter("order");

        String page = req.getParameter("page");
        String rows  =req.getParameter("rows");

        System.out.println("page="+page+"rows="+rows);

        PlayServiceImpl playService = new PlayServiceImpl();

        List<Play> playList = null;
        List<Play> allPlay = null;
        try {

            playList = playService.searchPlayLimit(Integer.parseInt(page),Integer.parseInt(rows));

            allPlay = playService.searchAllPlay();

        } catch (SQLException e) {
            e.printStackTrace();
        }
        if(playList != null && allPlay != null ){
            JSONArray list = JSONArray.fromObject(playList);

            JSONObject jsonObject = new JSONObject();

            jsonObject.put("total", allPlay.size());
            jsonObject.put("rows", list.toString());

            out.print(jsonObject.toString());
        }else{
            out.print(false);
        }
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        super.doGet(req, resp);
    }
}

```

- 处理修改

```java
package com.w.ttms.web.servlet.m_play;

import com.w.ttms.domian.Play;
import com.w.ttms.service.serviceImpl.PlayServiceImpl;
import net.sf.ezmorph.object.DateMorpher;
import net.sf.json.JSONObject;
import net.sf.json.util.JSONUtils;
import org.apache.commons.lang.StringUtils;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;
import java.sql.SQLException;

/**
 * ClassName: changeTicketServlet
 * Desrciption: TODO
 * Author: ANGLE0
 * CreateDate: 2019/6/3 22:32
 * Version: V1.0
 **/
@WebServlet("/change_play")
public class changePlayServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

        req.setCharacterEncoding("utf-8");
        resp.setCharacterEncoding("UTF-8");
        PrintWriter out = resp.getWriter();

        String type = req.getParameter("type");
        String row = req.getParameter("row");

        System.out.println("type=" + type + "前台发来要改变的数据为：" + row.toString());

        Play play = null;
        int answer = 0;
        int answer1 = 0;
        int answer3 = 0;
        PlayServiceImpl playService = new PlayServiceImpl();

        try {
            if ("update".equals(type) || "add".equals(type)) {
                JSONObject jsonObject = JSONObject.fromObject(row);
                play = (Play) JSONObject.toBean(jsonObject, Play.class);

                if ("update".equals(type)) {
                    System.out.println("play我们即将添加信息：" + play.toString());
                    answer = playService.updatePlay(play);
                } else if ("add".equals(type)) {
                    answer1 = playService.addPlay(play);
                }
            } else if("delete".equals(type)){
                row = StringUtils.substringBeforeLast(row, "\"");
                row = row.substring(1);

//                System.out.println("截取的字符串为"+ StringUtils.substringBeforeLast(row, "\""));
                answer3 = playService.deletePlay(Integer.parseInt( row));
                System.out.println(answer1);
            }else{
                return;
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }

        if (answer > 0) {
            out.print("修改剧目成功");
        } else if (answer1 > 0) {
            out.print("添加剧目成功");
        } else if(answer3 == -547618){
            out.print("操作失败,有计划已经安排了此剧目");
        }else if(answer3 > 0){
            out.print("删除剧目成功");
        }else{
            out.print("操作失败");
        }
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        super.doPost(req, resp);
    }
}
```

### 5.6util（DBUtils）

```java
package com.w.ttms.util;

import com.alibaba.druid.pool.DruidDataSourceFactory;

import javax.sql.DataSource;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.sql.*;
import java.util.Properties;

public class DBUtils {

    private static DataSource dataSource = null;

    static{
        String path = DBUtils.class.getClassLoader().getResource("dbconfig.properties").getPath();
        FileInputStream in = null;
        Properties p = new Properties();
        try {
            in = new FileInputStream(path);
            p.load(in);
            dataSource = DruidDataSourceFactory.createDataSource(p);
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }catch (Exception e) {
            e.printStackTrace();
        }
    }



    public static DataSource getDataSource(){
        /**
          * @Author ANGLE0
          * @Description 获取数据源
          * @Date 8:37 2019/5/23
          * @Param []
          * @return javax.sql.DataSource
          **/
        return dataSource;
    }

    public static Connection getConnection(){
        /**
          * @Author ANGLE0
          * @Description 获取连接
          * @Date 8:38 2019/5/23
          * @Param []
          * @return java.sql.Connection
          **/

        try {
            return dataSource.getConnection();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

    public static void release(Connection connection, PreparedStatement preparedStatement, ResultSet resultSet){
        /**
          * @Author ANGLE0
          * @Description 关闭数据源
          * @Date 8:38 2019/5/23
          * @Param [connection, preparedStatement, resultSet]
          * @return void
          **/
        if (resultSet != null) {
            try {
                resultSet.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if (preparedStatement != null) {
            try {
                preparedStatement.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if (connection != null) {
            try {
                connection.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
}
```

