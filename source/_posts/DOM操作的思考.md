---
title: DOM操作的性能问题
date: 2016-08-01 23:24:57
tags: [JavaScript,DOM]
---

> 在做项目的优化时,添加下拉菜单子项部分需要操作DOM,先用的是createElement创建后逐个添加,后面想到看到过DOM多次操作会影响性能,为此查了下相关资料并做记录备忘

<!--more-->

##### 1.DOM操作为什么拉低性能

DOM对象本身也是一个js对象,因此,拉低性能的部分并不是这个对象本身,而是之后触发的浏览器行为(布局,重绘等).

> 一个div元素的第一层属性:
``` bash
var div = document.createElement('div');
var str = ""
for(var key in div){
  str = str + key + "  ";
}
console.log(str);
//输出结果
align  title  lang  translate  dir  dataset  hidden   
tabIndex  accessKey  draggable  spellcheck  contentEditable   
isContentEditable  offsetParent  offsetTop  offsetLeft   
offsetWidth  offsetHeight  style  innerText  outerText  
webkitdropzone  onabort  onblur  oncancel  oncanplay   
oncanplaythrough  onchange  onclick  onclose  oncontextmenu   
oncuechange  ondblclick  ondrag  ondragend  ondragenter   
ondragleave  ondragover  ondragstart  ondrop  ondurationchange   
onemptied  onended  onerror  onfocus  oninput  oninvalid   
onkeydown  onkeypress  onkeyup  onload  onloadeddata   
onloadedmetadata  onloadstart  onmousedown  onmouseenter   
onmouseleave  onmousemove  onmouseout  onmouseover   
onmouseup  onmousewheel  onpause  onplay  onplaying   
onprogress  onratechange  onreset  onresize  onscroll   
onseeked  onseeking  onselect  onshow  onstalled   
onsubmit  onsuspend  ontimeupdate  ontoggle  onvolumechange   
onwaiting  click  focus  blur  namespaceURI  prefix  localName   
tagName  id  className  classList  attributes  innerHTML   
outerHTML  shadowRoot  scrollTop  scrollLeft  scrollWidth   
scrollHeight  clientTop  clientLeft  clientWidth  clientHeight   
onbeforecopy  onbeforecut  onbeforepaste  oncopy  oncut  onpaste   
onsearch  onselectstart  onwheel  onwebkitfullscreenchange   
onwebkitfullscreenerror  previousElementSibling  nextElementSibling   
children  firstElementChild  lastElementChild  childElementCount   
hasAttributes  getAttribute   
getAttributeNS  setAttribute  setAttributeNS   
removeAttribute  removeAttributeNS  hasAttribute  hasAttributeNS   
getAttributeNode  getAttributeNodeNS   
setAttributeNode  setAttributeNodeNS   
removeAttributeNode  closest  matches  webkitMatchesSelector   
getElementsByTagName  getElementsByTagNameNS  getElementsByClassName   
insertAdjacentElement  insertAdjacentText  insertAdjacentHTML   
createShadowRoot  getDestinationInsertionPoints  requestPointerLock   
getClientRects  getBoundingClientRect  scrollIntoView   
scrollIntoViewIfNeeded  animate  remove  webkitRequestFullScreen   
webkitRequestFullscreen  querySelector  querySelectorAll  ELEMENT_NODE   
ATTRIBUTE_NODE  TEXT_NODE  CDATA_SECTION_NODE  ENTITY_REFERENCE_NODE   
ENTITY_NODE  PROCESSING_INSTRUCTION_NODE  COMMENT_NODE  DOCUMENT_NODE   
DOCUMENT_TYPE_NODE  DOCUMENT_FRAGMENT_NODE  NOTATION_NODE  
DOCUMENT_POSITION_DISCONNECTED  DOCUMENT_POSITION_PRECEDING  
DOCUMENT_POSITION_FOLLOWING  DOCUMENT_POSITION_CONTAINS   
DOCUMENT_POSITION_CONTAINED_BY   
DOCUMENT_POSITION_IMPLEMENTATION_SPECIFIC   
nodeType  nodeName  baseURI  isConnected  ownerDocument  parentNode   
parentElement  childNodes  firstChild  lastChild  previousSibling   
nextSibling  nodeValue  textContent   
hasChildNodes  normalize  cloneNode   
isEqualNode  isSameNode  compareDocumentPosition   
contains  lookupPrefix   
lookupNamespaceURI  isDefaultNamespace  insertBefore  appendChild   
replaceChild  removeChild  addEventListener  removeEventListener   
dispatchEvent 
```

