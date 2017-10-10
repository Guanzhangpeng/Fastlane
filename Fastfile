
fastlane_version "2.28.7"
default_platform :ios

#============================================================================================
#==============================编译\上传CocoaPods库=========================================================
platform :ios do
# 一 :该航道的描述
desc "编译\上传CocoaPods库"
# 定义航道 PushPodsLib:航道名称
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

# git pull
git_pull


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

end
#============================================================================================
#==============================发布到AppStore=========================================================
desc "发布到AppStore"
# 任务名称
lane :Release do

# 创建 ITC 中的 App 信息
produce(
#通常这两项是通过AppFile加载,因此这里不需要手动指定;
#username: "admin@sutongwang.cn",
#   app_identifier: "gzp.fastlane.test",
    app_name: "XXXXX",
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
    scheme: "xxxxx",
    export_method:'app-store',
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


#============================================================================================
#==============================打包IPA到蒲公英或者是FIR.im=========================================================
def archive(path, name, scheme, method)
    gym(
        export_method: method,
        scheme: scheme,
        clean: true,
        output_directory: path,
        output_name: name,
        configuration: 'Release',
        include_symbols: false,
        include_bitcode: true
    )
end
desc "打包 type 1: 正式, 0: 测试"
lane :archive do |options|
    time = Time.new.strftime("%Y-%m-%d-%H:%M:%S")
    path = "./fastlane/" + time
    type = options[:type]

    # 使用证书创建私钥及签名
    cert

    # 不带adhoc参数，sigh会自动生成App Store证书(公司或个人帐户)
     sigh(
        adhoc: true,
        force: true,
    )

    if type == '1'
        name = '正式'
        archive(path, name, 'TargetName', 'appstore')
    elsif type == '0'
        name = '测试'
        archive(path, name, 'TargetName', 'ad-hoc')
    # 上传至蒲公英
    #system 'ipa distribute:pgyer -f ./' + time + '/' + name + '.ipa -u your_user_key -a '
    # 上传至fir
    system 'fir publish ./' + time + '/' + name + '.ipa -T your_access_key'

    else
    puts "type错误, type == '0' : 测试, type== '1' : 正式"
    end
end


#============================================================================================
#==============================打包IPA到蒲公英或者是FIR.im======================================================


end
