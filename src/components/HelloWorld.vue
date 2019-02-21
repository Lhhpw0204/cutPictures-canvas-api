<template>
  <div class="hello">
    <div class="uploadIptBox" v-show="isHide == true">
      <input class="uploadIpt" type="file" @change="select($event)">
      <div class="uploadDom">选择图片</div>
    </div>
    <canvas id="imgCanvas"></canvas>
    <canvas id="retCanvas"></canvas>
    <p>{{resopnseData}}</p>
  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  data(){
    return {
      isHide:true,
      imgData:"",
      resopnseData:""
    }
  },
  methods:{
    select:function (e){
      this.isHide = false;
      this.imgData = e.target.files[0];
      var imgCanvas = document.getElementById("imgCanvas"),
          canvasImg = imgCanvas.getContext("2d");

      var img = new Image();
      var reader = new FileReader();
      reader.readAsDataURL(this.imgData);
      reader.onload = function (e) {
        img.crossOrigin = "anonymous";
        img.src = e.target.result;
      }

      img.onload = function(){
        var imgWidth  = img.width || img.naturalWidth,
            imgHeight = img.height || img.naturalHeight;
        imgCanvas.width = 500;
        imgCanvas.height = imgHeight * (500/imgWidth);
        canvasImg.drawImage(img,0,0,imgCanvas.width,imgCanvas.height);

        var retCanvas = document.getElementById("retCanvas"),
            canvasRet = retCanvas.getContext("2d"),
            cutData = canvasImg.getImageData(0,0,imgCanvas.width,imgCanvas.height),
            flag = false,
            startX = 0,
            startY = 0;
        imgCanvas.addEventListener("mousedown",function(e){
          flag = true;
          startX = e.clientX;
          startY = e.clientY;
        });
        imgCanvas.addEventListener("mousemove",function(e){
          if(flag){
            canvasImg.clearRect(0,0,imgCanvas.width,imgCanvas.height);
            canvasRet.clearRect(0,0,cutData.width,cutData.height);

            canvasImg.drawImage(img,0,0,imgCanvas.width,imgCanvas.height);
            canvasImg.fillStyle = 'rgba(0,0,0,.5)';
            canvasImg.fillRect(0,0,e.clientX,startY);
            canvasImg.fillRect(e.clientX,0,imgCanvas.width,e.clientY);
            canvasImg.fillRect(startX,e.clientY,imgCanvas.width-startX,imgCanvas.height-e.clientY);
            canvasImg.fillRect(0,startY,startX,imgCanvas.height-startY);

            var canvasRetW = null,canvasRetH = null;
            if(e.clientX - startX < 0 || e.clientX - startX == 0){
              canvasRetW = 1;
            }else{
              canvasRetW = e.clientX - startX;
            }
            if(e.clientY - startY < 0 || e.clientY - startY == 0){
              canvasRetH = 1;
            }else{
              canvasRetH = e.clientY - startY;
            }
            retCanvas.width = canvasRetW;
            retCanvas.height = canvasRetH;
            var RetCutData = canvasImg.getImageData(startX,startY,canvasRetW,canvasRetH);
            canvasRet.putImageData(RetCutData,0,0);
          }
        })
        imgCanvas.addEventListener("mouseup",function(){
          flag = false;
          retCanvas.toBlob(function(blob){
            var formData = new FormData();
            formData.append("imgnum",blob);
            var httpDemo =  new XMLHttpRequest();
            httpDemo.open("post","https://user.intechtrading.com/api/Overseas/setOpenAcountInfo.ashx?action=addressphoto&proxyid=160&usertoken=DiB4pC5S2kL2jzGK4dkGB8rej9PVlVICxhnLihj5HUC6u5dGfzEaOSDJbJUOk8PjTOxxfMQxqAOjYkn6R6GMfQgPAEHArsKewycYkCNff2Wz7MqCuTaKJw**&lang=en",true);
            httpDemo.onreadystatechange = function(){
              if(httpDemo.readyState == 4 && httpDemo.status == 200){
                var data = JSON.parse(httpDemo.responseText);
                this.resopnseData = data;
              }
            };
            httpDemo.send(formData);
          })
        })
      }
    }
  }
}
</script>

<style scoped>
.uploadIptBox{
  position: relative;
  width: 200px;
  height: 40px;
  cursor: pointer;
}
.uploadIpt{
  display: block;
  position: absolute;
  width: 100%;
  height: 100%;
  font-size: 30px;
  opacity: 0;
  cursor: pointer;
}
.uploadDom{
  width: 100%;
  height: 100%;
  background-color: #2f7ce8;
  color: #fff;
  text-align: center;
  line-height: 40px;
  cursor: pointer;
}
</style>
