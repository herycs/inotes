# easyui弹窗设置

## 1.基本书写

​	窗体有验证框，回车切换输入焦点

```javascript
//窗体设置
$(function(){
    $('#dlg').dialog({
        title: "用户登陆",      //对话框标题
        width: 400,            //宽度
        height: 300,           //高度
        model: true,           //模式窗口
        buttons: '#login_btn',         //绑定按钮
        iconCls: 'icon-open',  //设置窗口图标
        cls: 'c6',             //使用颜色样式c1
    });
})
//验证设置
$('#u_name').validatebox({
    required: true,
    validType: 'length[5, 10]'
})
//密码验证
$('#u_password').validatebox({
    required: true,
    validType: 'length[5, 10]'
})
//设定button效果
//------
// $('button').linkbutton({
//     width: 60,
//     height: 30,
// }).click(function(){
//     alert("登陆中...")
// })
//------
$('#login_button').linkbutton({
    width : 60,
    height : 30,
    iconCls : 'icon-login',
    onClick :  function() {
        if(!$('#username').validatebox('isValid')) {
            $('#username').focus();
        } else if(!$('password').validatebox('isValid')) {
            $('#password').focus();
        } else {
            //加载进度条
            $.messager.progress({
                text: '正在登陆...',
            });
            $.ajax({
                //目标地址
                url : 'homo.html',
                type : 'get',
                data : {
                    username : $('#username').val(),
                    password : $('#password').val(),
                },
                beforeSend : function(){
                    $.messager.progress({
                        text : '正在登陆中...',
                    });
                },
                sucess : function (data){
                    $.messager.progress('close');
                    if(data == 0){
                        $.messager.alter('警告','用户名或密码错误，请重新输入', 'warning', function(){
                            $('#password').select();
                        });
                    } else {
                        location.href = 'home.html';
                    }
                }
            })
        }
    }
});
$('#u_password').passwordbox({
    inputEvents: $.extend({}, $.fn.passwordbox.defaults.inputEvents, {
        keypress: function (e) {
            var char = String.fromCharCode(e.which);
            $('#viewer').html(char).fadeIn(200, function () {
                $(this).fadeOut();
            });
        }
    })
})
```

