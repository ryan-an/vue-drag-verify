# vue-drag-verify
* 基于vue-drag-verify二次开发的vue组件   
* 单位计算改成rem自适应
* 自动适配浏览器缩放、不同分辨率屏幕  
##### 使用方法
1.引入包  

```
  import dragVerify from "@/components/dragVerify";
  components{
     dragVerify
  }
```

2.引入以下代码
```
<drag-verify 
      :width="width" 
      :height="height" 
      :text="text" 
      :success-text="successText" 
      :background="background" 
      :progress-bar-bg="progressBarBg" 
      :completed-bg="completedBg" 
      :handler-bg="handlerBg" 
      :handler-icon="handlerIcon" 
      :text-size="textSize" 
      :success-icon="successIcon"
      ref="Verify"
    >
</drag-verify>
```
```
data () {
    return {
      handlerIcon: "fa fa-angle-double-right",
      successIcon: "fa fa-check",
      background: "#cccccc",
      progressBarBg: "#4b0",
      completedBg: "#66cc66",
      handlerBg: "#fff",
      text: "请将滑块拖动到右侧",
      successText: "验证成功",
      width: 4.4, // 注意这里单位是rem
      height: 0.2, //单位rem
      textSize: "18px",
      isCircle:'true'
    }
  },
```
