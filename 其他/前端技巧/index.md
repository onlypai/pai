### 前端excel的导入和导出

#### 导出

**`responseType`** 是一个枚举字符串值，用于指定`响应中包含的数据类型`(服务器响应的数据类型)

* ' '：值为空字符串的responseType与默认类型 text 相同，
* text： [`DOMString`](https://developer.mozilla.org/zh-CN/docs/Web/API/DOMString) 对象中的文本,

* json：通过将接收到的数据内容解析为 JSON 而创建的 JavaScript 对象。    //axios中，responseType默认值为json，

* blob：包含二进制数据的 [`Blob`](https://developer.mozilla.org/zh-CN/docs/Web/API/Blob) 对象。
* arrayBuffer：包含二进制数据的 JavaScript [`ArrayBuffer`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer)。

> `Blob` 对象表示一个不可变、原始数据的类文件对象。它的数据可以按文本或二进制的格式进行读取，也可以转换成 [`ReadableStream`](https://developer.mozilla.org/zh-CN/docs/Web/API/ReadableStream) 来用于数据操作。 
>
> Blob 表示的不一定是JavaScript原生格式的数据。[`File`](https://developer.mozilla.org/zh-CN/docs/Web/API/File) 接口基于`Blob`，继承了 blob 的功能并将其扩展使其支持用户系统上的文件。

```js
//导出excel   api
export function exportExcel(params) {
    return request({
        url: "/ledger/exportExcel",
        method: 'get',
        params,
        responseType: 'blob'
    })
}
```

调用

```js
	toExcel() {
      exportExcel(params).then((res) => {
        console.log("excel", res);
        this.downloadfile(res);//下载
      });
    },

    downloadfile(res) {
      const fileName = "机电设备.xlsx";
      if ("download" in document.createElement("a")) {
        // 非IE下载
        const blob = new Blob([res], { type: "application/ms-excel" });
        const elink = document.createElement("a");
        elink.download = fileName;
        elink.style.display = "none";
        elink.href = URL.createObjectURL(blob);
        document.body.appendChild(elink);
        elink.click();
        URL.revokeObjectURL(elink.href); // 释放URL 对象
        document.body.removeChild(elink);
      }
    },
```

#### 导入

一般是调用接口导入到数据库，再请求数据进行展示

### 前端图片生成及打印

#### 生成

将responseType设置为blob

```js
//api
export function qrcode(code) {
    return request({
        url: '/common/qrcode/' + code,
        method: 'get',
        responseType: 'blob'
    })
}
//调用
generate() {
    qrcode(code).then((res) => {
        const blob = new Blob([res]);
        this.qrImg = window.URL.createObjectURL(blob);//生成图片src使用img元素展示
    });
},
```

#### 打印

根据后端返回的blob流，在线调取打印机，打印页面指定元素（区域）的内容，vue中可以使用插件  [vue-print-nb](https://www.npmjs.com/package/vue-print-nb)



### vue

#### vue滑屏

```vue
<template>
  <div class="fullPage" :style="{ height: screenHeight + 'px' }" ref="fullPage">
    <div
      class="fullPageContainer"
      ref="fullPageContainer"
      @mousewheel="mouseWheelHandle"
      @DOMMouseScroll="mouseWheelHandle"
    >
      <div class="section section1">1</div>
      <div class="section section2">2</div>
      <div class="section section3">3</div>
      <div class="section section4">4</div>
    </div>
    <ul type="circle" class="circle">
      <div
        @click="navTo(1, $event)"
        class="circleItem"
        :class="{ active: curIndex == 1 }"
      ></div>
      <div
        @click="navTo(2, $event)"
        class="circleItem"
        :class="{ active: curIndex == 2 }"
      ></div>
      <div
        @click="navTo(3, $event)"
        class="circleItem"
        :class="{ active: curIndex == 3 }"
      ></div>
      <div
        @click="navTo(4, $event)"
        class="circleItem"
        :class="{ active: curIndex == 4 }"
      ></div>
    </ul>
  </div>
</template>

<script>
export default {
  name: "Home",
  data() {
    return {
      screenHeight: 0, // 屏幕高度
      fullpage: {
        current: 1, // 当前的页面编号
        isScrolling: false, // 是否在滚动,是为了防止滚动多页，需要通过一个变量来控制是否滚动
        deltaY: 0, // 返回鼠标滚轮的垂直滚动量，保存的鼠标滚动事件的deleteY,用来判断是往下还是往上滚
      },
      curIndex: 1, // 当前页的index
    }
  },
  mounted() {
    this.screenHeight = document.documentElement.clientHeight
  },
  methods: {
    navTo(a) {
      console.log(a)
      this.fullpage.current = a
      this.move(a)
      this.curIndex = a
    },
    next() {
      // 往下切换
      let len = 4 // 页面的个数
      if (this.fullpage.current + 1 <= len) {
        // 如果当前页面编号+1 小于总个数，则可以执行向下滑动
        this.fullpage.current += 1 // 页面+1
        this.curIndex += 1
        this.move(this.fullpage.current) // 执行切换
      }
    },
    pre() {
      // 往上切换
      if (this.fullpage.current - 1 > 0) {
        // 如果当前页面编号-1 大于0，则可以执行向下滑动
        this.fullpage.current -= 1 // 页面+1
        this.curIndex -= 1
        this.move(this.fullpage.current) // 执行切换
      }
    },
    move(index) {
      this.fullpage.isScrolling = true // 为了防止滚动多页，需要通过一个变量来控制是否滚动
      this.directToMove(index) //执行滚动
      setTimeout(() => {
        //这里的动画是1s执行完，使用setTimeout延迟1s后解锁
        this.fullpage.isScrolling = false
      }, 1010)
    },
    directToMove(index) {
      let height = this.$refs["fullPage"].clientHeight //获取屏幕的宽度
      let scrollPage = this.$refs["fullPageContainer"] // 获取执行tarnsform的元素
      let scrollHeight // 计算滚动的告诉，是往上滚还往下滚
      scrollHeight = -(index - 1) * height + "px"
      scrollPage.style.transform = `translateY(${scrollHeight})`
      this.fullpage.current = index
    },
    mouseWheelHandle(event) {
      // 监听鼠标监听
      // 添加冒泡阻止
      let evt = event || window.event
      if (evt.stopPropagation) {
        evt.stopPropagation()
      } else {
        evt.returnValue = false
      }
      if (this.fullpage.isScrolling) {
        // 判断是否可以滚动
        return false
      }
      let e = event.originalEvent || event
      this.fullpage.deltaY = e.deltaY || e.detail // Firefox使用detail
      if (this.fullpage.deltaY > 0) {
        this.next()
      } else if (this.fullpage.deltaY < 0) {
        this.pre()
      }
    },
  },
}
</script>

<style scoped lang="scss">
.fullPage {
  height: 100%; //一定要设置，保证占满
  overflow: hidden; //一定要设置，多余的先隐藏
  background-color: rgb(189, 211, 186);
}
.fullPageContainer {
  width: 100%;
  height: 100%;
  transition: all linear 0.5s;
}
.section {
  width: 100%;
  height: 100%;
  background-position: center center;
  background-repeat: no-repeat;
}
.section1 {
  background-color: rgb(189, 211, 186);
}
.section2 {
  background-color: rgb(126, 182, 112);
}
.section3 {
  background-color: rgb(204, 134, 163);
}
.section4 {
  background-color: rgb(201, 202, 157);
}
.circle {
  position: fixed;
  right: 0;
  top: 420px;
  z-index: 100;
}
.circleItem {
  width: 7px;
  height: 7px;
  border-radius: 50%;
  background-color: #fff;

  margin: 20px;
}
.active {
  background-color: rgb(73, 129, 194);
}
</style>

```

#### 取消默认行为，阻止事件传递

添加对应事件配合修饰符使用

```vue
<template>
  <div @contextmenu.prevent="">
      <div @click.stop=""></div>
  </div>
</template>
```

#### node-sass报错

查看 node-sass 的版本是4.14.1，从 node-sass 的官方看到这个版本是不支持 node v16版本的，要从 node-sass 6.0.0 之后才开始支持。

尝试安装最新版本的 node-sass 报错依旧， node-gyp 环境问题造成的。

[gyp](https://gyp.gsrc.io/index.md) 是一个用来生成项目文件的工具，一开始是设计给 chromium 项目用的，生成项目文件后可以调用GCC、vsbuild、xcode等不通平台的编译工具进行编译。node 程序中需要调用一些其他语言编写的工具或dll，因此首先需要编译一下，否则会出现跨平台后无法正常使用的问题。

通过查看 node-sass 官方页面介绍，基本明确是当前使用的 node-sass 版本无法在高版本Node下正常运行，因此需要升级 node-sass。升级 node-sass 又依赖 node-gyp 的更高版本。因此，依次升级 node-gyp 、node-sass 后本地的问题就解决了。

#### vue中iframe打开html页面

> 需要嵌入的页面应该放到public文件下面的static文件夹中
>
> ```html
> <iframe src="/static/gantt/test.html" ref="iframe" width="100%" height="900px" scrolling="no"></iframe>
> ```

页面通信

一：父页面（调用iframe的页面）父页面中引入iframe

第一步：页面中引入iframe

```html
  <iframe
    id="PanIframe"
    ref="iframeRef"
    :frameBorder="0"
    :src="iframeURL"
  ></iframe>
```

第二步：父页面中监听iframe中发送过来的消息

```js
  mounted() {
    window.addEventListener('message', receiveMessageIframePage, false);
  },
  methods:{
    receiveMessageIframePage(event) {
      let iframes = document.getElementById("PanIframe") // 获取iframe元素
      // 如果监听到iframe暴露出一个type是getBoundFileKeys的消息，初始化给iframe发送一个boundFileKeys消息
      if (event.data.type === "getBoundFileKeys") {
          console.log(event.data.key)//aaasas
          //给iframe页面发送消息
          iframes.contentWindow.postMessage({ type: "boundFileKeys", data: "消息内容体" }, "*")
      }
      if (event.data.type === "check") {
        // 监听iframe暴露出来的check方法
      }
    }
  }
```

二：子页面（iframe页面）

发送消息给父页面获取初始化信息数据或其他业务

```js
window.parent.postMessage({
        type: 'getBoundFileKeys',
    	key:'aaasas'
}, '*');//第一个参数就是发出的数据
```

监听父页面发送过来的消息

```javascript

window.addEventListener('message', onMessage, false); // 接收接入方页面发送的消息
 
const onMessage = (e) => {
    if (!e || !e.data) { return; }
    switch (e.data.type) {
         case 'boundFileKeys':
            console.log(e.data.data)//消息内容体
            // TODO 处理自己的业务
            break;
     }
}
```

优化：判断是否处于iframe中

```js
getIsInIframe() {
    return window.self !== window.top;
}

getIsInIframe() && window.parent.postMessage({
        type: 'getBoundFileKeys',
    	key:'aaasas'
}, '*');
```

**总结：使用到的就是下面这几个方法**

```js

父页面监听iframe消息：
window.addEventListener('message', receiveMessageIframePage, false);
 
父页面发送给iframe页面的消息：
iframes.contentWindow.postMessage({ type: 'boundFileKeys', data: "消息内容体" }, '*');
 
 
 
iframe页面监听父页面发过来的消息：
window.addEventListener('message', onMessage, false); 
 
iframe页面给父页面发送消息：
window.parent.postMessage({ type: 'check',  data: "自定义需要发送的数据" }, '*')
```







### 饿了么

#### 修改部分样式

```css
// dialog
.el-dialog__header {
  background-color: rgb(34, 41, 55);
  display: flex;
}
.el-dialog__title {
  color: rgb(13, 191, 155);
}
.el-dialog__header::before {
  clear: both;
  content: "";
  display: block;
  width: 5px;
  height: 24px;
  background: rgb(13, 191, 155);
  margin-right: 10px;
}
.el-dialog__body {
  min-height: 200px;
  background-color: rgb(52, 57, 69);
}
```

#### 表格单（多）选（点击行实现）

```vue
<template>
    <!-- 表格 -->
    <div class="table">
        <el-table
                  ref="multipleTable"
                  @current-change="currentChange"
                  @selection-change="handleSelectionChange"
                  >
            <el-table-column type="selection" width="55"></el-table-column>
        </el-table>
</template>

<script>
export default {
  data() {
    return {
      dialogVisible: false,
      tableData: [],
      multipleSelection: [],
      isSingleSelect: false
    }
  },
  methods: {
    init(data = false) {
      this.isSingleSelect = data
      this.dialogVisible = true
    },
    handleClose() {
      this.dialogVisible = false
      this.isSingleSelect = false
      this.$refs.multipleTable.clearSelection()
    },
    handleSelectionChange(val) {
      if (this.isSingleSelect) {
        // 单选
        if (val.length > 1) {
          this.$refs.multipleTable.clearSelection()
          this.$refs.multipleTable.toggleRowSelection(val.pop())
        } else {
          this.multipleSelection = val.pop()
        }
      } else {
          //多选
        this.multipleSelection = val
      }
    },
      //点击行选中
    currentChange(currentRow, oldCurrentRow) {
      this.$refs.multipleTable.toggleRowSelection(currentRow)
    },

  },
}
</script>

```

#### 表格选择数据回显

```vue

          <el-table
            ref="multipleTable"
            :data="tableData"
            @current-change="currentChange"
            @selection-change="handleSelectionChange"
            row-key="id"//⭐
          >
            <el-table-column
              type="selection"
              width="55"
              :reserve-selection="true"//⭐
            ></el-table-column>
          </el-table>
<script>
export default {
  data() {
    return {
      tableData: [],
      multipleSelection: [],
    }
  },
  methods: {
    init(val) {
      val.forEach((row) => {//⭐，val就是需要回显的数据
        this.$refs.multipleTable.toggleRowSelection(row,true)
      })
    },
  }
}
</script>

```

#### el-upload组件，上传完成后隐藏上传按钮

```vue
<template>
	<el-upload
         :class="{ disUoloadSty:noneBtnImg }"
         :action="dealImgUrl"
         list-type="picture-card"
         :on-preview="handleDealImgPreview"
         :on-remove="handleDealImgRemove"
         :on-success="successDealImg"
         :before-upload="beforeUploadDealImg"
         :on-change="dealImgChange"
         :file-list="dealImgFileList"
         accept=".jpeg,.jpg,.gif,.png"
         :limit="limitCountImg"
        >
         <i class="el-icon-plus"></i>
    </el-upload>
</template>
<script>
	data(){
        return {
            noneBtnImg:false,//控制隐藏
            limitCountImg:1
        }
    },
    methods:{
        init(){
            //初始化样式为显示
            this.noneBtnImg = false
    	},
        //on-change
        dealImgChange(file, fileList){
            this.noneBtnImg = fileList.length >= this.limitCountImg;
	    },
        //on-remove
        handleDealImgRemove(file,fileList){
		   this.noneBtnImg = fileList.length >= this.limitCountImg;
	    }
    },
</script>
<style>
    ::v-deep .disUoloadSty .el-upload--picture-card{
      display:none;   /* 上传按钮隐藏 */
    }
</style>
```

#### 文件信息、图片转url、文件转base64编码

```vue
<template>
  <el-dialog
    :visible.sync="dialogVisible"
    @closed="handleClose"
    title="上传人员照片"
    width="25%"
    top="10vh"
    append-to-body
    :close-on-click-modal="false"
  >
    <div class="content">
      <div class="load_wrap">
        <img
          :src="faceImgUrl"
          width="370px"
          height="190px"
          v-if="faceImgUrl"
          alt=""
        />
        <span v-if="faceImgUrl" class="delete" @click="handleRemove()">
          <i class="el-icon-delete"></i>
        </span>
        <el-upload
          v-else
          class="upload-demo"
          drag
          action=""
          :on-change="handleChange"
          :auto-upload="false"
        >
          <div class="wrapper">
            <i class="el-icon-picture-outline"></i>
            <div class="el-upload__text">点击或将图片拖入</div>
          </div>
        </el-upload>
      </div>
      <footer class="dialog-footer">
        <el-button type="primary" @click="handleCommit">确 定</el-button>
        <el-button @click="handleClose">取 消</el-button>
      </footer>
    </div>
  </el-dialog>
</template>

<script>
import { addPhoto } from "@/api/accessCtrl/renyuanManage"
export default {
  name: "editDialog",
  data() {
    return {
      dialogVisible: false,
      faceImgUrl: "",
      data: {
        photoBase64: "",
        sysNo: "",
      },
    }
  },
  mounted() {},
  methods: {
    init(sysNo) {
      this.data.sysNo = sysNo
      this.dialogVisible = true
    },
    handleCommit() {
      if (!this.data.photoBase64) {
        this.$message({
          type: "warning",
          message: "您还未选择图片",
        })
        return
      }
      addPhoto(this.data).then((res) => {
        if (res.code === 10000) {
          this.$message({
            message: "上传成功",
            type: "success",
          })
          this.handleClose()
          this.$emit("changData")
        }
      })
    },
    handleClose() {
      this.dialogVisible = false
      this.data = {
        photoBase64: "",
        sysNo: "",
      }
      this.faceImgUrl = ""
    },
    // 选择图片⭐⭐
    handleChange(file) {
      console.log("file文件信息：", file.raw)
      const _this = this
      // 图片转url
      const blob = new Blob([file.raw])
      this.faceImgUrl = window.URL.createObjectURL(blob)

      //图片转base64
      let rd = new FileReader()
      rd.readAsDataURL(file.raw) // 文件读取转换为base64类型
      rd.onloadend = function (e) {
        //this指向当前方法onloadend的作用域, this.result就是文件的base64
        console.log(this.result)//base64
        //_this.data.photoBase64 = this.result.split(";base64,")[1]
      }
    },
    //移除
    handleRemove() {
      this.faceImgUrl = ""
      this.data.photoBase64 = ""
    },
  },
}
</script>

<style lang="scss" scoped>
.el-dialog__body {
  position: relative;
}
.content {
  width: 100%;
  height: 100%;
  padding-bottom: 50px;
  .load_wrap {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 40%;
    position: relative;
    img {
      border-radius: 10px;
    }
    .delete {
      position: absolute;
      left: 50%;
      top: 50%;
      transform: translate(-50%, -50%);
      font-size: 28px;
      cursor: pointer;
      &:hover {
        color: rgb(25, 137, 250);
      }
    }
  }
}
.el-upload {
  .wrapper {
    margin-top: 60px;
    i {
      margin-bottom: 10px;
      font-size: 27px;
    }
    .el-upload__text {
      font-size: 16px;
      font-family: Microsoft YaHei;
      font-weight: 400;
      color: #2e9af7;
    }
  }
}
::v-deep .el-upload-dragger {
  background-color: rgb(52, 57, 69) !important;
  border: 1px dashed rgb(77, 73, 73) !important;
}
.dialog-footer {
  position: absolute;
  bottom: 15px;
  right: 15px;
  left: 0;
  overflow: hidden;
  padding: 0;
  .el-button {
    float: right;
    margin-left: 15px;
    &:first-child {
      &:hover {
        color: #ccc;
      }
    }
  }
}
</style>

```

#### 饿了么表单输入数字、小数或负数

```html
//正整数       
    <el-input
              v-model="scope.row.value"
              oninput="value=value.replace(/[^\d]/g,'')"
              :maxlength="20"
              />
//小数或负数，保留2位小数
<el-input
                  v-model="scope.row.min"
                  oninput="if (value.slice(0,value.indexOf('-')+2)) {if(isNaN(value.slice(1,value.indexOf('-')+3))) { value = null }}
                              if(value.indexOf('.')>0){value=value.slice(0,value.indexOf('.')+3)}
                              if (value.slice(0,value.indexOf('-')+2)!=='-'){if(isNaN(value)){value=null}}"
                  :maxlength="20"
                />
```

#### ⭐el-dialog中包含el-tabs时关闭时浏览器卡死

关闭模态框时会造成浏览器卡死，这时将destroy-on-close设置为false  关闭时不销毁   解决问题



### 常见正则表达式

```js
const reg = /^[1-9]+\d*(\.\d{1,2})?$|^(0|0\.\d{1,2})$/     //非负，保留两位小数
const reg = /^1[3|4|5|7|8|9][0-9]\d{8}$/      //手机号
const reg = /(^(\d{3,4}-)?\d{7,8})$|(13[0-9]{9})/ //手机号及联系电话
const reg = /(ht|f)tp(s?)\:\/\/[0-9a-zA-Z]([-.\w]*[0-9a-zA-Z])*(:(0-9)*)*(\/?)([a-zA-Z0-9\-\.\?\,\'\/\\\+&amp;%\$#_]*)?/     //域名ip

```



### 插件

#### 树形控件[vue-treeselect](https://www.vue-treeselect.cn/)

#### 视频、音频解码器

官网：https://jsmpeg.com/

github:https://github.com/phoboslab/jsmpeg

官方例子:https://jsmpeg.com/perf.html

```js
window.JSMpeg={Player:null,VideoElement:null,...........}//源码修改
```

```js
<canvas id="video" ref="canvas"></canvas>

import JSMpeg from "@/assets/js/jsmpeg.min.js"


this.loading = true
var url = process.env.VUE_APP_WS_API + `/ws/ipc?deviceCode=${deviceCode}`
let _this = this
// console.log(canvas)
//节点未更新，获取不到canvas
this.$nextTick(() => {
    var canvas = this.$refs.canvas
    this.player = new JSMpeg.Player(url, {
        canvas: canvas,
        autoplay: true,
        // disableGl: true, //禁用 WebGL 并始终使用 Canvas2D 渲染器
        onSourceEstablished: function (source) {
            _this.loading = false
            console.log("成功")
        },
    })
})
```

#### 数字变化插件[countUp](http://inorganik.github.io/countUp.js/)

#### json编辑器

1.vue-json-editor

```shell
npm i vue-json-editor
```

```vue

<template>
  <div style="width: 70%;margin-left: 30px;margin-top: 30px;">
	<vue-json-editor
      v-model="resultInfo"  // 绑定数据resultInfo
      :showBtns="false"  // 是否显示保存按钮
      :mode="'code'"  // 默认编辑模式
      
      @json-change="onJsonChange"  // 数据改变事件
      @json-save="onJsonSave"  // 数据保存事件
      @has-error="onError"  // 数据错误事件
     />
    <br>
    <el-button type="primary" @click="checkJson">确定</el-button>
  </div>
</template>
 
<script>
  // 导入模块
  import vueJsonEditor from 'vue-json-editor'
 
  export default {
    // 注册组件
    components: { vueJsonEditor },
    data() {
      return {
        hasJsonFlag:true,  // json是否验证通过
        // json数据
        resultInfo: {
          'employees': [
            {
            'firstName': 'Bill',
            'lastName': 'Gates'
            },
            {
              'firstName': 'George',
              'lastName': 'Bush'
            },
            {
              'firstName': 'Thomas',
              'lastName': 'Carter'
            }
          ]
        }
      }
    },
    mounted: function() {
    },
    methods: {
      onJsonChange (value) {
        // console.log('更改value:', value);
        // 实时保存
        this.onJsonSave(value)
      },
      onJsonSave (value) {
        // console.log('保存value:', value);
        this.resultInfo = value
        this.hasJsonFlag = true
      },
      onError(value) {
        // console.log("json错误了value:", value);
        this.hasJsonFlag = false
      },
      // 检查json
      checkJson(){
        if (this.hasJsonFlag == false){
          // console.log("json验证失败")
          // this.$message.error("json验证失败")
          alert("json验证失败")
          return false
        } else {
          // console.log("json验证成功")
          // this.$message.success("json验证成功")
          alert("json验证成功")
          return true
        }
      },
    }
  }
</script>
```

2.bin-code-editor



