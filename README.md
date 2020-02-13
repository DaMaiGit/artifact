# artifact

## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Lints and fixes files
```
npm run lint
```

## 花絮

还有xx天就要到情人节了，都说女人永远缺一支口红，在这个武汉肺炎肆虐的情人节里，给心仪的女神一支红红火火的口红，隔空表达自己的想念之情准没错。想到肺炎好之后，女神收到口红后欣喜的样子，想想还有点小激动呢！

<img src="http://wx4.sinaimg.cn/bmiddle/006ARE9vgy1fyq29yeh2gj306o06odfx.jpg" alt="有点小激动（坏坏）" style="zoom:50%;" />

不过想到上一次送女神死亡芭比粉而翻车的经历还历历在目。作为技术直男，面对千千万万的口红色号，一不小心就会在色号这个坑中翻车。因此在这段宅家的日子里，我积极总结吸取教训的过程中，我突然想到能不能通过女神的照片，自动识别出口红的款式，帮助大家了解女神可能使用的口号。

转念间，这个口红分析神器就诞生了！（仅供参考）

有了这个神器，我们就能通过上传女神的照片，轻松的分析出女神所用的口红款式。投其所好，因此邂逅一段完美的爱情，我可真是个小机灵鬼！

## 开始动工

### 严谨的需求分析

在真正开始动工前，作为一名资深的老司机，我们应该进行一轮需求的分析，把握每个每个阶段的目标。因此经过深思熟虑的分析，我们口红分析神器仅仅需要以下两步即可实现：

1. 识别照片中的嘴唇部分的颜色
2. 对比口红颜色，筛选出可能的口红款式

是不是特别简单？想到这次不会翻车的概率更大了，我发出了鹅般的笑声。

### 识别照片中嘴唇颜色

识别照片中嘴唇的颜色，首先我们应该让机器能够像我们一样聪明，能够准确的识别出照片中嘴唇的位置。

