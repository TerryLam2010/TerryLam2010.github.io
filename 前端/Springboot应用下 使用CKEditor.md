# Springboot应用下 使用CKEditor

因为在我的快速开发框架里需要增加内容发布相关的功能，所以需要使用富文本编辑器。比较了目前热门的一些富文本编辑器，最后选定了CKEditor4。CKEditor4的优点是功能强大、插件超多、文档详细、更新及时。

#### 引入CKEditor4

在官网下载CKEditor4，下载地址 <https://ckeditor.com/ckeditor-4/download/>，我选择的是Full Package版本。



CKEditor4版本选择

 

```
<!DOCTYPE html>
<html lang="zh-CN">
    <head>
        <meta charset="utf-8">
        <title>CKEditor Sample</title>
        <!-- 引入ckeditor.js文件 -->
        <script src="../ckeditor.js"></script>
    </head>
    <body>
        <form>
            <textarea name="editor1" id="editor1" rows="10" cols="80">
            </textarea>
            <script>
                // 替换 <textarea id="editor1">为CKEditor实例
                // 使用默认配置
                CKEDITOR.replace( 'editor1' );
            </script>
        </form>
    </body>
</html>
```

在浏览器中打开，效果如下

 

![img](//upload-images.jianshu.io/upload_images/10135025-cee03ea9705c2433.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/554/format/webp)

CKEditor4默认配置

 

获取编辑器文本，使用getData方法

```
CKEDITOR.instances.editor1.getData()
```

设置编辑器初始文本，使用setData方法

```
CKEDITOR.instances.editor1.setData( '<p>This is the editor data.</p>' );
```

#### 自定义CKEditor4工具栏

CKEditor的工具栏按钮可以根据需求灵活的隐藏、显示、分组、排序。
 新建一个CKEditor的自定义配置文件editorConfig.js。
 在下载的程序包里提供了自定义工具栏工具，目录是ckeditor\samples\toolbarconfigurator，在浏览器里打开index.html。

 

![img](//upload-images.jianshu.io/upload_images/10135025-c3781cfef6ccde51.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/554/format/webp)

自定义工具栏工具

 

配置好后，点击Get toolbar config，把生成的配置内容复制到editorConfig.js配置文件里。

```
CKEDITOR.editorConfig = function (config) {

    config.toolbarGroups = [
        {name: 'document', groups: ['mode', 'document', 'doctools']},
        {name: 'tools', groups: ['tools']},
        {name: 'clipboard', groups: ['clipboard', 'undo']},
        {name: 'editing', groups: ['find', 'selection', 'spellchecker', 'editing']},
        {name: 'forms', groups: ['forms']},
        {name: 'basicstyles', groups: ['basicstyles', 'cleanup']},
        {name: 'colors', groups: ['colors']},
        {name: 'styles', groups: ['styles']},
        {name: 'paragraph', groups: ['list', 'indent', 'blocks', 'align', 'bidi', 'paragraph']},
        {name: 'others', groups: ['others']},
        {name: 'about', groups: ['about']},
        {name: 'links', groups: ['links']},
        {name: 'insert', groups: ['insert']}
    ];

    config.removeButtons = 'About,Save,NewPage,Preview,Print,Templates,Find,Replace,SelectAll,Scayt,Form,Checkbox,Radio,TextField,Textarea,Select,Button,ImageButton,HiddenField,Language,BidiRtl,BidiLtr,Flash,Iframe,PageBreak,SpecialChar,Smiley,Cut,Copy,Paste,PasteText,PasteFromWord,CopyFormatting,RemoveFormat,Anchor,Styles,Format,Font,JustifyLeft,JustifyCenter,JustifyRight,JustifyBlock';
};
```

在html页面里引入自定义配置文件。

```
<!DOCTYPE html>
<html lang="zh-CN">
    <head>
        <meta charset="utf-8">
        <title>CKEditor Sample</title>
        <!-- 引入ckeditor.js文件 -->
        <script src="../ckeditor.js"></script>
    </head>
    <body>
        <form>
            <textarea name="editor1" id="editor1" rows="10" cols="80">
            </textarea>
            <script>
                // 使用自定义配置
                var editorConfig = {
                    customConfig: './samples/editorConfig.js'
                };

                CKEDITOR.replace( 'editor1', editorConfig);
            </script>
        </form>
    </body>
</html>
```

在浏览器中打开，效果如下

 

![img](//upload-images.jianshu.io/upload_images/10135025-9113c7498252a7c3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/554/format/webp)

自定义工具栏

#### 自定义CKEditor4上传图片工具

CKEditor4默认的上传图片功能界面不够简洁，很繁重。

![img](//upload-images.jianshu.io/upload_images/10135025-88a99e552b179daf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/491/format/webp)

默认上传图片界面

 

https://ckeditor.com/cke4/addon/image2

```
config.extraPlugins = 'image2';
```

在浏览器中显示效果如下，默认只支持通过url发布图片。

 

![img](//upload-images.jianshu.io/upload_images/10135025-2143e5e0c2d6181f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/249/format/webp)

Enhanced Image Plugin插件

 

需要添加本地图片上传功能。在editorConfig.js文件中增加如下配置：

```
// 服务器端上传图片接口URL
config.filebrowserImageUploadUrl='/cms/content/uploadImage';
```

在浏览器中显示效果如下

 

![img](//upload-images.jianshu.io/upload_images/10135025-8da2bb873f2681f4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/247/format/webp)

上传本地文件

 

editorConfig.js文件完整配置

```
// CKEDITOR配置文件
CKEDITOR.editorConfig = function (config) {
    config.language = 'zh-cn';

    config.height = 400;

    config.extraPlugins = 'image2';

    config.filebrowserImageUploadUrl='/cms/content/uploadImage';

    config.toolbarGroups = [
        {name: 'document', groups: ['mode', 'document', 'doctools']},
        {name: 'tools', groups: ['tools']},
        {name: 'clipboard', groups: ['clipboard', 'undo']},
        {name: 'editing', groups: ['find', 'selection', 'spellchecker', 'editing']},
        {name: 'forms', groups: ['forms']},
        {name: 'basicstyles', groups: ['basicstyles', 'cleanup']},
        {name: 'colors', groups: ['colors']},
        {name: 'styles', groups: ['styles']},
        {name: 'paragraph', groups: ['list', 'indent', 'blocks', 'align', 'bidi', 'paragraph']},
        {name: 'others', groups: ['others']},
        {name: 'about', groups: ['about']},
        {name: 'links', groups: ['links']},
        {name: 'insert', groups: ['insert']}
    ];

    config.removeButtons = 'About,Save,NewPage,Preview,Print,Templates,Find,Replace,SelectAll,Scayt,Form,Checkbox,Radio,TextField,Textarea,Select,Button,ImageButton,HiddenField,Language,BidiRtl,BidiLtr,Flash,Iframe,PageBreak,SpecialChar,Smiley,Cut,Copy,Paste,PasteText,PasteFromWord,CopyFormatting,RemoveFormat,Anchor,Styles,Format,Font,JustifyLeft,JustifyCenter,JustifyRight,JustifyBlock';
};
```

#### 服务器端代码

上传图片接口需要返回如下约定的JSON字符串。

```
//上传成功结果示例
{
    "uploaded": 1,
    "fileName": "foo.jpg",
    "url": "/files/foo.jpg"
}

//上传失败结果示例
{
    "uploaded": 0,
    "error": {
        "message": "The file is too big."
    }
}
```

上传图片接口响应模型定义如下：

```
public class UploadImageResModel {
    /**
     * 1成功，0失败
     */
    private Integer uploaded;

    private String fileName;

    private String url;

    public Integer getUploaded() {
        return uploaded;
    }

    public void setUploaded(Integer uploaded) {
        this.uploaded = uploaded;
    }

    public String getFileName() {
        return fileName;
    }

    public void setFileName(String fileName) {
        this.fileName = fileName;
    }

    public String getUrl() {
        return url;
    }

    public void setUrl(String url) {
        this.url = url;
    }
}
```

假设上传图片的根目录是E:/upload/。需要把该目录做静态资源映射，映射到/upload/**路由下。新增配置文件：

```
@Component
public class WebConfig extends WebMvcConfigurerAdapter {

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/upload/**")
                .addResourceLocations("file:///E:/upload/");

        super.addResourceHandlers(registry);
    }
}
```

上传图片接口代码如下

```
@Controller
@RequestMapping("/cms/content")
public class ContentController {

    private static final Logger logger = LoggerFactory.getLogger(ContentController.class);

    @PostMapping("/uploadImage")
    @ResponseBody
    public UploadImageResModel uploadImage(@RequestParam("upload") MultipartFile multipartFile) {
        UploadImageResModel res = new UploadImageResModel();
        res.setUploaded(0);

        if (multipartFile == null || multipartFile.isEmpty())
            return res;

        //生成新的文件名及存储位置
        String fileName = multipartFile.getOriginalFilename();
        String newFileName = UUID.randomUUID().toString()
                .replaceAll("-", "")
                .concat(fileName.substring(fileName.lastIndexOf(".")));

        String fullPath = "E:/upload/".concat(newFileName);

        try {
            File target = new File(fullPath);
            if (!target.getParentFile().exists()) { //判断文件父目录是否存在
                target.getParentFile().mkdirs();
            }

            multipartFile.transferTo(target);

            String imgUrl = "/upload/".concat(newFileName);

            res.setUploaded(1);
            res.setFileName(fileName);
            res.setUrl(imgUrl);
            return res;
        } catch (IOException ex) {
            logger.error("上传图片异常", ex);
        }

        return res;
    }
}
```

最后，CKEditor除了支持浏览本地图片的方式上传图片，还支持把图片拖拽到编辑器方式以及从剪贴板粘贴方式上传图片。

 

注意： 使用vue时，无法使用v-model 直接绑定，不要钻牛角尖。



附上各种参考URL

<https://www.twblogs.net/a/5b8dd2f62b7177188340da89> 

<https://ckeditor.com/ckeditor-4/download/> 

config配置生成

<https://ckeditor.com/latest/samples/toolbarconfigurator/index.html#basic> 