---
title: '''threejs简单教程'''
date: 2018-02-28 18:06:27
comments: true
tags: 
  - 前端
  - threejs
---

![threejs](/assets/blogImg/threejs.png)      
> 简单介绍一下threejs的入门知识~~~~     

<!-- more -->

## 常用概念

- 场景

> 场景允许你设置哪些对象被`threejs`渲染以及渲染在哪里。我们在场景中放置对象、灯光和相机。

- 相机

> 相机的抽象基类。当我们构建新类型的相机时总是应该继承这个类。一般用的是远景相机以及正交相机。
>

- 渲染器

> 目前主要是WebGL渲染器，canvas渲染器在未来版本的`threejs`中将不会被支持。
>

- 光照

> 使用的比较多的是点光源、平行光、环境光以及聚光源，光照一般配合其相关的阴影一起使用。

- 模型

> 一个Geometry存有用来描述一个三维模型的所有必要数据。

- 材质

> 用于定义3D物体面所用材质，不同的材质其与光源相互作用产生的效果不同。

- 纹理

> 用于定义3D物体面上的图案和纹理，可以导入外部文件进行渲染。

- 对象

> 一个3D模型对应一个对象。

- 辅助对象

> 用于创建3D对象时添加相关的辅助点、线、面。

- 动画

> 定义3D物体的动画效果

## 开始

Threejs使用面向对象的方式来构建程序，包含3个基本对象： **场景（scene）**, **相机（camera）**, 以及一个**渲染器（renderer）**。 拿电影来类比的话，场景对应于整个布景空间，相机是拍摄镜头，渲染器用来把拍摄好的场景转换成胶卷（对于网页来讲，是电脑屏幕）。 场景和相机代表了3D观察空间和数据模型，渲染器则包含了WebGL绘图上下文和着色器。

````javascript
var scene = new THREE.Scene();
var camera = new THREE.PerspectiveCamera( 75, window.innerWidth / window.innerHeight, 0.1, 1000 );
var renderer = new THREE.WebGLRenderer();
renderer.setSize( window.innerWidth, window.innerHeight );
document.body.appendChild( renderer.domElement );
````

上面的代码构建了scene, camera 和 renderer。Three.js的架构支持多种camera，这里使用最常见的远景相机（PerspectiveCamera），也就是类似于人眼观察的方式。第一个属性75设置的是**视角（field of view）**。

第二个属性设置的是相机拍摄面的**长宽比（aspect ratio）**。我们几乎总是会使用元素的宽除以高，否则会出现挤压变形。

接下来的2个属性是**近裁剪面（near clipping plane）** 和 **远裁剪面（far clipping plane）**。下面这张图可以帮助你理解：

![img](http://techbrood.com/ueditor/php/upload/image/20160525/1464141326848754.png)

这几个参数所限定的绿色3D空间被称之为视椎体（View Frustum），用来裁剪视图，在该视锥体以外的物体将不会被渲染。我们暂时可以先不管，但你需要了解这个空间和渲染性能有关。

接下来是渲染器，所有魔法效果都在这里产生。除了我们这里使用的WebGLRenderer，three.js还支持一些其它渲染器，基本上只是用来回退处理那些不支持WebGL的旧式用户浏览器。

除了创建renderer实例，我们还需要设置渲染空间的尺寸，一般就使用目标屏幕的宽高（window.innerWidth和window.innerHeight），也可以给定一个小尺寸。

如果你想保持渲染空间的尺寸，但使用一个较低的分辨率，你可以在调用**setSize**的时候设置参数**updateStyle**为false，比如 **setSize(window.innerWidth/2, window.innerHeight/2, false)** 将使用1/2分辨率来绘制你的应用程序，假定<canvas>为100%的宽高。

最后，我们把 **renderer** 元素添加到HTML文档中。这里是一个 <canvas> 元素，渲染器用来显示场景。

上面的都是准备工作，电影布景都好了，演员还没进场。接下来我们添加“演员”（3D立方体）。

````javascript
var geometry = new THREE.BoxGeometry( 1, 1, 1 );
var material = new THREE.MeshBasicMaterial( { color: 0x00ff00 } );
var cube = new THREE.Mesh( geometry, material );
scene.add( cube );
camera.position.z = 5;
````

为了创建一个立方体，我们需要使用**盒子模型（BoxGeometry）**，这是一个包含立方体所有顶点和填充面的对象。

除了这个几何模型（geometry）外，我们还需要一个材料（material）来对其着色。Three.js支持多种材料，现在我们只使用 **网孔基础材料（MeshBasicMaterial）**。 所有材料都含有一个属性对象。这里简单起见，我们只提供了颜色值为**0x00ff00**，表示绿色。这和CSS和Photoshop中16进制颜色值一样。

第3件事是我们需要一个**网孔(Mesh)**。网孔是用来承载几何模型的一个对象，还可以把材料应用到它上面，然后添加到场景中完成旋转动画。

默认情况下，当我们调用 **scene.add()** 时，对象将被添加到原点处，即坐标点**(0,0,0)**，这将导致相机和立方体发生空间重叠。为了避免这样，我们把相机（camera）的位置移出来一些。

## 渲染场景

如果你在新建作品中拷贝了前述的代码，并点击[运行]，你还是看不到任何东西，因为我们还没有实际去渲染它。为此，我们需要一个 **渲染循环（render loop）**。

````javascript
function render() {
	requestAnimationFrame( render );
	renderer.render( scene, camera );
}
render();
````

这将创建一个循环，以每秒60次的频率来绘制场景。阅读过本站相关教程的同学，可能会熟悉requestAnimationFrame这个函数，它用来替代 **setInterval**， 这个新接口具备多个优点，比如浏览器Tab切换后停止渲染以节约资源、和屏幕刷新同步避免无效刷新、在不支持该接口的浏览器中能安全回退为setInterval。

## 创建动画

如果你一步步完成前述的代码，点击[运行]，你现在应该可以看到一个绿色的立方体，现在我们添加一点动画，让它转动起来。

把如下代码加在 **render** 函数中 **renderer.render** 这行代码之前：

````javascript
cube.rotation.x += 0.1;
cube.rotation.y += 0.1;
````

这样我们就使用Three.js完成了一个旋转的立方体。基本上，如果要改变立方体的运动，我们都是在render循环中处理。

## DEMO

````html
<html>
	<head>
		<title>My first Three.js app</title>
		<style>
			body { margin: 0; }
			canvas { width: 100%; height: 100% }
		</style>
	</head>
	<body>
		<script src="//wow.techbrood.com/libs/three.r73.js"></script>
		<script>
			var scene = new THREE.Scene();
			var camera = new THREE.PerspectiveCamera( 75, window.innerWidth/window.innerHeight, 0.1, 1000 );

			var renderer = new THREE.WebGLRenderer();
			renderer.setSize( window.innerWidth, window.innerHeight );
			document.body.appendChild( renderer.domElement );

			var geometry = new THREE.BoxGeometry( 1, 1, 1 );
			var material = new THREE.MeshBasicMaterial( { color: 0x00ff00 } );
			var cube = new THREE.Mesh( geometry, material );
			scene.add( cube );

			camera.position.z = 5;

			var render = function () {
				requestAnimationFrame( render );

				cube.rotation.x += 0.1;
				cube.rotation.y += 0.1;

				renderer.render(scene, camera);
			};

			render();
		</script>
	</body>
</html>
````

