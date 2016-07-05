[toc]

## CSS性能与优化

本质上，引擎需要解析的CSS规则越少，性能越好。MDN上将CSS选择符归类成四个主要类别，如下所示，性能依次降低：ID 规则、Class 规则、标签规则、通用规则。

**后代选择符最烂**。不仅性能低下而且代码很脆弱，**html代码和css代码严重耦合**，html代码结构发生变化时，CSS也得修改，这是多么糟糕，特别是在大公司里，写html和css的往往不是同一个人。

    // 烂透了
    html div tr td {..}

**避免链式（交集）选择符**。这和过度约束的情况类似，更明智的做法是简单的创建一个新的CSS类选择符。

    // 糟糕
    .menu.left.icon {..}

    // 好的
    .menu-left-icon {..}

**避免过度约束**。一条普遍规则，不要添加不必要的约束。

    // 糟糕
    ul#someid {..}
    .menu#otherid{..}

    // 好的
    #someid {..}
    #otherid {..}

**避免不必要的命名空间**

    // 糟糕
    .someclass table tr.otherclass td.somerule {..}

    //好的
    .someclass .otherclass td.somerule {..}

**尽可能精简规则**

    // 糟糕
    .someclass {
     color: red;
     background: blue;
     height: 150px;
     width: 150px;
     font-size: 16px;
    }

    .otherclass {
     color: red;
     background: blue;
     height: 150px;
     width: 150px;
     font-size: 8px;
    }

    // 好的
    .someclass, .otherclass {
     color: red;
     background: blue;
     height: 150px;
     width: 150px;
    }

    .someclass {
     font-size: 16px;
    }

    .otherclass {
     font-size: 8px;
    }

### 声明顺序

虽然有一些排列CSS属性顺序常见的方式，下面是我遵循的一种流行方式。

    .someclass {
     /* Positioning */
     /* Display & Box Model */
     /* Background and typography styles */
     /* Transitions */
     /* Other */
    }

参见：http://css-tricks.com/new-poll-how-order-css-properties/

### 待研究

https://developer.mozilla.org/en-US/docs/Web/Guide/CSS/Writing_efficient_CSS

https://developers.google.com/speed/pagespeed/
https://developers.google.com/speed/docs/best-practices/rendering#UseEfficientCSSSelectors

### 已参考

http://blog.jobbole.com/55067/

