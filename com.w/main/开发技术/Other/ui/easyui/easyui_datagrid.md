# datagrid

## 1.pagination

- 作用：用于导航分页数据

- 具体属性：
  - layout属性,设置分页组件的布局
  
    - 核心选项：['first', 'prev', 'next', 'last', 'manual']
  
    - 其它可放入的选项：'sep'，'info'，'list',  'refresh'
  
      - 解释：list和refresh在layout中没被列出，所以不显示
  
  
  | 属性           | 值类型  | 描述   | 默认值 |
  | -------------- | ------- | ------ | ------ |
  | total          | number  | 总记录 | 1      |
  | showPageLIst   | boolean |        |        |
  | showRefresh    | boolean |        |        |
  | showPageInfo   | boolean |        |        |
  | pageList       | array   |        |        |
  | pageSize       | number  |        |        |
  | pageNumber     | number  |        |        |
  | layout         | array   |        |        |
  | links          | number  |        |        |
  | beforePageText | string  |        |        |
  | afterPageText  | string  |        |        |
  | displayMsg     | string  |        |        |
  | loading        | boolean |        |        |
  | buttons        | array   |        |        |
  
  - button属性：
    - iconCls：背景图片
    - handler：点击触发函数
    
  - 选择器对象：
    - 本质DOM：元素
    - 使用：在button或者其他部分绑定这个对象即可
  
- 方法：动态交互及刷新

  | 方法名  |  参数   | 描述                                            |
  | :-----: | :-----: | :---------------------------------------------- |
  | options |  none   | 返回参数对象                                    |
  | select  |  page   | 选择新的数据页                                  |
  | refresh | options | 刷新并显示分页组件信息,不带参数时只刷新分页信息 |
  | loading |  none   | 将刷新按钮显示为正在加载                        |
  | loaded  |  none   | 将刷新按钮展示为已经加载完毕                    |

  ```javascript
  $('#button').searchbox({
          searcher:function (value) {
              if(!parseInt(value)) return;//搜索框中的值不大于0，返回
              $('#p').pagination('select',parseInt(value));
          }
      });
      $('#dg').pagination('refresh',{
          total: 180,
          pageNumber: 3,
          displayMsg: '从{from}到{to},共{total}条记录'
      });
  ```

  

- 事件：操作时触发

  （其中onBeforeRefresh,onRefresh针对单击刷新按钮）

  | 事件             | 事件参数            | 描述                              |
  | ---------------- | ------------------- | --------------------------------- |
  | onSelectPage     | pageNumber,pageSize | 选择新数据页时触发                |
  | onBeforeRefresh  | pageNumber,pageSize | 刷新数据之前触发，返回false可取消 |
  | onRefresh        | pageNumber,pageSize | 刷新数据之后触发                  |
  | onChangePageSize | pageSize            | 更改页面数据记录大小时触发        |

  ```javascript
  onSelectPage:function (pageNumber, pageSize) {
              $(this).pagination('loading');
              alert('当前分页数：'+pageNumber+',页面记录数据大小：'+pageSize);
              $(this).pagination('loaded');
          },
  ```

  

## 2.datagrid表格列和列属性

- 基础：标签方式创建数据表格
- js方式创建数据表格
  - array：数组
  
    > data:[
    >
    > {id = ...},
    >
    > ...
    >
    > {id=...}
    >
    > ],
  
  - object：data加载表格的基本属性，实际开发中使用url
  
    > data:{total: totalNumber, rows:[
    >
    > {id: ...},
    >
    > ..
    >
    > {id:...}
    >
    > ]}
- 列属性：
  - 展示信息：
    - 设置data
    - 设置显示的数据列的属性
    
  - column：使用时使用两个[[]]，目的为后期提供多层表头的操作
  
  - 列宽，列对齐，列隐藏，列排序
  
    - 数据右对齐：align:'right'
  
    - 列名居中： halign:'center'
  
    - 列隐藏：hidden:'true'
  
    - 列排序：sortable:true
  
      > 示例
  
      ```javascript
      <th data-options="field:'username',width:150,align:'right',halign:'center'">密码</th>
      ```
  
      > 列隐藏和列排序都不能对横向合并了了的单元格操作，（即：只有title，没有field的列）
  
  - 格式化单元格内容
  
    - formatter
      - 参数：
        - 字段名：value
        - 行记录数据：row
        - 行索引号：index
    - 特点：可显示样式和内容
  
  - 设置单元格样式
  
    - styler
      - 参数：
        - 字段名：value
        - 行记录数：row
        - 行索引号：index
    - 特点：只返回样式不返回内容
  
  - 列编辑器

