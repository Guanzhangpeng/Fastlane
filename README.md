年前的时候就想把关于fastlane的东西整理一下,但是一直苦于没有时间(其实就是懒),原谅我这么直白. 

###背景

我们都知道将开发好的APP发布到`AppStore`的流程大概是:`编译->打包IPA上传->填写应用更新数据->等待iTunesConnect编译->选择版本发布`，整个过程大概需要30分钟左右(前提是公司网络还要好,说多了都是泪啊~),  悲催的是如果某一次`Build Version`忘记增加了，那这个流程还得再来一遍,对于经常不断迭代的产品,这更是痛啊~~

还有一种情况,发布一个Pod远程私有库或者是Pod公开库的流程大概是:
`增加Podspec中的版本号->pod install -> Git Commit代码->打标签Tag->推送Tag到Origin->执行pod lib lint命令进行库验证->执行pod repo push命令发布库到私有仓库或者pod trunk push到CocoaPod公开库中`,整个过程虽然没有任何技术含量,但是整个过程下来半个小时没了,如果中间标签打错了,或者是标签忘记推送到远端了,~囧~,整个过程还得再来一遍!!!

那么有没有什么工具或者好的轮子可以解决上述问题呢?肯定是有的,下面我们就来聊聊今天的主角:`Fastlane`


