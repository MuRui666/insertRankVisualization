# canvas+js实现直接插入排序可视化
先展示一下动画效果：
![直接插入排序可视化](https://upload-images.jianshu.io/upload_images/19442495-85976b0384046c8c.gif?imageMogr2/auto-orient/strip)

####首先先简单了解一下什么是直接插入排序：
  直接插入排序会在遍历的过程中将一个序列分为两部分，前半部分是已经排好了的有序序列，后半部分是还未排序的无序序列。在无序序列中从左往右遍历，每一步将一个待排序的元素，按其排序码的大小，插入到前面已经排好序的一组元素的适当位置上去，直到元素全部插入为止。

####那基本概念已经了解了，现在就用代码将这样一个排序的过程展示出来：
1、既然要画画，那我们就需要一块画布：
```
<canvas id="canvas"></canvas>
```
并且对即将绘制的图形进行一些设置：
```
  const WIDTH = 500,HEIGHT = 300//画布的宽高
  const COLUMN_WIDTH = 15,COLUMN_MARGIN = 1//条形图的一个竖条的宽以及两个竖条之间的间隔
  const LENGTH = Math.floor(WIDTH / (COLUMN_WIDTH + COLUMN_MARGIN))//根据画布的大小、竖条的宽以及两个竖条之间的距离计算出一个画布可以容纳的竖条
  const canvas = document.getElementById('canvas')//获取画布
  const ctx = canvas.getContext('2d')//创建画笔
  let sortArray = new Array()//存储需要排序的序列
  let animationTime = 3
  canvas.height = HEIGHT
  canvas.width = WIDTH
```
2、准备工作做好了，那我们就需要有一个需要排序的序列，这里就用随机数的方式创建一个无序数组：
```
 function init(){
            for(let i = 0; i < LENGTH; i++){
                sortArray[i] = Math.floor(Math.random() * HEIGHT + 1)
            }
        }
```
3、将这个无序序列的每一项变成条形：
 ```
 function drawColumn(ctx, index, height, color){//画笔、当前当前遍历的元素的索引、该元素的高度、颜色
            ctx.fillStyle = color
            let x = index * (COLUMN_MARGIN + COLUMN_WIDTH)
            let y = HEIGHT
            ctx.fillRect(x,y,COLUMN_WIDTH,-height)//左上角的x坐标，左上角的y坐标，元素的宽度，元素的高度
        }
 ```
4、将变成条形的序列渲染出来：
```
function render(ctx){
            for(let i = 0; i < sortArray.length; i++){
                drawColumn(ctx,i,sortArray[i],'black')
            }
        }
```
调用render(ctx),之后我们打开浏览器就可以看到这个无序序列就已经展示在页面上啦,如下图所示：
![未排序的静止状态](https://upload-images.jianshu.io/upload_images/19442495-0d0994d0f41b6a7d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
5、我们可以发现这个序列虽然展示出来了，但还是无序的。接下来就用直接插入排序的方式给他排个序：
```
  function sort(ctx){
            let item//哨兵，用来保存要交换位置的其中一个元素，这样交换的时候就不会有将上一个元素覆盖之后丢失的情况了
            for(let i = 0; i < LENGTH; i++){
                for(let j = i + 1; j > 0; j--){
                    if(sortArray[j-1] > sortArray[j]){
                        item = sortArray[j-1]
                        sortArray[j-1] = sortArray[j]
                        sortArray[j] = item
                    }else{
                        break
                    }
                }
            }
        }
```
排完序后，可以看到序列已经从小到的排好序了：
![已排序的静止状态](https://upload-images.jianshu.io/upload_images/19442495-879b5e9cff88b07e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
6、接下来就要让它从无序变为有序的这个过程以动画的形式展示出来。所以在sort函数中，每当当前索引改变的时候都需要更新一下视图。这里需要对sort函数进行改造：
```
  function sort(ctx){
            let item
            for(let i = 0; i < LENGTH; i++){
                for(let j = i + 1; j > 0; j--){
                    if(sortArray[j-1] > sortArray[j]){
                        updateView(ctx, copyArray(sortArray), i, j - 1, j)//当交换位置的时候更新视图
                        item = sortArray[j-1]
                        sortArray[j-1] = sortArray[j]
                        sortArray[j] = item
                    }else{
                        break
                    }
                    updateView(ctx, copyArray(sortArray), i, j - 1, j)//当索引跳到下一个未排序的元素时更新视图
                }
                updateView(ctx, copyArray(sortArray), i, -1, -1)//当当前索引改变时更新视图
            }
        }
```
每当原数组发生改变，就将它复制一份,传入updateView中更新视图：
```
 function copyArray(arr){
            let newArr = new Array()
            for(let i = 0; i < arr.length; i++){
                newArr[i] = arr[i]
            }
            return newArr
        }
```
update函数中有一个定时器，每隔一定的时间就会重新渲染一次：
```
function updateView(ctx, arr, currentIndex, compareIndex1, compareIndex2){
            setTimeout(() => {
                render(ctx, arr, currentIndex, compareIndex1, compareIndex2)
            },animationTime++*20)
        }
```
7、为了能让整个过程更加清晰，这里在渲染的时候会将已排序的元素用绿色表示出来，未排序的元素用黑色表示，当前正在比较的元素左侧的用橙色表示，右侧的用绿色表示，这里需要改造一下render函数：
```
  function render(ctx, arr, currentIndex, compareIndex1, compareIndex2){
            ctx.clearRect(0,0,WIDTH,HEIGHT)
            for(let i = 0; i < arr.length; i++){
                if(i == compareIndex1){
                    drawColumn(ctx,i,arr[i],'orange')
                }else if(i == compareIndex2){
                    drawColumn(ctx,i,arr[i],'blue')
                }else if(i < currentIndex){
                    drawColumn(ctx,i,arr[i],'green')
                }else{
                    if(currentIndex == arr.length - 1){
                        drawColumn(ctx,i,arr[i],'green')
                    }else{
                        drawColumn(ctx,i,arr[i],'black')
                    }
                }
            }
        }
```
到这里就结束啦，运行render函数就可以看到动态的排序过程。