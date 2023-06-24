# eluosifangkcly.github.io
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <script src="https://cdn.bootcdn.net/ajax/libs/lodash.js/4.17.21/lodash.min.js"></script>
    <style>
        *{
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        body{
            height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center; 
        }
        .container{
            position: relative;
            width: 280px;
            height: 504px;
           /*  background-image: url(star.jpg);
            background-size: 100% 100%; */
            box-shadow: 0 0 5px rgb(218, 218, 218),
            0 0 10px rgb(151, 151, 151);
            backdrop-filter: blur(3px);
            background-color: rgba(0, 0, 0,.2);
        }
        .mark{
            font-family: 'fangsong';
            position: absolute;
            height: 50px;
            width: 150px;
            bottom: 50px;
            right: -150px;
            text-align: center;
            color: rgb(197, 197, 197);
            line-height: 50px;
            font-size: 25px;
            user-select: none;
        }
        .again{
            font-family: 'fangsong';
            position: absolute;
            height: 30px;
            width: 100px;
            right: -125px;
            bottom: 10px;
            line-height: 30px;
            text-align: center;
            box-shadow: inset 0 0 10px white;
            color: rgb(255, 255, 255);
            cursor: pointer;
            border-radius: 5px;
            user-select: none;
        }
        .again:hover{
            background-color: rgb(21, 168, 29);
        }
        .kuai{
            position: absolute;
            width: 28px;
            height: 28px;
            /* border: 0.1px solid rgb(0, 0, 0); */
        }
        .ding{
            position: absolute;
            width: 28px;
            height: 28px;
            background-color: rgb(243, 243, 243);
        }
        video{
            position: fixed;
            width: 100%;
            height: 100%;
            object-fit: cover;
            z-index: -10;
        }
        h1{
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%,-50%);
            font-size: 100px;
            font-family: 'fangsong';
            color: rgb(1, 175, 255);
            text-shadow: 0 0 10px rgb(1, 175, 255),
            0 0 15px rgb(1, 175, 255),
            0 0 20px rgb(1, 175, 255);
        }
    </style>