## 3.加载数据及分页

- 加载数据：

  - 加载json数据，js的url中添加为文件
  - 远程数据加载，后台需要返回json类型数据
  - 有关字段：method , queryParams, loadMsg

- 分页

  - 数据分页：datagrid中pagination 属性设置为true
    - 请求服务器时会同时向服务器发送两个数据:
      - page
      - rows
  - 分页相关属性

- 数据排序

  - 本地数据排序

    - 字段属性中sortable:true

  - 一个数据页面内排序：

    - remoteSort: false

  - 远程数据排序：

    - remoteSort: true

  - 注意：本地数据和远程数据排序并非完全割裂

    - 远程排序: remoteSort： true

      服务器根据收到的sort 和order处理

    - 页内排序：remoteSort设置为flase

    - 可以设置默认排序

  - 初始化排序：

    - sortName:指定排列字段
    - sortOrder:排序方式

  - 多列排序

    - sortable:true

    - 要求多列排序时，后一次排序建立在第一次排序上

      multiSort:true

    - 服务器端接收到的数据:

      sort中没有“，”则是单列排序，

      sort中有“，”则是多列排序，此时需要处理排序的设定

    - 设置默认排序：
      - sortName: 排序字段
      - sortOrder: 排序方式,asc, desc

- 数据加载

  - loder属性
    - 本质：ajax
    - 返回：false时，取消加载操作
  - loadFiter属性
    - 用于返回过滤后的数据，对动态的加载数据本地数据均有效

## 3.datagrid外观，编辑器和视图属性

- 属性

  ​	

  |                        |      |
  | ---------------------- | ---- |
  | 选择行                 |      |
  | 自动调整列宽           |      |
  | 冻结列                 |      |
  | 超过内容是否换行       |      |
  | 行列号                 |      |
  | 滚动条宽度             |      |
  | 标题行                 |      |
  | 行脚                   |      |
  | 多选Ctrl+鼠标单击      |      |
  | 是否只允许选择一条语句 |      |
  | 点击复选框时是否选择行 |      |
  | 顶部工具栏             |      |
  | datagrid视图           |      |
  |                        |      |
  |                        |      |
  |                        |      |
  |                        |      |
  |                        |      |
  |                        |      |
  |                        |      |
  |                        |      |
  |                        |      |
  |                        |      |

- 属性：styler
  - 参数：index,row
  - 使用范围：列
  
- 属性：rowStyle
  
  - 使用范围：datageid,对行有效
  
- 相关应用：

  - 改变行脚样式
  - 改变表格数据
  - 卡片式数据显示效果

> 编辑数据时至少保证当前行至少有一列指定了编辑器

## 4.datagrid方法

- 常用传递参数
  - index行索引，每页的索引从0开始
  - row:行数据
  - field: 列名称
- 选择数据行返回数据
- 数据记录编辑
  - 对数据的增删改查可以调用直接方法
    - appendRow
    - insertRow
    - updateRow
    - deleteRow
  - 表格中手工验证了或者修改某行数据时，必须开始使用编辑方法，且数据表格中至少有一列指定了编辑器，否则无法进入编辑状态
  - 以上方法基于客户端，若要修改数据库需要添加表格事件
- 数据加载刷新事件

## 5.datagrid事件

- 数据加载事件
  - 合并同类项事件
  - 生成动态提示框
- 选择行，排序及右键菜单事件
  - check事件仅对checkbox为true生成复选列有效，对设定编辑器为checkbox的列无效
  - onHeaderContextMenu：鼠标右击表头时触发
    - 参数：e,field
  - onRowContextMenu：鼠标右击行时触发
    - 参数：e,index,row
