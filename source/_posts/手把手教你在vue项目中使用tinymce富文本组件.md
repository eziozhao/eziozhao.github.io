---
title: 手把手教你在vue项目中使用tinymce富文本组件
date: 2020-09-12 12:30:04
tags:
- 富文本
categories:  
- 前端
---
## 安装

```bash
npm install tinymce
npm install @tinymce/tinymce-vue
```

## Demo

不想看的后面的直接复制走

1、封装好的richtext组件

```jsx
<template>
  <editor :id="tinymceId" :init="init">
  </editor>
</template>

<script>
import tinymce from 'tinymce/tinymce'
import Editor from '@tinymce/tinymce-vue'
import 'tinymce/themes/silver'
import 'tinymce/icons/default/icons.js'
import 'tinymce/plugins/image'

export default {
  name: 'RichText',
  props: {
    id: {
      type: String,
      default: function () {
        return 'vue-tinymce-' + +new Date() + ((Math.random() * 1000).toFixed(0) + '')
      }
    },
    value: {
      type: String,
      default: ''
    },
  },
  watch: {
    value (val) {
      if (!this.hasChange && this.hasInit) {
        this.$nextTick(() =>
          window.tinymce.get(this.tinymceId).setContent(val || ''))
      }
    }
  },
  data () {
    const _this = this
    return {
      tinymceId: this.id,
      hasInit: false,
      hasChange: false,
      init: {
        language_url: '/static/tinymce/zh_CN.js', // 语言包的路径
        language: 'zh_CN', // 语言
        skin_url: '/static/tinymce/skins/ui/oxide', // skin路径
        height: 300, // 编辑器高度
        toolbar: ' undo redo |  bold italic underline strikethrough image',
        branding: false, // 去水印
        elementpath: false, // 禁用编辑器底部的状态栏
        statusbar: false, // 隐藏编辑器底部的状态栏
        paste_data_images: true, // 允许粘贴图像
        menubar: false, // 隐藏最上方menu
        plugins: 'image', // 图片插件

        images_upload_handler: (blobInfo, success, failure) => {
          // 这里可以请求接口
          success('data:image/jpeg;base64,' + blobInfo.base64())
        }, // 上传本地图片
        init_instance_callback: editor => {
          if (_this.value) {
            editor.setContent(_this.value)
          }
          _this.hasInit = true
          // 监听富文本内容的改变 将变化传回richtext的v-model
          editor.on('NodeChange Change KeyUp SetContent', () => {
            this.hasChange = true
            this.$emit('input', editor.getContent())
          })
        }
      }
    }
  },
  components: {Editor},
  mounted () {
    tinymce.init({})
  }
}
</script>

<style scoped></style>
```

2、在父组件中调用 

```jsx
<template>
	<RichText  v-if="isShow" v-model="content" />
</template>
<script>
import RichText from '@/components/RichText' //记得改路径

export default {
	name: 'Father',
  components: { RichText },
	data(){
		return{
			isShow:false,
			content:'your content'
		}
	}
}
<script/>
<style scoped></style>
```

注意：v-model绑定的值会自动赋给RichText的props属性中的value。并且只在初始化时执行一次，如果你希望每次打开富文本编辑框时，都能获取到最新的content值，就要在关闭富文本组件时进行销毁。

我这里用了v-if来实现，父组件关闭时，将isShow设为false。这样下次打开父组件时，RichText 子组件会重新加载mounted中的 tinymce.init({}) 方法，从而获取最新的content

## 在vue项目新建组件

我这里命名为RichText，注意要在你的组件中import必要的文件。比如icons.js没有引入会导致所有的图标都显示not found

关于init中的属性说明：

1、skin_url是必填属性，没有的话会导致组件不能显示。需要到node_modules中找到tinymce对应的文件夹，将其中的skins文件夹复制出来。我这里复制到了 static/tinymce 中，然后在init中填写对应路径

![](https://imgchr.com/i/waAnII)

2、没有toolbar属性，组件会只显示一个文本框，所以该属性必填。

3、具备以上两个属性已经可以实现基本的富文本操作。其他属性的作用参考注释。

中文语言包可以到[官网](https://www.tiny.cloud/get-tiny/language-packages/)下载



## 插件的使用

官方提供了很多插件可供使用，可在init中添加plugins属性。

注意：使用了一个插件，就要在组件中import它。具体可使用的插件，可在node_modules的tinymce中的plugins中查看

![](https://imgchr.com/i/waAKit)

## 关于图片上传插件

官方提供的插件默认是直接插入图片的url，但是很多时候，我们希望能直接从本地选取图片。这时需要在init中加入一个images_upload_handler函数进行处理。本文的例子中没有与后端交互。如果需要将图片上传到后台，只需在images_upload_handler中请求接口，并在success中填上请求成功后返回的图片url即可
