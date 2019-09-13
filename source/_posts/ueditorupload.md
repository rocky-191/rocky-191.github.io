---
title: ueditor图片上传方式处理
date: 2019-09-13 18:59:42
tags:  
	- javascript
categories: "javascript"
---

最近项目里需要用到富文本编辑器，同事选择里百度出的ueditor，但是里面自带的图片上传功能需要后台配合，配置成服务器地址，和我们实际情况不是太符合，于是另想办法，搞定图片上传。

### 重写配置项
首先重写里toolbars配置。最重要的是要把原先的上传图片功能按钮去掉，下面是我用到的配置项

```
toolbars: [
          [
            "fullscreen",
            "source",
            "undo",
            "redo",
            "bold",
            "italic",
            "underline",
            "fontborder",
            "strikethrough",
            "superscript",
            "subscript",
            "removeformat",
            "formatmatch",
            "autotypeset",
            "blockquote",
            "pasteplain",
            "|",
            "forecolor",
            "backcolor",
            "insertorderedlist",
            "insertunorderedlist",
            "selectall",
            "cleardoc",
            "mergeright", //右合并单元格
            "mergedown", //下合并单元格
            "deleterow", //删除行
            "deletecol", //删除列
            "splittorows", //拆分成行
            "splittocols", //拆分成列
            "splittocells", //完全拆分单元格
            "deletecaption", //删除表格标题
            "inserttitle", //插入标题
            "mergecells", //合并多个单元格
            "deletetable", //删除表格
            "cleardoc", //清空文档
            "insertparagraphbeforetable", //"表格前插入行"
            "fontfamily", //字体
            "fontsize", //字号
            "paragraph", //段落格式
            "inserttable", //插入表格
            "edittable", //表格属性
            "edittd", //单元格属性
            "link" //超链接
          ]
        ]
```
更多配置可参考[官网](http://fex.baidu.com/ueditor/#start-toolbar)

### 添加初始化方法
初始化ueditor的时候触发一个方法，因为我的项目是用vue写的，而且封装了一层ueditor,所以就对外暴露了一个beforeInit方法。

```
<fw-ueditor-wrap
  v-model="mainBody"
  :config="myConfig"
  @beforeInit="addCustomDialog"
  :key="1"
></fw-ueditor-wrap>
```

```
// 添加自定义弹窗
    addCustomDialog(editorId) {
      let that = this;
      window.UE.registerUI(
        "test-dialog",
        function(editor, uiName) {
          // 参考http://ueditor.baidu.com/doc/#COMMAND.LIST
          var btn = new window.UE.ui.Button({
            name: "dialog-button",
            title: "上传图片",
            cssRules: `background-image: url('/image/upload.png') !important;background-size: cover;`,
            onclick: function() {
              // 渲染dialog
              that.dialogVisible = true;
              editor.execCommand(uiName);
            }
          });

          return btn;
        },
        100 /* 指定添加到工具栏上的那个位置，默认时追加到最后 */,
        editorId /* 指定这个UI是哪个编辑器实例上的，默认是页面上所有的编辑器都会添加这个按钮 */
      );
    }
```
其实就是在toolbar工具栏后面又加了一个自定义的按钮，实现上传功能。
### element弹窗设置
弹窗我用的是element的弹窗，使用方式参考element官网弹窗。并且使用了element的upload上传组件
```
<el-dialog
      title="上传图片"
      v-if="dialogVisible"
      :visible.sync="dialogVisible"
      width="30%"
    >
      <el-upload
        class="upload-demo"
        drag
        accept=".png, .jpg"
        :headers="headers"
        :action="uploadAddr"
        :beforeUpload="beforeAvatarUpload"
        :on-success="uploadImageSuccess"
        :on-error="uploadImageError"
      >
        <i class="el-icon-upload"></i>
        <div class="el-upload__text">
          将文件拖到此处，或
          <em>点击上传</em>
        </div>
        <div class="el-upload__tip" slot="tip">
          只能上传jpg/png文件，且不超过5M
        </div>
      </el-upload>
    </el-dialog>
```
关键的就是上传成功后需要触发uploadFile方法，将上传成功的图片插入到富文本编辑器中

```
uploadFile(file) {
//关键
      let editor = document.querySelector(".edui-default").getAttribute("id");
      window.UE.getEditor(editor).execCommand("insertimage", {
        src: file.url,
        width: "100",
        height: "100"
      });
      this.dialogVisible = false;
    },
    // eslint-disable-next-line no-unused-vars
    uploadImageSuccess(response, file, fileList) {
      if (response) {
        this.$message({
          message: "上传成功",
          type: "success"
        });
        let fileObj = {
          name: response.originalName,
          url: response.url
        };
        // this.fileList.push(fileObj);
        this.uploadFile(fileObj);
      } else {
        this.$message({
          message: "上传失败",
          type: "warning"
        });
      }
    }
```
大功告成，搞定！