- 单击，双击，编辑事件
  - 编辑后，换行双击时应当对上一个编辑的行验证
- 增加数据，修改数据结束时会触发onAfterEdit事件

## 6.一些代码片

- 设置按钮的可用不可用状态

  ```javascript
  var obj={
      
      enable:function () {
          $('#s1').linkbutton('disable');
          $('#s2').linkbutton('disable');
          $('#s3').linkbutton('disable');
          $('#s4').linkbutton('enable');
          $('#s5').linkbutton('enable');
      },
      disable: function(){
          $('#s1').linkbutton('disable');
          $('#s2').linkbutton('disable');
          $('#s3').linkbutton('disable');
          $('#s4').linkbutton('enable');
          $('#s5').linkbutton('enable');
      }
  }
  ```

- 设置排序

  > 特殊对本地数据排序：表属性设置为 remoteSort: false
  >
  > 对远程排序：表属性设置为 remoteSort:true

  ```javascript
  <th data-options="field:'u_id',width:80,
  	sortable:true,//当前列可排序
      sorter:function(a,b){//排序规则
          return (parseFloat(a)>parseFloat(b)?1:-1);
      }">ID</th>
  ```

- 设置样式

  - 参数：value,row,index具体传入几个自己选择

  - 对单元格，内容都修改

    - formatter

    ```javascript
    formatter:function(val, row){
    	if(val == 0){
    		return '<span style=color:red;>'+'未激活'+'</span>';
    	}else{
    		return '<span style=color:green;>'+'已激活'+'</span>';
    }
    },
    ```

  - 对单元格样式修改，对内容不修改

    - styler

      ```javascript
      styler:function(value, row, index){
      	if(value == ''){
      		return 'background-color: #acdefa; color: red;';
      	}
      },
      ```

  - 设置百分比（体现销售额）

    ```javascript
    formatter:function(val){
        if(val){
            var s = '<div style="width: 100%; background: blue;">'+
            '<div style="width: '+parseFloat(val)+'%;background: red;color:white">'+ val+
            '</div>'+
            '</div>';
            return s;
        }else{
       		return '';
        }
    }
    ```

- 设置编辑器

  - 设置每一列的编辑器

    ```javascript
    editor:{
        type:'numberbox',
        options:{precision:2}//保留2位小数
    }">个人描述</th>
    
    editor:{
        type:'checkbox',
        options:{on:'1',off:'0'}//编辑时提供单选框
    }">账号状态</th>
    ```

  - 具体的编辑时输入类型可自选，easyui提供的大多数的都支持

  - js重写编辑器，修改编辑器默认值时需要使用

    ```javascript
    editors:datatimebox{{
        init:function (container, options) {
        var input = $('<'+'input'+'type="text">').appendTo(container);
        options.editable = false;//限制不可手动输入
        input.datatimebox(options);
        return input;
        },
        getValue:function (target) {
    
        },
        setValue:function (target, value) {
    
        },
        resize:function (target, width) {
    
        },
        destory:function (target) {
    
        }
    }}
    ```

    

- 创建工具栏并将工具栏拉入datagrid

  - 创建工具栏

  - 1.

    ```html
    <div id="toolb">
        <button id="s1"></button>
        <button id="s2"></button>
        <input id="s3"></input>
        <button id="s4"></button>
        <select id="s5"></select>
    </div>
    ```

  - 2.

    ```html
    
    <div id="toolb" style="height:auto">
        <a href="javascript:void(0)" class="easyui-linkbutton" data-options="iconCls:'icon-add',plain:true" onclick="append()">Append</a>
        <a href="javascript:void(0)" class="easyui-linkbutton" data-options="iconCls:'icon-remove',plain:true" onclick="removeit()">Remove</a>
        <a href="javascript:void(0)" class="easyui-linkbutton" data-options="iconCls:'icon-save',plain:true" onclick="accept()">Accept</a>
        <a href="javascript:void(0)" class="easyui-linkbutton" data-options="iconCls:'icon-undo',plain:true" onclick="reject()">Reject</a>
        <a class="easyui-searchbox" onclick="getChanges()" prompt="用户名"></a>
    </div>
    ```

    

  - 美化工具栏

    ```javascript
    $('#s1').linkbutton({
            width: 80,
            iconCls: 'icon-save',
            text:'保存数据',
            plain:true,
        });
    $('#s3').databox({
            width: 100
        });
    $('#s5').combobox({
            width: 180,
        	url: 'XXXX',
        	mode:'remote',
        	queryParams{
        		field:'area',
    		}
            valueField:'area',
            textField:'area',
        })
    ```

  - 拉入datagrid

    ```html
    <table id="dg" class="easyui-datagrid" title="Row Editing in DataGrid" style="width:700px;height:400px;"
    data-options="toolbar: '#toolb'">
    </table>
    ```