1. 浏览器如何呈现页面
    1. 解析HTML,生成DOM树
    2. 解析各种样式,结合DOM树生成Render树
    3. 对Render树各节点计算布局信息
    4. 根据Render树,利用浏览器的UI层进行绘制
    
    > DOM树和Render树并非一一对应,display:none就不会出现在Render树上
    
    > 在这个过程中,最耗费时间的就是**绘制**和**布局**
    
##### 2. 什么情况下会触发重新布局
js脚本执行时,DOM是不会更新的,对DOM的修改会被暂存在一个队列中,在当前js的执行上下文完成执行后,会根据这个队列中的修改,进行一次重新布局;但有时会希望立刻获得最新的DOM节点信息,浏览器不得不提前执行布局,这是DOM性能问题的主要原因.

* 以下操作会导致浏览器重新布局:
    * 通过js获取需要计算的DOM属性
    * 添加或删除DOM元素
    * resize浏览器窗口大小
    * 改变字体
    * css伪类的激活，比如:hover
    * 通过js修改DOM元素样式且该样式涉及到尺寸的改变
    
多次操作DOM的代码:
``` bash
$.each(req.entry,function (i,item) {
    $("<option></option>")
        .val(item["id"])
        .text(item["name"])
        .appendTo($("#actCategories"));
});
```

> 上面代码部分涉及多次添加DOM元素,导致页面多次操作DOM,十分影响性能
    
改进后的代码:
``` bash
for (var i = 0;i < req.entry.length;i++){
    html += '<option value="' + req.entry[i].id + '">' 
    + req.entry[i].name + '</option>'
}
$("#actCategories").html(html);
```
    
> 该解决方法是通过字符串在js执行中拼接完成后插入到父元素中,只执行一次DOM操作,思路是**在JS里完成操作,页面更新后直接全部渲染这部分内容,不必手动操作DOM**,模板引擎渲染页面和表格利用的就是这种原理




##### 3. 关于Virtual DOM
>上面改进部分的解决方案也有很多问题,最主要的是在视图和状态绑定的MVVM模式下,每次一个小小的状态变更都会重新构造整个DOM树,而且input和textarea等会失去自己的焦点,对于局部的小视图更新,上面的方法没有问题,但大型视图或很多局部视图要更新时则不可取.

js表示的ul的DOM对象:

``` bash
var element = {
    tagName: 'ul',
    props: {id: 'list'},
    children: [
        {tagName: 'li', props: {class: 'item'}, children: ["Item 1"]}, 
        {tagName: 'li',props: {class: 'item'},children: ["Item 2"]}, 
        {tagName: 'li', props: {class: 'item'}, children: ["Item 3"]},
    ]
}
//对应的HTML
<ul id='list'> 
    <li class='item'>Item 1</li> 
    <li class='item'>Item 2</li> 
    <li class='item'>Item 3</li> 
</ul>
```

Visual DOM算法的步骤:

1. 用 JavaScript 对象结构表示 DOM 树的结构；然后用这个树构建一个真正的 DOM 树，插到文档当中
2. 当状态变更的时候，重新构造一棵新的对象树。然后用新的树和旧的树进行比较，记录两棵树差异
3. 把2所记录的差异应用到步骤1所构建的真正的DOM树上，视图就更新了

Virtual DOM 本质上就是在 JS 和 DOM 之间做了一个缓存。可以类比 CPU 和硬盘，既然硬盘这么慢，我们就在它们之间加个缓存：既然 DOM 这么慢，我们就在它们 JS 和 DOM 之间加个缓存。CPU（JS）只操作内存（Virtual DOM），最后的时候再把变更写入硬盘（DOM）。**视图的结构确实是整个全新渲染了，但是最后操作DOM的时候确实只变更有不同的地方。**