![](http://upload-images.jianshu.io/upload_images/1154433-021cc49e7c5ef341.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###fastlane 简介

Fastlane 是一个完全开源的项目,是一款为 `iOS` 和` Android` 开发者提供的自动化构建工具，它可以帮助开发者将 `App` 打包、签名、测试、发布、信息整理、提交 `App Store` 等工作完整的连接起来，实现完全自动化的工作流，如果使用得当，可以显著的提高开发者的开发效率。
下面是fastlane的`GitHub`的链接和官方地址;

[GitHub链接](https://github.com/fastlane/fastlane) 

[官方文档](https://docs.fastlane.tools)

###fastlane 的使用

安装前请确保你已经安装了最新的 `Xcode command line tools`，通过在终端中执行下面的命令你可以进行检查。

```
xcode-select --install
```
安装 fastlane

```
sudo gem install -n /usr/local/bin fastlane
```
查看版本 
![1](http://upload-images.jianshu.io/upload_images/1154433-807a62f8b6cee03e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


为项目配置fastlane

```
$ cd 项目根目录

$ fastlane init
```
如果期间报错 `Connection reset by peer - SSL_Connect`,就需要执行:

```
$ brew update && brew install ruby
// 重装
$ sudo gem install -n /usr/local/bin fastlane
```
然后重新执行

```
$ fastlane init
```

在这期间会让你输入 `Apple ID` 账号密码,这个信息会存储在钥匙串中,后续使用无需再输入密码
会检测当前的 `app identifier` 是否在 `Apple Dev Center` 中
会检测当前 app 是否在 `iTunes Connect` 中
如果已经在 `ADC` 和 `ITC` 中创建相应的信息,那么过程会很顺利,如下图:


![](http://upload-images.jianshu.io/upload_images/1154433-36d8ce5998787ed7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

执行完成之后会在项目目录中生成如下文件结构:
![](http://upload-images.jianshu.io/upload_images/1154433-9594663ad54c9c8e.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

但是如果我们没有在`iTunes Connect`中创建APP,那么就不会创建上述`metadata`和`screenshots`两个文件夹;当然我们也可以后续创建,执行如下操作即可:

```
$ fastlane produce init
```

###上传 ipa 到蒲公英或者Fir.im上

####上传到蒲公英上

上传 ipa 的命令如下:

```
$ ipa distribute:pgyer -f path/to/ipa -u USER_KEY -a APP_KEY
```
其中`USER_KEY` 和` APP_KEY` 可以在蒲公英上查到。

####上传到Fir.im上

利用 `fir-cli` 将打好的包，通过命令，上传到Fir.im平台上
FIR.im CLI 使用 `ruby` 构建，只要安装相应 `ruby gem` 即可：

```
$ gem install fir-cli —verbose
```
上传 ipa 的命令如下:

```
$ fir publish path/to/ipa -T YOUR_FIR_TOKEN
```
其中 `YOUR_FIR_TOKEN` 可以在 `Fir.im` 上查到。

####完整 Fastfile 代码

```
fastlane_version "2.28.7"

default_platform :ios

def archive(path, name, scheme, method)

gym(
    export_method: method,
    scheme: scheme,
    clean: true,
    output_directory: path,
    output_name: name,
    configuration: 'Release',
    include_symbols: false,
    include_bitcode: true,
)

end

platform :ios do

desc "打包 type 1: 正式, 0: 测试"

lane :archive do |options|

time = Time.new.strftime("%Y-%m-%d-%H:%M:%S")

path = "./fastlane/" + time

type = options[:type]

# 如果你没有申请adhoc证书，sigh会自动帮你申请，并且添加到Xcode里
sigh(adhoc: true)

if type == '1'
    name = '正式'
    archive(path, name, 'yourTargetName', 'appstore')
elsif type == '0'
    name = '测试'
    archive(path, name, 'yourTargetName', 'ad-hoc')
# 上传至蒲公英
    system 'ipa distribute:pgyer -f ./' + time + '/' + name + '.ipa -u your_user_key -a your_app_key'
# 上传至fir
    system 'fir publish ./' + time + '/' + name + '.ipa -T your_token'
else
    puts "type错误, type == '0' : 测试, type== '1' : 正式"
end

end

end
```

CD 项目根目录，就可以通过 `fastlane archive type:0` 来调用打包脚本了,登录`Fir.im`和蒲公英平台上我们就可以看到刚刚发布上去的APP了:



![](http://upload-images.jianshu.io/upload_images/1154433-bfc6f46f3f4e019b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![](http://upload-images.jianshu.io/upload_images/1154433-3a7fcf98455fbc4d.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

####Sigh

如果你不确定证书目前是否可用，可以用`Sigh`自动生成获取证书。`Sigh`会自动根据`Appfile`里设置的`app_identifier`从`ADC(苹果开发者中心)`生成证书，并下载到项目根目录下(不是fastlane目录)，下载后自动安装。你可以通过指定`output_path`指定证书下载位置。

PS：建议不要把这个文件夹同步到项目的git中(`Fastlane`提供了`match`专门管理所有证书)。可以在`.gitignore`中忽略这个文件夹。

`sigh` 的具体用法可以参考[GitHub](https://github.com/fastlane/fastlane/tree/master/sigh) 或者[官方文档](https://docs.fastlane.tools/actions/#sigh)  
而且上面的两个链接我都是帮你们搜好了的,直接点击就可以到达你们想要的位置,不用谢我哈~~

####Gym

Gym 是 `Fastlane`家族的自动化编译工具，和其他工具配合的非常默契,它可以用来编译,打包iOS app,生成签名的ipa文件.下面是GitHub上的描述,我感觉更加贴切些

>gym is part of fastlane: The easiest way to automate beta deployments and releases for your iOS and Android apps.
>gym builds and packages iOS apps for you. It takes care of all the heavy lifting and makes it super easy to generate a signed ipa or app file 💪

>gym is a replacement for `shenzhen`.

具体用法参考[GitHub](https://github.com/fastlane/fastlane/tree/master/gym) 或者[官方文档](https://docs.fastlane.tools/actions/#gym)

####Deliver

我们可以使用`deliver`命令将屏幕截图、元数据和 `IPA` 文件上传到` iTunes Connect`中，

其实上传 ITC 最主要的文件是 `Deliverfile`,配置好` Deliverfile` 后,可以删除 `metadata `文件夹中的文本配置.因为`Deliverfile`的优先级比`metadata`要高:

>Values provided in the Deliverfile or Fastfile will be take priority over values from these files.

将`metadata`中的所有文件删除以后,将`1024*1024`的icon图标放入到该目录下,然后创建一个`分级`的json文件,这里假如叫做:`rating.json`,具体内容如下:

```
{
    //#0: None              无
    //#1: Infrequent/Mild   偶尔/轻微的
    //#2: Frequent/Intense  频繁的/强烈的
    //#卡通或幻想暴力
    "CARTOON_FANTASY_VIOLENCE": 0,
    //#现实暴力
    "REALISTIC_VIOLENCE": 0,
    //#大量露骨或残暴的现实暴力
    "PROLONGED_GRAPHIC_SADISTIC_REALISTIC_VIOLENCE": 0,
    //#低俗笑话
    "PROFANITY_CRUDE_HUMOR": 0,
    //#成人/性暗示题材
    "MATURE_SUGGESTIVE": 0,
    //#恐怖/惊悚题材
    "HORROR": 0,
    //#医学/医疗信息
    "MEDICAL_TREATMENT_INFO": 0,
    //#使用或提及烟、酒或毒品
    "ALCOHOL_TOBACCO_DRUGS": 0,
    //#模拟赌博
    "GAMBLING": 0,
    //#色情或裸露内容
    "SEXUAL_CONTENT_NUDITY": 0,
    //#色情及裸体画面
    "GRAPHIC_SEXUAL_CONTENT_NUDITY": 0,
    //#无限制的网站访问
    "UNRESTRICTED_WEB_ACCESS": 0,
    //#赌博和竞赛
    "GAMBLING_CONTESTS": 0
}
```

具体配置参加[官方文档](https://github.com/fastlane/fastlane/blob/master/deliver/Reference.md)

![222-w218](http://upload-images.jianshu.io/upload_images/1154433-fc530a549d31329d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


`Deliverfile`的具体配置可以参考:[Deliverfile文档](https://github.com/fastlane/fastlane/blob/master/deliver/Deliverfile.md)

###发布到App Store

先看下lane

```
desc "发布到AppStore"
# 任务名称
lane :Release do

# 创建 ITC 中的 App 信息
produce(
#通常这两项是通过AppFile加载,因此这里不需要手动指定;
    #username: "admin@sutongwang.cn",
    #app_identifier: "gzp.fastlane.test",
    app_name: "fastlane测试",
    language: "zh-Hans",
    app_version: "1.0",
    sku: "gzp.fstlane.test"
)
# 使用证书创建私钥及签名
    cert
# 每次运行时创建新的配置文件
    sigh(force: true)

# 指定输出目录
gym(
    output_directory: './build',
    configuration: 'Release',
    include_symbols: false,
    include_bitcode: true,
)
# 上传所有信息到App Store
deliver(
    force: true,
    #自动发布,false:手动发布
    #submit_for_review: true
)
end
```

配置完之后执行: `fastlane Release`

###发布到CocoaPods公开库或者是私有库


```
desc "编译\上传CocoaPods库"
lane :PushPodsLib do |options|
    targetName = options[:targetName]  #项目名称
    commitMsg = options[:commitMsg] #commit 日志信息
    tagName = options[:tagName]     #标签名称
    path    = "#{targetName}.podspec"

# 根据我们制定的自动化流程去写对应的action

# pod install
cocoapods(
    podfile: "./Example/Podfile"
)

# git add .
    git_add(path: ".")

# git commit -m "xxx"
    git_commit(path: ".", message: commitMsg)


# git push origin master
    push_to_git_remote

# 判断标签是否已经存在
# 如果存在, 应该删除标签(本地标签, 远程标签)
if git_tag_exists(tag: tagName)
UI.message("已经发现存在#{tagName}这个标签, 此处, 删除该标签对应的本地和远程标签")
remove_tag(tagName: tagName)

end

# git tag -a '#{tag}'
add_git_tag(
tag: tagName
)
# git push --tags
push_git_tags

# pod lib lint
pod_lib_lint(allow_warnings: true)

#通过trunk push到Cocoapods公开库中
# pod trunk push "#{targetName}.podspec"
pod_push(path: path,allow_warnings:true)

## You may also push to a private repo instead of Trunk 发布到私有库中
# pod repo push XMGFMSpecs "#{targetName}.podspec"
# pod_push(path: "#{targetName}.podspec", repo: repoName, allow_warnings:true)

```
其中`git_tag_exists`在`fastlane`的`Action`中并不存在的,所以需要我们自定义一个这样判断tag是否存在的Action,如果tag存在,则删除本地和远程的tag,具体是如何自定义的,可以参考我的[GitHub](https://github.com/Guanzhangpeng/Fastlane/blob/master/actions/remove_tag.rb)


###写在最后

我只想说码字真的很伤脑,尤其是想把一个东西讲明白就更烧脑

参考:
[GitHub](https://github.com/fastlane/fastlane)

[fastlane官方文档](https://docs.fastlane.tools)

[Fastlane 入门实战教程](https://github.com/mythkiven/AD_Fastlane)