- 右键菜单

  ```javascript
  $(function(){
      $('#dg').datagrid({
          onHeaderContextMenu: function(e, field){
              e.preventDefault();
              if (!cmenu){
                  createColumnMenu();
              }
              cmenu.menu('show', {
                  left:e.pageX,
                  top:e.pageY
              });
          }
      });
  });
  var cmenu;
  function createColumnMenu(){
      cmenu = $('<div/>').appendTo('body');
      cmenu.menu({
          onClick: function(item){
              if (item.iconCls == 'icon-ok'){
                  $('#dg').datagrid('hideColumn', item.name);
                  cmenu.menu('setIcon', {
                      target: item.target,
                      iconCls: 'icon-empty'
                  });
              } else {
                  $('#dg').datagrid('showColumn', item.name);
                  cmenu.menu('setIcon', {
                      target: item.target,
                      iconCls: 'icon-ok'
                  });
              }
          }
      });
      var fields = $('#dg').datagrid('getColumnFields');
      for(var i=0; i<fields.length; i++){
          var field = fields[i];
          var col = $('#dg').datagrid('getColumnOption', field);
          cmenu.menu('appendItem', {
              text: col.title,
              name: field,
              iconCls: 'icon-ok'
          });
      }
  }
  ```

- 获取选中的值

  ```javascript
  function getSelected(){
      var row = $('#dg').datagrid('getSelected');
      if (row){
     alert('Item ID:'+row.u_id+"\nPrice:"+row.u_role);
      }
  }
  function getSelections(){
      var ids = [];
      var rows = $('#dg').datagrid('getSelections');
      for(var i=0; i<rows.length; i++){
     		ids.push(rows[i].u_id);
      }
      alert(ids.join('\n'));
  }
  ```

- 加载无数据显示为设置的默认提示

  ```javascript
  onLoadSuccess: function (data) {
    if (data.total == 0) {
      var body = $(this).data().datagrid.dc.body2;
      body.find('table tbody').append('<tr><td width="' + body.width() + '" style="height: 35px; text-align: center;"><h1>暂无数据</h1></td></tr>');
    }
  }
  ```

- 单击编辑行

  ```javascript
  onClickRow: function (index){   //单击选中
          if (editIndex != index) {
              if (endEditing()) {
                  $('#m_mans').datagrid('selectRow', index).datagrid('beginEdit', index);
                  editIndex = index;
              } else {
                  $('#m_mans').datagrid('selectRow', editIndex);
              }
          }
      }
  ```

- 双击选中列

  ```javascript
  onDblClickCell:function (index, field, value) {
          if(obj.editIndex != index){
              //选择了别的行
              if(obj.endEditing()){
                  $(this).datagrid('beginEdit', index);
                  var ed = $(this).datagrid('getEditor', {index: index, field: field});
                  if(ed != null){
                      $(ed.target).focus();
                      obj.editIndex = index;
                  }
              }else{
                  $(this).datagrid('unselectRow', index);
              }
          }
  
      }
  ```

  

## 7.完整示例

- 创建数据表格

  ```javascript
      title: '用户管理',            //标题
      iconCls: 'icon-mans',       //图标
      width: 1303,                //表格宽度
      height: 586,                //表格高度
      url: 'find_all_users',      //表格数据源
      method: 'get',              //表格数据请求方式
      pagination:true,            //表格分页信息
      align: center,              //列数据对齐方式
      halign: center,             //列标题对齐方式
  ```

  