# GoJS绘图插件使用笔记

## GoJS介绍
```
GoJS 是一个实现交互式图表(Diagram)的Javascript 库。 因为GoJS是一个依赖于HTML5特性的JavaScript库，所以你要搞清楚浏览器是否支持HTML5
```

## 引库
```
<script src="../release/go.js"></script>
<script src="../assets/js/goSamples.js"></script>
```

当然你也可以直接引用这个cdn
```$xslt
<script src="https://cdnjs.cloudflare.com/ajax/libs/gojs/1.7.2/go-debug.js"></script>
```

引库完成以后，就可以创建一个dom节点用于绑定Diagram


## 常用api介绍
1 画布元素属性定义类API
```javascript
    function init() {
        var $ = go.GraphObject.make; // for conciseness in defining templates

        const myDiagram = $(
             go.Diagram,
             'myDiagramDiv', // 将节点绑定到id为'myDiagramDiv'的dom上
             // self.id,
             {
                 // allowSelect: false, // 允许选中
                 // allowZoom: false, // 允许缩放，false为否
                 // allowHorizontalScroll: false, // 允许水平方向拖拽
                 // hasVerticalScrollbar  : false, // 显示垂直方向滚动条
                 // initialContentAlignment: go.Spot.LeftCenter, // 画布的位置设置（居中，靠左等）
                 // 'undoManager.isEnabled': true, // enable undo & redo
                 // isEnabled: false, // 禁止编辑操作
                 // isReadOnly: true, // 禁止拖拽节点
                 // initialPosition: new go.Point(0, 0), // 画布起始位置
                 // padding: 0,
                 "animationManager.isEnabled": true,
             }
        );
    }
```


2 节点属性定义类API
```javascript
// myDiagram.nodeTemplate=$(go.Node,***) //***为对节点的定义

myDiagram.nodeTemplate = $(
    go.Node,
    'Vertical', // the Shape will go around the TextBlock
    { selectionChanged: nodeSelectionChanged },//节点选中改变事件，nodeSelectionChanged为回调的方法名

    // 定义节点的坐标
    new go.Binding('location', 'loc', go.Point.parse), 

    // 节点的连线出发点和连接点位置
    { fromSpot: go.Spot.Right, toSpot: go.Spot.Left },

    // 节点形状属性
    $(
        go.Shape,
        'Circle',
        {
            strokeWidth: 1, // 节点边框宽度
            fill: 'red', // 节点填充颜色
            stroke: 'white' // 边框填充色
        },
    ),

    // 节点内容            
    $(
        go.TextBlock,
        {
            visible: false, // 是否显示
            font: 'normal 10px sans-serif',
            stroke: 'white', //文本样式
            text: '节点内容文字'
        }
    )
);
```

3 连线属性定义类API
```javascript
   myDiagram.linkTemplate = $(
       go.Link,
       {
           routing: go.Link.AvoidsNodes, //Normal: 直线, Orthogonal: 曲线相交, or AvoidsNodes: 曲线不相交.
           corner: 10 // 曲线连线的弧度
       },
   
       //线
       $(
           go.Shape,
           {
               strokeWidth: 1, // 连线宽度
               stroke: '#2EC7C9' // 连线颜色
           }
       ),
       //箭头
       $(
           go.Shape,
           {
               toArrow: 'OpenTriangle', // 箭头样式 OpenTriangle or Triangle
               visible: false, // 是否显示
               stroke: '#2EC7C9', // 边框颜色
               fill: '#2EC7C9' // 填充颜色
           }
       ),
   
       // 线上面的文字
       $(
           go.TextBlock,
           '显示在连线上的内容',
           {
               segmentIndex: -1, // 0内容在线的左边， -1内容在线的右边， 不设即在中间
               segmentOffset: new go.Point(-20, 10) // 文本的偏移量
           }
       )
   )
```

##绘图方法

只需调用GraphLinksModel(), 接收两个对象数组，一个节点对象list， 一个连线对象list
节点对象中必须有一个唯一的key, loc是我动态绑定的节点坐标
连线对象中必须有出发节点的key和接收节点的key
```javascript
   myDiagram.model = new go.GraphLinksModel(
        [
          { key: "Alpha", loc: "20 20"},
          { key: "Beta", loc: "60 20"},
          { key: "Gamma", loc: "100 20"},
          { key: "Delta", loc: "140 20"}
        ],
        [
          { from: "Alpha", to: "Beta" },
          { from: "Alpha", to: "Gamma" },
          { from: "Beta", to: "Beta" },
          { from: "Gamma", to: "Delta" },
          { from: "Delta", to: "Alpha" }
   ]);
```

```$xslt
    上面的节点模板和连线模板都是写死了的数据属性缺乏灵活性，如果要是动态绑定这些属性该怎么做呢，这时候后就得
    使用GoJS的Binding方法了。涉及到设置节点和连线的属性都可以使用动态绑定
    用法： new go.Binding(动态绑定的key, 传入key的值)
    例如上面的节点模板定动态绑定的一个坐标属性new go.Binding('location', 'loc'), loc就是传进去的key的值
```

## 生成图片
因为使用GoJS绘图时需要绑定一个宽高固定的dom， 但是很多情况下，不能确定画出来的图表的大小，导致出现画布
大没法自适应。也有其他业务情况下也有存在图片导出的情况

```javascript
// 生成imgSrc
myDiagram.makeImageData()

// 生成image
myDiagram.makeImage()

// 生成SVG
myDiagram.makeSvg()

// 常用参数
myDiagram.makeImageData({
    scale: 1, // 缩放
    size: new go.Size(1200, NaN), //大小 
    // maxSize: new go.Size(1190, Infinity), // 最大
    // position: new go.Point(35, 0), // 起始位置
    // showTemporary: true,
    // showGrid: true, // 显示网格
    padding: new go.Margin(0, 0, 30, 50),
    position: new go.Point(0, 0)
})
```


GoJS去水印

在go.js文件中搜索关键字：7eba17a4ca3b1a8346，会找到类似
```$xslt
a.Kv=d[w.Kg(“7eba17a4ca3b1a8346”)]w.Kg(“78a118b7”);
```
把a.Kv=d[w.Kg(“7eba17a4ca3b1a8346”)]w.Kg(“78a118b7”);替换为a.Kv =function(){return true;};就可以了.
去水印后


