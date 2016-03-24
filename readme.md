# Css Modulization

参考以下几个工具的实现：
1. NEJ
2. ShadowDom for Scoping
3. ReactJS Inline Style
4. [CssModules](https://github.com/css-modules)

## 背景：

&emsp;&emsp;前端领域， Web组件化思想已经较普及，无论各种框架Regular/ReactJs， 或者各种开发工具如模块化工具seajs，都在推进着组件化的发展；目前，随着Regular/ReactJs等框架的出现，让我们可以很容易的实现一个控件；然而，我们离真正完整的组件化还有距离，很多人从不同的方向角度推进着前端组件化的发展，如Flux架构思想的提出以及其各种的实现；在这么多高大上的新东西中，都是围绕程序代码为核心的，提到组件化，很少有人会联想到Css样式；在我看来，作为组件化的一部分，模板+逻辑+样式每一部分都不应当落下；

## css模块化之路

### 原生css
&emsp;&emsp;写过css样式的都深有感触，原生Css最大的问题，就是`Everything in CSS is a global`，如果没有一套规则， 我们很难保证样式之间的冲突；通常样式表是与工程、页面相关联的，粒度并没有达到组件的程度，对于通用的组件， 我们就要统一维护在一个文件中， 让所有页面引用，这样即使一个页面没有使用任何通用的组件， 也加载了全局的样式，造成了浪费；虽然可以使用inline样式保证浪费的问题，但是在没组件化之前，满页面的inline样式让本来就不怎么整洁的html看起来更乱，造成了难维护的问题；

### [NEC](http://nec.netease.com/)
&emsp;&emsp;nec提供了一套很好的命名规范，最大程度保证了样式命名的合理性， 防止出现样式冲突；从样式命名上达到了组件化的目的，实现了css样式的组件化，它与js除了命名外，完全无关联，这样你可以在你自己的css文件中，重新实现一套控件的样式；通常情况下，我们使用别人控件的时候，需要分别引入对应的JS代码和Css样式，如果使用的控件多，需要引入很多css，也比较难维护；
```
/* 这是某个模块 */
.m-nav{}/* 模块容器 */
.m-nav li,.m-nav a{}/* 先共性  优化组合 */
.m-nav li{}/* 后个性  语义化标签选择器 */
.m-nav a{}/* 后个性中的共性 按结构顺序 */
.m-nav a.a1{}/* 后个性中的个性 */
.m-nav a.a2{}/* 后个性中的个性 */
```

### MCSS/SASS
&emsp;&emsp;如果没用过，可以参考[官网](https://github.com/leeluolee/mcss);这里只说下存在的问题；虽然这些预处理工具可以让我们更简洁的书写样式，但是本质上它还是单纯的样式模块化，并没有解决一个完整控件的组件化；

## 组件化样式参考
![理想的组件组织方式](http://haitao.nos.netease.com/f6798300b49f46bfb0580caebf12484d.jpg)
&emsp;&emsp;上图是我们理想的组织一个组件的方式，包括组件的模板，组件的逻辑处理代码以及组件的样式；在使用过程中，我们只需要引入js，直接使用组件，无需关注它的模板和它的样式就可以展现完整的组件功能；一些工具已经实现了这样的功能，下面分别介绍下；

### NEJ
&emsp;&emsp;在我们每次写一个NEJ的组件时，都会在最开始指定对应的模板和样式， 如下：这里有一组方法`_$pushCSSText`和`_$dumpCSSText`， 前者将样式文件解析后存放在cache中，后者在head尾部生成一个style标签，将cache中的所有样式join后放入其中；
![](http://haitao.nos.netease.com/73f663df83d14e6aa5fb2d56a9ed4a16.jpg)
&emsp;&emsp;首先，样式文件是这样声明的,每个样式前都加了一个`.#<uispace>`
![](http://haitao.nos.netease.com/7fddf0f3adf84a358559e1982b93a9a5.jpg)

```javascript
_p._$pushCSSText = (function(){
    var _reg = /#<(.*?)>/g,
        _seed = +new Date;
    return function(_css,_data){
        if (!_cspol){
            _cspol = [];
        }
        var _class = 'auto-'+_u._$uniqueID(),
            _dmap = _u._$merge({uispace:_class},_data);
        _cspol.push(
            _css.replace(_reg,function($1,$2){
                return _dmap[$2]||$1;
            })
        );
        return _class;
    };
  })();

```
&emsp;&emsp;这段代码会匹配`#<xxx>`这段字符，将其替换成一个uniqId，放入cache中，最后返回这个uniqId，返回的目的是在组件的最外层元素中加上这个class名，这样就可以与样式关联起来了；
再调用_$dumpCssText方法的时候，统一将cache中的样式一次性放入到页面中， 如下图：
![](http://haitao.nos.netease.com/e424212ca694486ab864ab3ebfab42c2.jpg)


### ShadowDom for Scoping
&emsp;&emsp;ShadowDOM解决了DOM树的封装问题；通过下面的代码，我们可以通过一个shadow树替换掉了我们 .widget div 的文本内容，textContent可以是一段模板，并且包含样式，它提供了一个域空间，这里命名的样式仅在这里才有效，内部的逻辑包括样式对于使用者是不可见的；
```
<div class="widget">Hello, world!</div>  
<script>  
    var host = document.querySelector('.widget');
    var root = host.createShadowRoot();
    root.textContent = '我在你的 div 里！';
</script> 
```
&emsp;&emsp;由于它的兼容性比较差，所以目前基本没有很多相关的应用出现：
![](http://haitao.nos.netease.com/4f05b025d0aa4c4e8d79955bb8d5bfdb.jpg)

### ReactJS Inline Style
&emsp;&emsp;InlineStyle目前在React中存在比较大的争议，按照传统的方式，我们会反感看到写在模板中的Inline样式，但是究其原因应该是两方面：1.无法复用；2. 模板混乱； 使用react等框架解决了模板复用的问题，并且提供的`style`属性可以赋值变量，这样样式不会太长，模板就不会很乱；所以在ReactJS中使用InlineStyle很常见, InlineStyle可以更进一步的组件化我们的控件；
```
var MyDiv = React.createClass({
  render: function() {
    var style = {
      color: 'white',
      fontSize: 200
    };

    return <div style={style}> Have a good and productive day! </div>;
  }
});
```
&emsp;&emsp;上面说了使用InlineStyle目前是有争议的，所以React中css样式的解决方案也有很多：他们也基本使用了这里介绍的几种方式，生成uniqId，shadowDom等；
![](http://haitao.nos.netease.com/865742c5e8514594a48a732267e84f44.jpg)

### CSS Modules
&emsp;&emsp;CssModules同样是生成uniqId的方式， 与Nej不同的是Nej是在运行时生成的，而CssModules是一个预处理工具，结合gulp等工具，可以实现自动化生成解析对应的css到指定文件中；

## 参考文章：
1. [Shadow DOM 201](http://www.html5rocks.com/zh/tutorials/webcomponents/shadowdom-201/)
2. [The Debate Around “Do We Even Need CSS Anymore?”](https://css-tricks.com/the-debate-around-do-we-even-need-css-anymore/)
3. [CssModules](https://github.com/css-modules/css-modules)
4. [Comparison of CSS in JS Libraries for React](https://github.com/FormidableLabs/radium/blob/master/docs/comparison/README.md)
5. [React.js inline style best practices](http://stackoverflow.com/questions/26882177/react-js-inline-style-best-practices)
6. [Why You Shouldn’t Style React Components With JavaScript](http://jamesknelson.com/why-you-shouldnt-style-with-javascript/)
7. [2015前端组件化框架之路](https://github.com/xufei/blog/issues/19)
8. [ShadowDom基础](http://www.ituring.com.cn/article/177461)