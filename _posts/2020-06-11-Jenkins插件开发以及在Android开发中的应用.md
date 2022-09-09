## Jenkins插件开发以及在Android开发中的应用

### 背景

18年的时候，随着业务迭代速度加快，开发测试人员的增加，以及逐渐开始的多版本并行开发，导致在交付APK给测试同学的时候频繁发生了一些降低开发效率的问题

- 开发同学正在开发分支开发功能的时候，测试同学过来找你要另一个分支的测试包
  
- 测试过来找你要某某版本的测试包，而你本地又没有存档
  
- 你打完包发现环境错了需要重新打包
  
- 用微信发给产品运营同学的APK，由于微信自动改APK后缀，产品同学不知道怎么安装
  
- APK的大小超过了微信的50M文件发送上限
  
  ......
  

当这些问题一次次发生，我知道，我们需要一台 "打包机"

### 需求分析

我们需要一台机器能为我们自动化的输出APK，它需要做到：

- 打包完成后提供一个二维码，扫描二维码可以下载apk
- 将本次打包的相关产物保存起来，例如`*-mapping.txt`。一次编译可能需要保存多个文件
- 可以配置域名，选择打包分支，是否混淆，编译类型以及设置打包版本号
- 保存历史apk可供查询

### 方案调研

经过调研，我们发现业界对此类问题已有成熟方案，即 Jenkins，Jenkins是根据MIT许可发布的开源自动化服务器，它用来将构建,测试以及交付或部署软件有关的各种任务自动化。它包含一个插件系统，以及大量的相关插件。

我们需要开发一个插件来将APK上传到百度云或者其他云平台，然后获取链接调用开源工具生成二维码，最后将二维码图片上传到云端。

整个过程如下

1. 编译结束后根据配置的规则查找指定的文件
2. 上传文件到云端并获取下载链接
3. 根据下载链接生成二维码
4. 上传二维码图片到云端
5. 根据二维码图片以及其他信息设置编译后的描述信息

### 插件开发

#### jenkins插件系统

