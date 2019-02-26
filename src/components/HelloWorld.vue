<template>
  <div class="hello">
    <div class="uploadIptBox" v-show="isHide == true">
      <input class="uploadIpt" type="file" @change="select($event)">
      <div class="uploadDom">选择图片</div>
    </div>
    <canvas id="imgCanvas" v-show="isCutted == false" @mousedown="cutImgBefore($event)" @mousemove="cutImgIng($event)" @mouseup="cutImgEnd($event)"></canvas>
    <canvas id="retCanvas"></canvas>
    <button type="button" class="commitBut" @click="uploadImg">Commit</button>
    <div class="progress">
        <div class="progress-item"></div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  data(){
    return {
      isHide:true,
      isCutted:false,
      imgData:"",
      flag : false,
      startX : 0,
      startY : 0,
      cutDataW:"",
      cutDataH:"",
      retImg:"",
      canvasImgd:"",
      initialWidth:0,
    }
  },
  methods:{
    select:function (e){
      this.isHide = false;
      this.imgData = e.target.files[0];
      let imgCanvas = document.getElementById("imgCanvas"),
          canvasImg = imgCanvas.getContext("2d");
      this.canvasImgd = canvasImg;

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
        localStorage.setH = imgHeight * (500/imgWidth);
        canvasImg.drawImage(img,0,0,500,localStorage.setH);

        var cutData = canvasImg.getImageData(0,0,500,localStorage.setH);
        this.cutDataW = cutData.width;
        this.cutDataH = cutData.height;
      }
      this.retImg = img;
    },
    cutImgBefore:function(e){
      this.flag = true;
      this.startX = e.clientX;
      this.startY = e.clientY;
    },
    cutImgIng:function(e){
      if(this.flag){
        let retCanvas = document.getElementById("retCanvas"),
            canvasRet = retCanvas.getContext("2d");
        this.canvasImgd.clearRect(0,0,500,localStorage.setH);
        canvasRet.clearRect(0,0,this.cutDataW,this.cutDataH);

        this.canvasImgd.drawImage(this.retImg,0,0,500,localStorage.setH);
        this.canvasImgd.fillStyle = 'rgba(0,0,0,0.5)';
        this.canvasImgd.fillRect(0,0,e.clientX,this.startY);
        this.canvasImgd.fillRect(e.clientX,0,500,e.clientY);
        this.canvasImgd.fillRect(this.startX,e.clientY,500-this.startX,localStorage.setH-e.clientY);
        this.canvasImgd.fillRect(0,this.startY,this.startX,localStorage.setH-this.startY);

        var canvasRetW = null,canvasRetH = null;
        if(e.clientX - this.startX < 0 || e.clientX - this.startX == 0){
          canvasRetW = 1;
        }else{
          canvasRetW = e.clientX - this.startX;
        }
        if(e.clientY - this.startY < 0 || e.clientY - this.startY == 0){
          canvasRetH = 1;
        }else{
          canvasRetH = e.clientY - this.startY;
        }
        retCanvas.width = canvasRetW;
        retCanvas.height = canvasRetH;
        var RetCutData = this.canvasImgd.getImageData(this.startX,this.startY,canvasRetW,canvasRetH);
        canvasRet.putImageData(RetCutData,0,0);
      }
    },
    cutImgEnd:function(){
      this.flag = false;
      this.isCutted = true;
      localStorage.removeItem("setH");
    },
    uploadImg:function(){
      if(this.imgData === ""){
        alert("请先选择图片");
      }else{
        var retCanvas    = document.getElementById("imgCanvas"),
            progressItem = document.querySelector(".progress-item"),
            progress     = document.querySelector(".progress"),
            self         = this,
            widthTimer = setInterval(function(){
                if(self.initialWidth < 300){
                    self.initialWidth += 3;
                    progressItem.style.width = self.initialWidth + 'px';
                }else{
                    clearInterval(widthTimer);
                }
            },100);
        retCanvas.toBlob(function(){
            var formData = new FormData(),
                httpDemo = new XMLHttpRequest();
            formData.append("imgnum",self.imgData);

            httpDemo.timeout = 10000;
            //请求完成后执行的操作
            httpDemo.onreadystatechange = function(){
                if(httpDemo.readyState == 4 && httpDemo.status == 200){
                    progress.style.display = "none";
                }
            };
            httpDemo.ontimeout = function () {
                clearInterval(widthTimer);
            }
            httpDemo.open("post","https://user.intechtrading.com/api/Overseas/setOpenAcountInfo.ashx?action=addressphoto&proxyid=160&usertoken=DiB4pC5S2kL2jzGK4dkGB8rej9PVlVICxhnLihj5HUC6u5dGfzEaOSDJbJUOk8PjTOxxfMQxqAOjYkn6R6GMfQgPAEHArsKewycYkCNff2Wz7MqCuTaKJw**&lang=en",true);
            httpDemo.send(formData);
        });
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
.commitBut{
  display: block;
  width: 200px;
  height: 40px;
  background-color: #2f7ce8;
  color: #fff;
  text-align: center;
  line-height: 40px;
  border: none;
  outline: none;
  cursor: pointer;
}
.progress {
  position: relative;
  height: 10px;
  width: 300px;
  border: 1px solid #3cf14a;
  border-radius: 5px;
}
.progress .progress-item {
  position: absolute;
  left: 0;
  top: 0;
  height: 100%;
  background: #77fb81;
  border-radius: 20px;
  transition: width .3s linear;
}
</style>