Google的AI框架[TensorFlow](https://www.tensorflow.org/)能够帮助开发者搭建训练机器学习的模型。

![image-20200120105947019](/Users/damai/Library/Application Support/typora-user-images/image-20200120105947019.png)

但是对于一般的开发者来说，使用TensorFlow自己去训练模型不是一触而就的，这里面涉及到了许多深度学习的基本概念和TensorFlow的基本概念，例如张量、隐藏层、激活函数、卷积层、池化层等。

<img src="http://wx3.sinaimg.cn/bmiddle/415f82b9gy1fixcz8nckaj20ig0i2js4.jpg" alt="恋爱，真叫人头秃  - 真叫人头秃 _头秃_秃头表情" style="zoom:50%;" />

好在深度学习的应用并不困难，这让我想起了justadudewhohacks大神的人脸识别模型框架:[face-api](https://github.com/justadudewhohacks/face-api.js/)。face-api本质上是构建于TensorFlow.js核心库之上，基于MobileNetV1的神经网络进行人脸样本的训练。总而言之，借助它，我们能够快速使我们的程序获得人脸识别的能力。

#### 深度学习的原理

深度学习可以理解为使用深度神经网络来训练机器，使机器能够获得像人一样的分析学习能力，能够完成识别文字、图像和语音等任务。

在深度学习的过程中主要包括以下三个步骤：

1. 构建深度神经网络
2. 输入数据样本，获得模型并评估效果
3. 优化模型并输出

face-api就已经帮助我们已经做完了以上三个步骤，为我们提供了人脸识别的模型，并封装成了简易的api，帮助开发者在前端项目上方便的使用人脸识别的技术。其中蓝色的部分则是face-api替我们完成的任务。

<img src="https://raw.githubusercontent.com/DaMaiGit/artifact/master/image/1.png" style="zoom:50%;" />



而我们要做的仅仅是输入人脸的图片到已经训练好的模型中，得到输出值即可，即橙色框内的部分。

#### face-api的使用

##### 引入方式

方式1:

如果我们不使用打包工具的话，可以直接将github库中的dist目录下的脚本face-api.js直接导入

```
<script src="face-api.js"></script>
```

方式2:

如果使用npm，我们可以使用npm来引入face-api库

```
npm i face-api.js
```

##### 初始化模型

face-api为我们提供了多个模型，这些模型使用不同的卷积神经网络完成，faceapi包括以下几种模型：

```javascript
var nets = {
    ssdMobilenetv1: new SsdMobilenetv1(), // ssdMobilenetv1 目标检测
    tinyFaceDetector: new TinyFaceDetector(),  // 人脸识别（精简版）
    tinyYolov2: new TinyYolov2(),   // Yolov2 目标检测（精简版）
    mtcnn: new Mtcnn(),   // MTCNN
    faceLandmark68Net: new FaceLandmark68Net(),  // 面部 68 点特征识别
    faceLandmark68TinyNet: new FaceLandmark68TinyNet(), // 面部 68 点特征识别（精简版）
    faceRecognitionNet: new FaceRecognitionNet(),  // 面部识别
    faceExpressionNet: new FaceExpressionNet(),  //  表情识别
    ageGenderNet: new AgeGenderNet()  // 年龄识别
};
```

<img src="https://user-gold-cdn.xitu.io/2020/2/7/1701e6bef6a6864b?w=596&h=1224&f=png&s=146119" style="zoom:50%;" />

face-api工程中的`weights`目录放置了模型文件，我们只需要将`weights`全部拷贝到我们的工程目录中即可。

通过`faceapi.net`我们可以加载对应的模型：

```
faceApi.nets.ssdMobilenetv1.load('/weights') //加载ssdMobilenetv1 目标检测模型
faceApi.nets.faceLandmark68Net.load('/weights')//加载面部 68 点特征识别模型
```

在口红分析神器中，我们仅需要家在ssdMobilenetv1和faceLandmark68Net模型即可。

在模型加载完成以后，我们可以通过调用`faceApi.detectSingleFace(input)`来识别单个人脸或使用`faceapi.detectAllFaces(input)`来识别多张人脸。

```
const detections1 = await faceapi.detectSingleFace(input) //return FaceDetection | undefined
const detections2 = await faceapi.detectAllFaces(input) //return Array<FaceDetection>
```

其中返回值对象FaceDetection包含了一些人脸识别的信息，包括置信度、人脸边框等信息：

```json
{
	"_imageDims": {
		"_width": 225,
		"_height": 225
	},
	"_score": 0.9931827187538147,
	"_classScore": 0.9931827187538147,
	"_className": "",
	"_box": {
		"_x": 75.58209523558617,
		"_y": 12.725257873535156,
		"_width": 86.51270046830177,
		"_height": 99.15830343961716
	}
  ....
}
```

仅仅靠FaceDetection返回的对象我们还无法满足我们的需求，我们可以通过链式的调用，返回人脸的68个特征点信息：

```json
{
  detection: FaceDetection, 
  landmarks: FaceLandmarks68
}
```

其中`landmarks`对象包括了人脸的68个特征点信息，我们可以通过以下代码查看效果：

```javascript
let canvas = faceApi.createCanvasFromMedia(image)
faceApi.draw.drawFaceLandmarks(canvas,landmarks.landmarks) //绘画68个特征点
canvas.toBlob(function(blob){
	console.log(URL.createObjectURL(blob))
})
```

<img src="https://user-gold-cdn.xitu.io/2020/2/7/1701e6b5545f9fa5?w=1210&h=1250&f=png&s=2036527" style="zoom: 33%;" />



可以看到模型通过68个特征点，标记了人脸五官位置。我们可以通过`FaceLandmarks68.positions`获得一个Array(68)的数组，这68个数组元素则是68个特征点。而这68个点位的分布如下：

​	<img src="https://user-gold-cdn.xitu.io/2020/2/7/1701e6a7391e809c?w=1156&h=854&f=png&s=192734" style="zoom: 33%;" />

`FaceLandmarks68`还提供了一些方法，来获取五官的点位信息，帮助我们更方便的使用：

```javascript
let landmarkPositions = landmarks.positions  // 获取全部 68 个点
let jawline = landmarks.getJawOutline()  // 下巴轮廓 1~17
let leftEyeBbrow = landmarks.getLeftEyeBrow()  // 左眉毛 18~22
let rightEyeBrow = landmarks.getRightEyeBrow()  // 右眉毛 23~27
let nose = landmarks.getNose()  // 鼻子  28~36
let leftEye = landmarks.getLeftEye()  // 左眼 37~40
let rightEye = landmarks.getRightEye()  // 右眼 43~48
let mouth = landmarks.getMouth()  // 嘴巴 49~68
```

因此我们可以通过`FaceLandmarks68.getMouth() `方法获取嘴部的20个特征点。并通过循环遍历特征点的位置，来获取对应位置的canvas信息。

```javascript
		/**
     * 获取嘴部20个特征点的颜色
     * @param canvans
     * @param mouthPoint
     */
    getMouthColor:function (canvans,mouthPoint) {
      let context = canvans.getContext("2d")
      for (let i =0;i<mouthPoint.length;i++){
        let data = context.getImageData(mouthPoint[i]["_x"],mouthPoint[i]["_y"],1,1)
        this.mouthColors[i] = data.data
      }
    }
```

到此我们已经完成了获得女神嘴唇位置的颜色，接下来就是将嘴部20个特征点的颜色和口红的颜色进行匹配，选择相近的颜色值。

#### 口红颜色匹配

面对无穷无尽的颜色，作为钢铁直男的我一时无从下手，当我准备去口红店偷口红颜色的时候，突然看到github上一个口红可视化颜色库**[ lipstick](https://github.com/Ovilia/lipstick)**。这个库收录了纪梵希、圣罗兰等口红品牌的口红颜色。

![image-20200205103144601](/Users/damai/Library/Application Support/typora-user-images/image-20200205103144601.png)

项目中的口红颜色以一个json格式存储：

```json
{
	"brands":[
    {
      "name":"圣罗兰",
      "series":[
        {
          "name": "莹亮纯魅唇膏",
          "lipsticks": [{
            "color": "#D62352",
            "id": "49",
            "name": "撩骚"
         },
         ... 
        }
      ]
    },
    ...
  ] 
}
```

因此我们直接复制json格式文件，将嘴部的20个特征点的颜色分别遍历口红库中口红的颜色，进行对比。获得差距最小的一款口红作为我们匹配的结果值即可。

在这里计算嘴唇的颜色和口红的颜色，采用了欧几里得距离公式进行计算。在这里我们只需要进行两步。

1、将口红库的`color`字段从十六进制转换为RGB模式

2、将口红的RGB和嘴唇部分的20个嘴部特征点RGB带入欧几里得距离公式，计算距离最小的一款口红作为最终结果

我们可以将RGB作为颜色的三个维度的方向，带入欧几里得公式计算：

![img](https://gss2.bdstatic.com/-fo3dSag_xI4khGkpoWK1HF6hhy/baike/pic/item/ac4bd11373f08202bc7559a147fbfbedaa641b4d.jpg)

```
compareColor:function(r1,g1,b1,r2,g2,b2){
      let r = Math.pow((r1-r2),2)
      let g = Math.pow((g1-g2),2)
      let b = Math.pow((b1-b2),2)
      return Math.sqrt((r+g+b))
}
```

将20个嘴部特征点依次遍历口红库中的颜色，记录距离最小的一款口红作为最终结果展示到UI界面上。

到此，我们的口红分析神奇就大功告成啦！

### 成果展示

<img src="https://user-gold-cdn.xitu.io/2020/2/7/1701e69e9b449690?w=1278&h=1340&f=png&s=557600" style="zoom:30%;" />

### 写在最后

在这个项目中还有很多可以优化的点，例如：口红颜色的库的完善，部分照片模型识别的准确度等。这篇文章只是起到抛砖引玉的作用，能够帮助我们非人工智能的开发工程师感受AI的魅力和乐趣，也为我们在日常的开发过程中，提供了一个新的维度方向去思考问题的解决方案。

因此AI相关的知识是值得我们在闲暇时间学习的，如果你想继续学习TensorFlow相关的知识，推荐可以阅读《TensorFlow实战Google深度学习框架》这本书或直接访问[TensorFlow官网](https://www.tensorflow.org/)，如果想要通过视频学习AI相关的知识，推荐可以观看吴恩达教授的《机器学习》课程。在此过程中，你一定会更深入的了解机器学习相关知识，并受益匪浅。

最后，受武汉肺炎影响，希望大家能够出门戴口罩，保护好自己。分隔两地的情侣也请避免在这时候千里迢迢相见，毕竟真爱不会因为距离所阻隔。最后的最后，祝天下所有的有情人终成眷属！