Jenkins定义了一些可[扩展点](https://www.jenkins.io/doc/developer/extensions/)，它们是对构建系统的一个方面进行建模的接口或抽象类。Jenkins允许插件贡献这些实现

常用扩展点：

| 我想要 | 扩展点 | 插件示例 |
| --- | --- | --- |
| 每次编译时做点什么 | [BuildStep](http://javadoc.jenkins-ci.org/hudson/tasks/BuildStep.html) |     |
| 在每个项目构建中记录一些统计信息 | [Recorder](http://javadoc.jenkins-ci.org/hudson/tasks/Recorder.html) | [DiscardBuildPublisher](https://wiki.jenkins-ci.org/display/JENKINS/Discard+Old+Build+plugin) |
| 构建完成后触发一些操作 | [Publisher](http://javadoc.jenkins-ci.org/hudson/tasks/Publisher.html) |     |
| 添加新的构建类型或操作 | [Builder](http://javadoc.jenkins-ci.org/hudson/tasks/Builder.html) |     |
| 触发构建 | [Trigger](http://javadoc.jenkins-ci.org/hudson/triggers/Trigger.html) | [Files Found Trigger](https://wiki.jenkins-ci.org/display/JENKINS/Files+Found+Trigger) |
| 添加登录到Jenkins的方法 | [SecurityRealm](http://javadoc.jenkins-ci.org/?hudson/model/Descriptor.html) | [Google Login Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Google+Login+Plugin) |
| 在jenkins项目页面左侧添加导航页 | [Action](https://javadoc.jenkins-ci.org/hudson/model/Action.html) |     |

以`Publisher`扩展点为起点，我们可以开始插件的开发

#### 创建插件

官方中文文档很详细，略过

[为插件开发做准备](https://www.jenkins.io/zh/doc/developer/tutorial/prepare/)

插件项目结构

```shell
.
├── pom.xml
├── main
│   ├── java
│   │   └── io
│   │       └── jenkins
│   │           └── plugins
│   │               └── sample
│   │                   ├── ApkUploaderPublisher.java
│   └── resources
│       └── io
│           └── jenkins
│               └── plugins
│                   └── sample
│                       ├── ApkUploaderPublisher
│                       │   ├── config.jelly
└── test
```

按照官方文档创建模版项目之后，下载依赖然后执行 `mvn hpi:run`会启动一个依赖中自带的jenkins。不需要手动安装jenkins

#### 添加UI界面

jenkins的UI框架叫 `Jelly`, 基础用法请看官方文档 [Basic guide to Jelly usage in Jenkins](https://wiki.jenkins.io/display/JENKINS/Basic+guide+to+Jelly+usage+in+Jenkins)

先上效果图

![image20200613163959540](https://raw.githubusercontent.com/erleizh/erleizh.github.io/master/screenshots/2020-06-11/image-20200613163959540.png)

图中主要包含了：

- 主文件查找规则，即需要生成二维码的apk文件
- 附件的查找规则以及排除规则 (混淆的映射文件以及其他需要保存的文件)
- 需要展示出来的属性，例如版本号，打包分支之类的属性
- logo文件的路径，支持变量替换
- 百度云的配置信息

布局文件如下：

`src/main/resources/io/jenkins/plugins/sample/ApkUploaderPublisher/config.jelly`

```xml
<?jelly escape-by-default='true'?>
<j:jelly xmlns:j="jelly:core" xmlns:st="jelly:stapler" xmlns:d="jelly:define" xmlns:l="/lib/layout" xmlns:t="/lib/hudson" xmlns:f="/lib/form">
    <f:entry title="MainFile" description="需要生成二维码的文件的查找规则">
        <f:textbox field="mainFile" default="app/build/**/*.apk"/>
    </f:entry>
    <f:entry title="Includes" description="附件的查找规则">
        <f:textbox field="includes" default="app/build/outputs/**/*.txt,app/build/outputs/**/*.json"/>
    </f:entry>
    <f:entry title="Excludes" description="附件的排除规则">
        <f:textbox field="excludes"/>
    </f:entry>
    <f:advanced>
        <f:entry title="LogoPath" description="Logo 文件路径，用于嵌入二维码中,可以使用环境变量进行替换">
            <f:textbox field="logoPath"/>
        </f:entry>
        <f:entry title="编译的属性" description="需要展示出来的属性">
            <f:textarea field="properties" default="BUILD_ID=$${BUILD_ID}"/>
        </f:entry>
        <f:entry title="编译任务的名字" description="">
            <f:textarea field="buildName" default="#$${BUILD_TYPE}-$${BUILD_USER}"/>
        </f:entry>
    </f:advanced>
<!--    可以集成多个云，通过 radioBlock 来单选使用其中一个-->
<!--    <f:radioBlock title="七牛云" checked="${instance.isCheckedStorage('qiniu')}" name="storage"-->
<!--                  value="qiniu" inline="true">-->
<!--    </f:radioBlock>-->
    <f:radioBlock title="百度云" checked="${instance.isCheckedStorage('baidu')}" name="storage"
                  value="baidu" inline="true">
        <f:nested>
            <f:entry title="bucketName" field="bucketName"><f:textbox/></f:entry>
            <f:entry title="accessKey" field="accessKey"><f:textbox/></f:entry>
            <f:entry title="secretKey" field="secretKey"><f:textbox/></f:entry>
            <f:advanced>
                <f:entry title="endpoint" field="endpoint"><f:textbox/></f:entry>
                <f:entry title="存储路径，默认 /jenkins/" field="targetPath"><f:textbox/></f:entry>
            </f:advanced>
        </f:nested>
    </f:radioBlock>
</j:jelly>
```

- 在`jelly`文件中，每一个`field`属性都可以通过`@DataBoundSetter,@DataBoundConstructor`注解注入到对应字段中
- 如果需要检查 `field`是否输入正确，可以在`DescriptorImpl`中添加以`doCheck`加字段名的方法进行参数合法性检查
- 可以通过`${instance.isCheckedStorage('baidu')}`调用对应java类的方法, `instance`是一个预定义对象。其他预定义对象请查看官方文档

`src/main/java/io/jenkins/plugins/sample/ApkUploaderPublisher.java`

```java
public class ApkUploaderPublisher extends Recorder {

        //.... 省略部分代码

    public String isCheckedStorage(String storage) {
        return getStorage().equalsIgnoreCase(storage) ? "true" : "";
    }

    @DataBoundSetter
    public void setEndpoint(String endpoint) {
        config.setEndpoint(endpoint);
    }

    @DataBoundConstructor
    public ApkUploaderPublisher(String includes, String excludes, String mainFile, String logoPath, String properties, String targetPath, String buildName, String storage) {
        this.excludes = excludes;
           ....
        this.storage = storage != null && !storage.isEmpty() ? storage : "baidu";
    }

    @Extension
    public static final class DescriptorImpl extends BuildStepDescriptor<Publisher> {

        public FormValidation doCheckEndPoint(@QueryParameter String value)
                throws IOException, ServletException {
            return FormValidation.ok();
        }
        public FormValidation doCheckAccessKey(@QueryParameter String value)
                throws IOException, ServletException {
            if (value.length() == 0) return FormValidation.error("请设置 AccessKey");
            return FormValidation.ok();
        }
                ....

        @Override
        public boolean isApplicable(Class<? extends AbstractProject> aClass) {
            return true;
        }

        @Override
        public String getDisplayName() {
            return "Apk Uploader";
        }
    }
}
```

#### 文件上传功能

文件上传通过百度云的java sdk 实现

pom.xml 中添加百度云sdk依赖

```xml
<dependencies>
  <dependency>
    <groupId>com.baidubce</groupId>
    <artifactId>bce-java-sdk</artifactId>
    <version>0.10.101</version>
    <exclusions>
      <exclusion>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-lang3</artifactId>
      </exclusion>
      <exclusion>
        <groupId>commons-io</groupId>
        <artifactId>commons-io</artifactId>
      </exclusion>
      <exclusion>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-log4j12</artifactId>
      </exclusion>
      <exclusion>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-log4j12</artifactId>
      </exclusion>
    </exclusions>
  </dependency>
<dependencies>
```

为方便扩展，定义一个`CloudStorage`接口

```java
public interface CloudStorage {
    void upload(String src, String dest);
      void delete(String path);
      void download(String src, String dest);
      List<String> list(String dir);
      String getUrl(String file);
}
```

然后实现一个`BaiduCloudStorage`

```java
@Override
public void upload(String src, String dest) {
    File file = new File(src);
    if (!file.exists() || !file.canRead()) throw new IllegalStateException(src + " 文件不存在或者不可读！");
    PutObjectRequest request = new PutObjectRequest(bucketName, dest, file);
    client.putObject(request);
}

@Override
public String getUrl(String file) {
    URL url = client.generatePresignedUrl(bucketName, file, -1);
    return url.toString();
}
```

#### 二维码生成

使用`zxing`生成带logo的二维码图片

```xml
<dependencies>
    <dependency>
        <groupId>com.google.zxing</groupId>
        <artifactId>javase</artifactId>
      <version>3.3.0</version>
    </dependency>
<dependencies>
```

```java
public static void createQRCode(String url, int size, int margin, int logoSize, File logo, String desc) {
  QRCodeWriter writer = new QRCodeWriter();
  BitMatrix bitMatrix;
  try {
    FileOutputStream fos = new FileOutputStream(desc);
    // 根据指定的宽度创建一个二维码，内容为url，并移除白边
    bitMatrix = removeWhiteBorder(writer.encode(url, BarcodeFormat.QR_CODE, size, size, hints));
    MatrixToImageConfig config = new MatrixToImageConfig(MatrixToImageConfig.BLACK, MatrixToImageConfig.WHITE);
    BufferedImage qrImage = MatrixToImageWriter.toBufferedImage(bitMatrix, config);
    // 加载logo图片
    Image logoImage = ImageIO.read(logo).getScaledInstance(logoSize, logoSize, SCALE_DEFAULT);
    // 创建一张白色背景图
    BufferedImage combined = new BufferedImage(size, size, BufferedImage.TYPE_INT_ARGB);
    Graphics2D graphics = (Graphics2D) combined.getGraphics();
    graphics.setColor(Color.WHITE);
    graphics.fillRect(0, 0, size, size);
    graphics.setRenderingHint(RenderingHints.KEY_ANTIALIASING, RenderingHints.VALUE_ANTIALIAS_ON);
    //将二维码画到背景图上，并设置边距
    graphics.drawImage(qrImage.getScaledInstance(size - margin, size - margin, SCALE_DEFAULT), margin / 2, margin / 2, null);
    graphics.setComposite(AlphaComposite.getInstance(AlphaComposite.SRC_OVER, 1f));
    //画一个比logo稍大的白色圆角小方块作为logo的背景
    int logoBg = logoSize + margin * 2;
    graphics.fillRoundRect((size - logoBg) / 2, (size - logoBg) / 2, logoBg, logoBg, 20, 20);
    //将logo画到白色小方块之上
    graphics.drawImage(logoImage, Math.round((size - logoSize) / 2F), Math.round((size - logoSize) / 2F), null);
    //将最终数据写入指定的文件
    ImageIO.write(combined, "png", fos);
  } catch (Exception e) {
    e.printStackTrace();
  }
}
```

#### 文件扫描

使用`DirectoryScanner`来扫描文件

```java
/**
    * 查找符合条件的文件
    *
    * @param baseDir  路径
    * @param includes 排除的规则
    * @param excludes 需要查找的文件规则,多条用,分割
    * @return 文件列表
    */
@Nonnull
public static List<File> findFiles(String baseDir, String includes, String excludes) {
  if (includes == null || includes.isEmpty()) {
    return new ArrayList<>(0);
  }

  DirectoryScanner scanner = new DirectoryScanner();
  scanner.setBasedir(baseDir);
  scanner.setIncludes(includes.split(","));
  scanner.setExcludes(excludes.split(","));
  scanner.setCaseSensitive(true);
  scanner.scan();
  String[] uploadFiles = scanner.getIncludedFiles();

  if (uploadFiles == null || uploadFiles.length == 0) {
    return new ArrayList<>(0);
  }
  ArrayList<File> files = new ArrayList<>(uploadFiles.length);
  for (String uploadFile : uploadFiles) {
    files.add(new File(baseDir, uploadFile));
  }
  return files;
}
```

#### 生成描述信息

jenkins 每个build的描述都是可以自定义的，`Jenkins -> Configure Global Security -> Markup Formatter`选择 `Safe HTML`，然后通过`build.setDescription(descBuilder.build());`来设置编译描述

通过`hudson.Util#replaceMacro()`方法，可以直接将`分支名 : ${BRANCH}`形式的字符串通过环境变量替换成对应的值，这样就很方便的通过jenkins 插件配置中在描述信息增加属性值，而不必更改代码

由于html比较简单，直接通过文本格式化的方式创建。

最终效果如下图所示

![image20200613234048328](https://raw.githubusercontent.com/erleizh/erleizh.github.io/master/screenshots/2020-06-11/image-20200613234048328.png)

#### 集成

整个插件以`perform`为入口，所有的逻辑都在这里开始

```java
public class ApkUploaderPublisher extends Recorder {

  ....

    @Override
  public boolean perform(AbstractBuild<?, ?> build, Launcher launcher, BuildListener listener){
    try {
      CloudStorage storage = createStorage(build);
      DescribeBuilder descBuilder = new DescribeBuilder();
      //上传文件并生成二维码，然后将二维码上传
      handleQRCode(build, storage, descBuilder);
      //上传附件列表，并获取链接
      handleAttachments(build, storage, descBuilder);
      //处理用户配置的属性列表
      handleProperties(build, storage, descBuilder);
            //取出构建时填写的CHANGE_LOG,添加到描述信息
      descBuilder.setChangeLog(Util.replaceMacro("${CHANGE_LOG}", build.getEnvironment(listener)));
      //根据二维码，属性列表，附件生成描述网页
      build.setDescription(descBuilder.build());
      if (params.buildName != null && !params.buildName.isEmpty()) {
        //设置 DisplayName
        build.setDisplayName(Util.replaceMacro(params.buildName, build.getEnvironment(listener)));
      }
    } catch (Exception e) {
      e.printStackTrace();
    }
    return true;
  }
  ....
}
```

#### 调试

插件UI部分直接刷新浏览器就可以实时看到效果，但是java代码调试需要以下步骤：

1. 在`IntelliJ IDEA`中选中 `Run -> Edit Configurations -> Add New Configurations -> Remote `创建一个调试配置，端口号输入`8000`
  
  ![image20200614002320717](https://raw.githubusercontent.com/erleizh/erleizh.github.io/master/screenshots/2020-06-11/image-20200614002320717.png)
  
2. 在项目路径执行`mvnDebug hpi:run`
  
  ```shell
  ➜  apk-uploader git:(master) ✗ mvnDebug hpi:run 
  Preparing to execute Maven in debug mode
  Listening for transport dt_socket at address: 8000
  ```
  
3. 在`ApkUploaderPublisher#perform`方法添加断点，然后点击 `Debug`按钮
  
4. 打开浏览器，访问`http://localhost:8080/jenkins/` 创建一个项目，jenkins 提供了多种项目类型，这里我们选用`Freestyle Project`即可，然后输入项目名，点击 `OK` 项目就创建好了
  
5. 打开该项目的`configure`页面，在`Post-build Actions`中选中我们刚刚创建的插件
  
6. 点击 `立即构建` 按钮，可以看到程序断在了`perform`中
  

#### 生成插件

开发完毕之后就可以生成 .hpi 文件了。通过`mvn package`命令即可打包，文件保存在项目路径`./target/apk-uploader.hpi`

插件实际上只是一个遵循某些约定的jar文件

```
apk-uploader.hpi
 +- META-INF
 |   +- MANIFEST.MF
 +- WEB-INF
 |   +- classes
 |   +- lib
 +- (static resources)
```

### 部署

插件开发完毕后，我们需要部署到云服务器上。找运维同学申请资源之后就可以开始部署打包机了

#### 1.安装jenkins

首先需要在服务器上安装jenkins，官方文档已经很详细了，不在赘述

[安装jenkins](https://www.jenkins.io/zh/doc/book/installing/)

#### 2.创建项目

`Jenkins -> New Item`

jenkins 提供了多种项目类型，这里我们选用`Freestyle Project`即可，然后输入项目名，点击 `OK` 项目就创建好了

#### 3.安装插件

在经过调研之后，我们选用了以下第三方插件来实现需求

- [GIt](https://plugins.jenkins.io/git/) 拉取项目源码
- [Build Timestamp Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Build+Timestamp+Plugin)将时间戳注入到环境变量
- [list-git-branches-parameter](https://plugins.jenkins.io/list-git-branches-parameter/) 列出分支名作为构建参数
- [Environment Injector Plugin](https://wiki.jenkins.io/display/JENKINS/EnvInject+Plugin)注入环境变量到jenkins
- [Gradle Plugin](https://wiki.jenkins.io/display/JENKINS/Gradle+Plugin) 使jenkins支持gradle构建
- [user build vars plugin](http://wiki.jenkins-ci.org/display/JENKINS/Build+User+Vars+Plugin) 将发起此次build 的用户相关信息注入jenkins 环境变量

在jenkins插件管理页面直接搜索安装，然后重启

还有我们自己开发的 `apk-uploader`插件，在`Plugin Manager -> advanced`中点击`Upload Plugin`上传刚才生成的hpi文件。

#### 4.配置项目

项目配置分为五大类

- General
  
  此处需要开启`参数化构建`选项，以便将一些常见配置作为变量注入到环境变量中，然后在jenkins插件或项目的gradle脚本中使用
  
  例如 域名，是否混淆，是否加固，编译类型，选择git分支等等
  
  ![image20200612105602499](https://raw.githubusercontent.com/erleizh/erleizh.github.io/master/screenshots/2020-06-11/image-20200612105602499.png)
  
- Source Code Management
  
  此处需要配置git的仓库地址，证书，以及打包分支，可以通过`${BRANCH}`获取参数化构建中配置的分支名
  
- Build Triggers
  
  这里可以配置提交代码的时候触发构建
  
- Build
  
  配置gradle插件，指定项目路径以及gradle task ，例如 ：`clean :app:assemble${BUILD_TYPE}`,
  
  `BUILD_TYPE`来自于参数化构建选项中的配置
  
- Post-build Actions
  
  在这里添加我们刚才创建的插件并添加配置
  
  此处还可以配置钉钉通知插件，将编译信息发送到钉钉群组
  

#### 5.部署完成

到这里，整个打包机就已经可以工作了，整个工作流程就是 在Jenkins上选择构建参数之后，点击开始构建，然后jenkins自动从git仓库拉取代码，执行gradle插件中指定的gradle task，编译完成后按顺序执行指定的Post-build Actions。例如 ：将apk上传到百度云，修改编译后描述信息将二维码展示出来，发送钉钉通知等

![image20200612110856246](https://raw.githubusercontent.com/erleizh/erleizh.github.io/master/screenshots/2020-06-11/image-20200612110856246.png)

![image20200612110649541](https://raw.githubusercontent.com/erleizh/erleizh.github.io/master/screenshots/2020-06-11/image-20200612110649541.png)

### 总结

本文介绍了jenkins的一种使用方式，并创建了一个jenkins插件，了解了jenkins插件相关知识

通过jenkins，提高了开发效率，使得开发人员从重复劳动中解脱出来。