</head>
<body onload="rukou()">

     <video src="11.mp4" autoplay muted loop></video>
     <h1>俄罗斯方块</h1>

    <div class="container" id="container">
       <!--  <div class="kuai"></div> -->
       <div class="mark">分数：0 </div>
       <div class="again">再来一局</div>
    </div>

     <script>
         /* 显示分数 */
         var mark = document.querySelector(".mark");
         /* 分数变量 */
         var marks = 0;
         /* 再来一局 */
         var again = document.querySelector(".again");

         /* 常量，不可修改 */
         /* 一次走的距离 */
         var STEP = 28;

         /* 容器一共18行，10列 */
         var ROW_COUNT = 18 , COL_COUNT = 10;

         /* 记录所有出现的块元素的位置 k=行_列 ：v=块元素 */
         var fixedBlocks = {}

         /* 创建每个模型的数据源 */
         var MODELS = [
             /* L型模型，里面每个盒子相对16宫格的坐标 */
             {
                0:{row:2,col:0},
                1:{row:2,col:1},
                2:{row:2,col:2},
                3:{row:1,col:2}
             },
             // 第2个模型数据源(凸)
            {
        0: {
          row: 1,
          col: 1
        },
        1: {
          row: 0,
          col: 0
        },
        2: {
          row: 1,
          col: 0
        },
        3: {
          row: 2,
          col: 0
        }
      },
      // 第3个模型数据源(田)
      {
        0: {
          row: 1,
          col: 1
        },
        1: {
          row: 2,
          col: 1
        },
        2: {
          row: 1,
          col: 2
        },
        3: {
          row: 2,
          col: 2
        }
      },
      // 第4个模型数据源(一)
      {
        0: {
          row: 0,
          col: 0
        },
        1: {
          row: 0,
          col: 1
        },
        2: {
          row: 0,
          col: 2
        },
        3: {
          row: 0,
          col: 3
        }
      },
      // 第5个模型数据源(Z)
      {
        0: {
          row: 1,
          col: 1
        },
        1: {
          row: 1,
          col: 2
        },
        2: {
          row: 2,
          col: 2
        },
        3: {
          row: 2,
          col: 3
        }
      }

         ]
        
         /* 变量，当前使用的模型 */  
         var currentModel = {} 
         /* 变量，16宫格的位置 */
         var currentX = 0 , currentY = 0;
         
         /* 根据模型数据源来创建块元素 */         
         function createModel() {
             /* 判断满足游戏结束条件没 */
             if(isGameOver()){
                 ganmeOver();
                 return;
             }
             /* 先确定创建哪个模型 */
              currentModel = MODELS[_.random(0,MODELS.length -1)];
             /* 重新初始化16宫格的位置 */
            currentX=0;
            currentY=0;
             /* 来个随机颜色 */
              let color = new Array("#FF1493","#FF00FF","#0000FF","	#1E90FF","#00FFFF","#00FF7F","#00FF00","#FFFF00","#FF6600");
              let yanse = color[Math.floor(Math.random() * color.length)];
               /* 遍历对象，生成数量的块元素 */
              for(var key in currentModel){
                var divKuai = document.createElement("div");
                divKuai.className = "kuai";
/*                 divKuai.style.backgroundColor = color ; */
                divKuai.style.cssText = `box-shadow: inset 0 0 5px ${yanse},
                inset 0 0 10px ${yanse},
                inset 0 0 15px ${yanse};
                border: 1px solid ${yanse};`
                document.getElementById("container").appendChild(divKuai);
              }
              /* 定位 */
              locationBlocks();
              /* 自动下落 */
              autoDown();
         }

         /* 根据创建好的块元素给他定位位置，生成相应的模型 */
           function locationBlocks(){
             /* 判断越界 */
            checkBound();
               /* 得到当前所有块元素 */
               var blocks = document.getElementsByClassName("kuai");
               for(var i=0;i<blocks.length;i++ ){
                   /* 每个块元素 */
                   var oneBlock = blocks[i];
                   /* 得到每个块元素的坐标,16宫格位置加上本身相对16宫格位置 */
                   var locationKuai = currentModel[i];
                   oneBlock.style.top = (locationKuai.row+currentY) * STEP + "px";
                   oneBlock.style.left = (locationKuai.col+currentX) * STEP + "px";
               }
           }


        /*  入口方法 */
         function rukou (){
               onKeyDown();
               createModel();
         }


         /* 用户点击键盘事件 */       
        function onKeyDown(){
            document.addEventListener('keydown',function(event){
                 
                 switch(event.keyCode){
                     case 38: console.log("shang"); 
                     rotate();
                     break;
                     case 39: console.log("you"); 
                     move(1,0);
                     break;
                     case 40: console.log("xia");
                     move(0,1);
                      break;
                     case 37: console.log("zuo");
                     move(-1,0);
                      break;

                 }
            })
        }

        /* 移动函数 */       
        function move(x,y){
            /* 判断触碰，就是判断活动中的模型将要移动到的位置是否存在已经固定的模型（块元素） */
            if(isMeet(currentX + x , currentY + y , currentModel)){
                /* 若发生触碰，且由y轴引起，那么说明模型的底部发生触碰了，那么该停下了 */
                if(y!==0){
                    fixedBottomModel();
                }
                /* 直接return掉move方法，不再执行 */
                return;
            }
            /* 移动16宫格 */
            currentX = currentX + x;
            currentY = currentY + y;
            locationBlocks();
        }

         /* 旋转模型函数 */
         function rotate(){
             // 克隆一下 currentModel  用引入的js文件里的方法
             var cloneCurrentModel = _.cloneDeep(currentModel);
              /* 算法：移动后的行=移动前的列  移动后的列=3-移动前的行 */
              /* 得到当前使用的模型数据源里的块元素的坐标 */
              for(var key in cloneCurrentModel){
                    var blocksModel = cloneCurrentModel[key];
                    /* 实现算法进行变换 */
                    let temp = blocksModel.row;
                    blocksModel.row = blocksModel.col;
                    blocksModel.col = 3 - temp;
              }
                /* 判断触碰，就是判断活动中的模型将转的地方是否存在已经固定的模型（块元素） */
              if(isMeet(currentX,currentY,cloneCurrentModel)){
                  return;
              }
              /* 接收了这次旋转 */
              currentModel = cloneCurrentModel; 

              locationBlocks();
         }

        /* 控制模型只能在游戏区域里移动 */
        function checkBound(){
            /* 模型可活动的边界 */
            var leftBound = 0,rightBound = COL_COUNT,bottomBound = ROW_COUNT;
            /* 当块元素超出边界后，让16宫格退一步，这样一走一退相互抵消 */
             for(var key in currentModel){
                    var blocksModel = currentModel[key];
                    /* 左侧边界 */
                    if((blocksModel.col+currentX)<leftBound)
                    {
                        currentX++;
                    }
                    /* 右侧边界 */
                     if((blocksModel.col+currentX)>=rightBound){
                        currentX--;
                    }
                    /* 底侧边界 */
                    if((blocksModel.row+currentY)>=bottomBound){
                        currentY--;
                        /* 固定在底部 */
                        fixedBottomModel();
                    }
                                       
              }
        }

        /* 当模型到达底部时固定在底部 */
        function fixedBottomModel(){
            /* 1.改变模型颜色样式 */
            var activityModels = document.getElementsByClassName("kuai");
            /* 因为js特效，要从后往前遍历 */
            for(var i=activityModels.length-1;i>=0;i--){
                var activityModel = activityModels[i];
                 /* 让其不再移动，改变类名就行，这样位置函数将对其无效了 */
                activityModel.className = "ding";
                /* 把其放在记录块元素位置的变量中 */
                var blocksModel = currentModel[i];
                fixedBlocks[(currentY+blocksModel.row)+"_"+(currentX+blocksModel.col)]=activityModel;
               
            }
            /* 判断一行是否被铺满 */
              isRemoveLine();

            /* 创建新模型 */
            createModel();
        }

        /* 判断模型之间的触碰问题 */
        /* x，y为16宫格将要去到的位置，model表示当前模型将要完成的变化 */
            function isMeet(x,y,model){
                /* 判断触碰，就是判断活动中的模型将要移动到的位置是否存在已经固定的模型（块元素） */
                 /* 该位置是否存在块元素？ 只有一个return会生效*/
                 for(var k in model) {
                     var blockModel = model[k];
                     if(fixedBlocks[(y+blockModel.row)+"_"+(x+blockModel.col)]){
                         return true;
                     }
                 }
                 return false;
            }

            /* 判断一行是否被铺满 */
            function isRemoveLine(){
                /* 判断某行中，每一列是否都存在块元素，是则清理 */
                /* 遍历所有行所有列 */
                for(var i=0;i<ROW_COUNT;i++){
                    /* 假设当前行铺满，设flag=true */
                    var flag = true;
                    for(var j=0;j<COL_COUNT;j++){
                        /* 当前行没铺满 */
                         if(!fixedBlocks[i + "_" + j]){
                             flag = false;
                             break;
                         }
                    }
                    /* 当某行铺满,flag==true */
                    if(flag){
                          removeLine(i);
                    }
                }
            }

            /* 清理被铺满的一行 */
            function removeLine(line){
                /* 删除该行的块元素 */
                for(var i=0;i<COL_COUNT;i++){
                document.getElementById("container").removeChild(fixedBlocks[line+"_"+i]);
                  /* 数据源也删除 */
                 fixedBlocks[line+"_"+i] = null;
                }
                downLine(line);
                /* 加分数 */
                marks++;
                mark.innerHTML = "分数：" + marks ;

            }

            /* 被清理行上面的块元素下落 */
            function downLine(line){
                /* 遍历被清理行上的所有行 */
                 for(var i = line-1;i>=0;i--){
                     /* 该行的所有列 */
                    for(var j=0;j<COL_COUNT;j++){
                           if(!fixedBlocks[i+"_"+j]) continue;
                           /* 被清理行上面的块元素数据源所在行数+1 */
                           fixedBlocks[(i+1)+"_"+j] = fixedBlocks[i+"_"+j];
                           /* 让块元素下落 */
                           fixedBlocks[(i+1)+"_"+j].style.top = (i+1)*STEP+'px';
                           /* 清理之前块元素 */
                           fixedBlocks[i+"_"+j] = null;
                    }
                 }
            }

            /* 模型自动下落 */
            
            var mInterval = null;

            function autoDown(){
                if(mInterval){
                    clearInterval(mInterval);
                }
               mInterval = setInterval(function(){
                    move(0,1);
                },500)
            }

            /* 判断游戏结束 */
            function isGameOver(){
                /* 第0行也有元素时结束游戏 */
                for(var i=0;i<COL_COUNT;i++){
                    if(fixedBlocks["0_" + i]){
                        return true;
                    }
                }
                return false;
            }
            /* 结束游戏 */
            function ganmeOver(){
                /* 停止定时器 */
                if(mInterval){
                    clearInterval(mInterval);
                }
                /* 弹出对话框 */
                alert("游戏结束!   分数为："+marks+"。");
            }

            /* 点击再来一局 */
            again.addEventListener('click',function(){
                  
                /* 简单粗暴直接刷新好吧 */
                  location.reload(true);   
            })

     </script>

</body>
</html>
