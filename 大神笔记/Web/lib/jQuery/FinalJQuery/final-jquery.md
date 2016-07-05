[toc]

## 技巧

在创建元素时，如何**使用对象字面量（literal）来定义属性**：

	var e = $("", { href: "#", class: "a-class another-class", title: "..." });

如何在一段时间之后自动隐藏或关闭元素（支持1.4版本）：

    //这是1.3.2中我们使用setTimeout来实现的方式
    setTimeout(function() {
      $('.mydiv').hide('blind', {}, 500)
    }, 5000);
    //而这是在1.4中可以使用delay()这一功能来实现的方式（这很像是休眠）
    $(".mydiv").delay(5000).hide('blind', {}, 500);

在jQuery中如何使用.siblings()来选择同辈元素

    // 不这样做
    $('#nav li').click(function(){
        $('#nav li').removeClass('active');
        $(this).addClass('active');
    });
    //替代做法是
    $('#nav li').click(function(){
        $(this).addClass('active').siblings().removeClass('active');
    });

如何把一个元素放在屏幕的中心位置：

    jQuery.fn.center = function () {
        this.css('position','absolute');
        this.css('top', ( $(window).height() - this.height() ) / +$(window).scrollTop() + 'px');
        this.css('left', ( $(window).width() - this.width() ) / 2+$(window).scrollLeft() + 'px');
        return this;
    }
    //这样来使用上面的函数：
    $(element).center();

如何使用jQuery来检测右键和左键的鼠标单击两种情况：

    $("#someelement").live('click', function(e) {
        if( (!$.browser.msie && e.button == 0) || ($.browser.msie && e.button == 1) ) {
            alert("Left Mouse Button Clicked");
        } else if(e.button == 2) {
            alert("Right Mouse Button Clicked");
        }
    });

如何使用多个属性来进行过滤

    //在使用许多相类似的有着不同类型的input元素时，
    //这种基于精确度的方法很有用
    var elements = $('#someid input[type=sometype][value=somevalue]').get();

如何隐藏一个包含了某个值文本的元素：

    $("p.value:contains('thetextvalue')").hide();

如何检查某个元素是否存在

    if ($('#someDiv').length) {
    	//万岁！！！它存在……
    }

如何自动滚动到页面中的某区域

    jQuery.fn.autoscroll = function(selector) {
        $('html,body').animate(
            {scrollTop: $(selector).offset().top},
            500
        };
    }
    //然后像这样来滚动到你希望去到的class/area上。
    $('.area_name').autoscroll();

如何禁用右键单击上下文菜单：

    $(document).bind('contextmenu',function(e){
        return false;
    });

如何定义一个定制的选择器

    $.expr[':'].mycustomselector = function(element, index, meta, stack){
    // element- 一个DOM元素
    // index – 栈中的当前循环索引
    // meta – 有关选择器的元数据
    // stack – 要循环的所有元素的栈
    // 如果包含了当前元素就返回true
    // 如果不包含当前元素就返回false };
    // 定制选择器的用法：
    $('.someClasses:test').doSomething();

如何设置IE特有的功能：

    if ($.browser.msie) {
    // Internet Explorer就是个虐待狂
    }

如何检测各种浏览器：

    检测Safari (if( $.browser.safari)),
    检测IE6及之后版本 (if ($.browser.msie && $.browser.version > 6 )),
    检测IE6及之前版本 (if ($.browser.msie && $.browser.version <= 6 )),
    检测FireFox 2及之后版本 (if ($.browser.mozilla && $.browser.version >= '1.8' ))

如何使用jQuery来预加载图像：

    jQuery.preloadImages = function() {
        for(var i = 0; i < arguments.length; i++) {
            $("<img />").attr('src', arguments[i]);
        }
    };
    //用法
    $.preloadImages('image1.gif', '/path/to/image2.png', 'some/image3.jpg');

如何验证某个元素是否为空：

    if ($('#keks').html() == null) {
    //什么都没有找到;
    }

如何创建嵌套的过滤器：

    //允许你减少集合中的匹配元素的过滤器，
    //只剩下那些与给定的选择器匹配的部分。在这种情况下，
    //查询删除了任何没（:not）有（:has）
    //包含class为“selected”（.selected）的子节点。
    .filter(":not(:has(.selected))")

任何使用has()来检查某个元素是否包含某个类或是元素：

	$("input").has(".email").addClass("email_icon");

如何使用jQuery来切换样式表

    // 找出你希望切换的媒体类型（media-type），然后把href设置成新的样式表。
    $('link[media='screen']').attr('href', 'Alternative.css');

如何限制“Text-Area”域中的字符的个数：

    jQuery.fn.maxLength = function(max){
        this.each(function(){
            var type = this.tagName.toLowerCase();
            var inputType = this.type? this.type.toLowerCase() : null;
            if(type == "input" && inputType == "text" || inputType == "password"){
                //Apply the standard maxLength
                this.maxLength = max;
            }
            else if(type == "textarea"){
                this.onkeypress = function(e){
                    var ob = e || event;
                    var keyCode = ob.keyCode;
                    var hasSelection = document.selection? document.selection.createRange().text.length > 0 : this.selectionStart != this.selectionEnd;
                    return !(this.value.length >= max && (keyCode > 50 || keyCode == 32 || keyCode == 0 || keyCode == 13) && !ob.ctrlKey && !ob.altKey && !hasSelection);
                };
                this.onkeyup = function(){
                    if(this.value.length > max){
                        this.value = this.value.substring(0,max);
                    }
                };
            }
        });
    };
    //用法
    $('#mytextarea').maxLength(500);

链式日志记录：

    // 允许链式日志记录
    // 用法：
    $('#someDiv').hide().log('div hidden').addClass('someClass');
    jQuery.log = jQuery.fn.log = function (msg) {
        if (console){
            console.log("%s: %o", msg, this);
        }
        return this;
    };

如何使用jQuery来解析XML（基本的例子）：

    function parseXml(xml) {
        //找到每个Tutorial并打印出author
        $(xml).find("Tutorial").each(function() {
            $("#output").append($(this).attr("author") + "");
        });
    }