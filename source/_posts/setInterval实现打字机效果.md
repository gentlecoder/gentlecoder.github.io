---
title: setInterval实现打字机效果
date: 2019-04-10 23:23:35
tags:
  - JavaScript
---

## 简单打字机

**HTML**

```html
<pre id="test">
</pre>
```

js

```js
const test = `function loadItem(container, text) {
  let num = 0
  let sum = text.length
  let interval = 16

  const startLoad = () => {
    setTimeout(() => {
      num += 1
      if (num <= sum) {

        let str = text.substr(0, num)

        container.scrollTop = 100000

        container.innerHTML = str

        setTimeout(() => {
          startLoad()
        }, interval)

      }
    }, interval)
  }

  startLoad()
}`;

const loadText = (str, container) => {
  let num = 0,
    len = str.length;
  let interval = setInterval(() => {
    if (num > len) clearInterval(interval);
    container.textContent = str.slice(0, num++);
  }, 20);
};
loadText(test, document.getElementById("test"));
```

## 来点复杂的

```html
<!doctype html>
<html>
<head>
<meta content="text/html; charset=UTF-8" http-equiv="Content-Type" />
<title>JavaScript Typewriter - Example</title>
<script type="text/javascript">
function Typewriter (sSelector, nRate) {

  function clean () {
    clearInterval(nIntervId);
    bTyping = false;
    bStart = true;
    oCurrent = null;
    aSheets.length = nIdx = 0;
  }

  function scroll (oSheet, nPos, bEraseAndStop) {
    if (!oSheet.hasOwnProperty("parts") || aMap.length < nPos) { return true; }
		// 通过bExit来控制是否已经打印完成
    var oRel, bExit = false;

    if (aMap.length === nPos) { aMap.push(0); }

    while (aMap[nPos] < oSheet.parts.length) {
      oRel = oSheet.parts[aMap[nPos]];

      scroll(oRel, nPos + 1, bEraseAndStop) ? aMap[nPos]++ : bExit = true;
			// 没有停止或者擦除并且是存在值的文字节点
      if (bEraseAndStop && (oRel.ref.nodeType - 1 | 1) === 3 && oRel.ref.nodeValue) {
        bExit = true;  // 控制循环
        oCurrent = oRel.ref;
        sPart = oCurrent.nodeValue;  // 保存内容
        oCurrent.nodeValue = "";
      }
			// 这里清空了文字子节点的内容，不然会直接出现文字
      oSheet.ref.appendChild(oRel.ref);
      if (bExit) { return false; }
    }

    aMap.length--;
    return true;
  }

  function typewrite () {
    // 当aSheets[0]遍历完了之后才开始nIdx++
    if (sPart.length === 0 && scroll(aSheets[nIdx], 0, true) && nIdx++ === aSheets.length - 1) { clean(); return; }
		// 开始打字
    oCurrent.nodeValue += sPart.charAt(0);
    sPart = sPart.slice(1);
  }

  function Sheet (oNode) {
    this.ref = oNode;
    if (!oNode.hasChildNodes()) { return; }
    this.parts = Array.prototype.slice.call(oNode.childNodes);
		// 将需要打字机效果的节点的子节点清空，递归遍历，将节点信息存入对象中
    for (var nChild = 0; nChild < this.parts.length; nChild++) {
      oNode.removeChild(this.parts[nChild]);
      this.parts[nChild] = new Sheet(this.parts[nChild]);
    }
  }

  // aMap控制打印队列
  var
    nIntervId, oCurrent = null, bTyping = false, bStart = true,
    nIdx = 0, sPart = "", aSheets = [], aMap = [];

  this.rate = nRate || 100;
 
  this.play = function () {
    if (bTyping) { return; }
    if (bStart) {
      var aItems = document.querySelectorAll(sSelector);

      if (aItems.length === 0) { return; }
      for (var nItem = 0; nItem < aItems.length; nItem++) {
        aSheets.push(new Sheet(aItems[nItem]));
        /* Uncomment the following line if you have previously hidden your elements via CSS: */
        // aItems[nItem].style.visibility = "visible";
      }

      bStart = false;
    }

    nIntervId = setInterval(typewrite, this.rate);
    bTyping = true;
  };
 
  this.pause = function () {
    clearInterval(nIntervId);
    bTyping = false;
  };
 
  this.terminate = function () {
    oCurrent.nodeValue += sPart;
    sPart = "";
    for (nIdx; nIdx < aSheets.length; scroll(aSheets[nIdx++], 0, false));
    clean();
  };
}
 
/* usage: */
var oTWExample1 = new Typewriter(/* elements: */ "#article, h1, #info, #copyleft", /* frame rate (optional): */ 15);
 
/* default frame rate is 100: */
var oTWExample2 = new Typewriter("#controls");

/* you can also change the frame rate value modifying the "rate" property; for example: */
// oTWExample2.rate = 150;
 
onload = function () {
  oTWExample1.play();
  oTWExample2.play();
};
</script>
<style type="text/css">
span.intLink, a, a:visited {
  cursor: pointer;
  color: #000000;
  text-decoration: underline;
}
 
#info {
  width: 180px;
  height: 150px;
  float: right;
  background-color: #eeeeff;
  padding: 4px;
  overflow: auto;
  font-size: 12px;
  margin: 4px;
  border-radius: 5px;
  /* visibility: hidden; */
}
</style>
</head>
 
<body>

<p id="copyleft" style="font-style: italic; font-size: 12px; text-align: center;">CopyLeft 2019 by <a href="http://gentlecoder.cn/" target="_blank">gentlecoder</a></p>
<p id="controls" style="text-align: center;">[&nbsp;<span class="intLink" onclick="oTWExample1.play();">Play</span> | <span class="intLink" onclick="oTWExample1.pause();">Pause</span> | <span class="intLink" onclick="oTWExample1.terminate();">Terminate</span>&nbsp;]</p>
<div id="info">
Vivamus blandit massa ut metus mattis in fringilla lectus imperdiet. Proin ac ante a felis ornare vehicula. Fusce pellentesque lacus vitae eros convallis ut mollis magna pellentesque. Pellentesque placerat enim at lacus ultricies vitae facilisis nisi fringilla. In tincidunt tincidunt tincidunt.
</div>
<h1>JavaScript Typewriter</h1>
 
<div id="article">
<p>Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nullam ultrices dolor ac dolor imperdiet ullamcorper. Suspendisse quam libero, luctus auctor mollis sed, malesuada condimentum magna. Quisque in ante tellus, in placerat est. Pellentesque habitant morbi tristique senectus et netus et malesuada fames ac turpis egestas. Donec a mi magna, quis mattis dolor. Etiam sit amet ligula quis urna auctor imperdiet nec faucibus ante. Mauris vel consectetur dolor. Nunc eget elit eget velit pulvinar fringilla consectetur aliquam purus. Curabitur convallis, justo posuere porta egestas, velit erat ornare tortor, non viverra justo diam eget arcu. Phasellus adipiscing fermentum nibh ac commodo. Nam turpis nunc, suscipit a hendrerit vitae, volutpat non ipsum.</p>
<form name="myForm">
<p>Phasellus ac nisl lorem: <input type="text" name="email" /><br />
<textarea name="comment" style="width: 400px; height: 200px;">Nullam commodo suscipit lacus non aliquet. Phasellus ac nisl lorem, sed facilisis ligula. Nam cursus lobortis placerat. Sed dui nisi, elementum eu sodales ac, placerat sit amet mauris. Pellentesque dapibus tellus ut ipsum aliquam eu auctor dui vehicula. Quisque ultrices laoreet erat, at ultrices tortor sodales non. Sed venenatis luctus magna, ultricies ultricies nunc fringilla eget. Praesent scelerisque urna vitae nibh tristique varius consequat neque luctus. Integer ornare, erat a porta tempus, velit justo fermentum elit, a fermentum metus nisi eu ipsum. Vivamus eget augue vel dui viverra adipiscing congue ut massa. Praesent vitae eros erat, pulvinar laoreet magna. Maecenas vestibulum mollis nunc in posuere. Pellentesque sit amet metus a turpis lobortis tempor eu vel tortor. Cras sodales eleifend interdum.</textarea></p>
<p><input type="submit" value="Send" />
</form>
<p>Duis lobortis sapien quis nisl luctus porttitor. In tempor semper libero, eu tincidunt dolor eleifend sit amet. Ut nec velit in dolor tincidunt rhoncus non non diam. Morbi auctor ornare orci, non euismod felis gravida nec. Curabitur elementum nisi a eros rutrum nec blandit diam placerat. Aenean tincidunt risus ut nisi consectetur cursus. Ut vitae quam elit. Donec dignissim est in quam tempor consequat. Aliquam aliquam diam non felis convallis suscipit. Nulla facilisi. Donec lacus risus, dignissim et fringilla et, egestas vel eros. Duis malesuada accumsan dui, at fringilla mauris bibStartum quis. Cras adipiscing ultricies fermentum. Praesent bibStartum condimentum feugiat.</p>
<p>Nam faucibus, ligula eu fringilla pulvinar, lectus tellus iaculis nunc, vitae scelerisque metus leo non metus. Proin mattis lobortis lobortis. Quisque accumsan faucibus erat, vel varius tortor ultricies ac. Lorem ipsum dolor sit amet, consectetur adipiscing elit. Sed nec libero nunc. Nullam tortor nunc, elementum a consectetur et, ultrices eu orci. Lorem ipsum dolor sit amet, consectetur adipiscing elit. Pellentesque a nisl eu sem vehicula egestas.</p>
</div>
</body>
</html>
```

