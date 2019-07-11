# vue-upImg
这是一个用于PC端的相册上传的vue组件,可点击html页面直接查看效果。该项目也完成vue组件（Update.vue）,可直接用于项目中。

---
## 优点

在发送图片数据前,可以提前渲染预览,并删除,添加和拖拽。减少请求重复发送的次数。

---

## 功能实现

拖拽多个文件和input选择单个文件实现选中和预览。

鼠标放置在图片上会显示删除按钮。

如果是非图片文件，会被筛选，同时限制图片上传个数。

同时可以查看已经被选择的图片的总大小。

axios的方式提交formdata。



## 版本参考

```json
"axios": "^0.19.0",
"vue": "^2.5.2",
```
采用cdn外链引入到upImg.html文件中,具体项目请先安装好axios和后台数据接口在测试。


## 预览效果

![](https://raw.githubusercontent.com/qheyes/vue-upImg/master/img/update1.png)

![](https://raw.githubusercontent.com/qheyes/vue-upImg/master/img/update2.png)

## 功能主要代码

```javascript
  //遍历数组
  filesList(files){
    for (let i = 0; i < files.length; i++) {
      canupList=["image/gif","image/jpeg","image/png","image/x-icon"];
      if(canupList.indexOf(files[i].type)===-1){continue;}
      this.fileAdd(files[i]);
    }
  },
  //添加到渲染列表
  fileAdd(file){
    this.size = this.size + file.size;//总大小
    let reader = new FileReader();
    let that = this; 
    reader.readAsDataURL(file);
    reader.onload = function(){
      file.src = this.result;
      that.imgList.push({
        file
      });
    }
  },
  //单文件上传
  updateImg(el){
    if (!el.target.files[0].size) return;
    if(this.imgList.length >= this.maxNumber){
      alert("已经超出张数！！！")    
      return;
    }else{
      this.filesList(el.target.files);
    }     
  },
  //删除图片
  delImg(index){
    this.size = this.size - this.imgList[index].file.size;//总大小
    this.imgList.splice(index, 1);
  }

```

## 后台对接示例

```python
#上传相册接口（django rest framework）
class Add_album(APIView):
    def post(self,request):
        title = request.data.get("title",'')
        motif = request.data.get("motif",'').strip()
        length = request.data.get("length",'')
        if request.FILES.get("file0") is None:
            return Response({"msg": '上传失败 无文件'})
        if int(length)<1 or int(length)>9:
            return Response({"msg": '上传失败 文件个数不匹配'})
        try:
            imgurl_list=[]
            for i in range(int(length)):
                file_obj = request.FILES.get('file'+str(i))
                #给图片拼接静态目录路径，并去掉图片文件名中可能含有的百分号。
                imgurl = os.path.join('static', 'picture','img', str(int(time.time()+i))+file_obj.name.replace("%",""))
                imgurl_list.append(imgurl)
                f = open(os.path.join(BASE_DIR, imgurl), 'wb')
                for chunk in file_obj.chunks():
                    f.write(chunk)
                f.close()
        except:
            return Response({"msg": '储存过程失败 可以反馈给程序猿'})#一个chunk：2.5M
        else:
            imgurl_list_json = json.dumps(imgurl_list)
            models.Album.objects.create(title=title, imgurl=imgurl_list_json,imglen=length, motif=motif)
            return Response({"msg": '发表成功'})
```



##### 推荐联系方式

 暂无

