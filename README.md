# AutomaticPackaging-of-ipa
***自动化打包ipa****

* ![开始执行脚本](https://github.com/cuishengxi/AutomaticPackaging-of-ipa/blob/master/开始执行shell.png)
-----------------
*![结束-输入文件](https://github.com/cuishengxi/AutomaticPackaging-of-ipa/blob/master/结束shell.png)

--------
* 1.将脚本复制到工程的根目录
* 2.用代码编辑软件（比如Xcode）打开脚本，然后根据情况修改脚本内的一些参数
* 3.打开终端，输入 sh '打包脚本的全路径' <font color="#dd0000">打包脚本的全路径</font><br /> 就可执行打包脚本。



****脚本文件的内容
```
#!/bin/sh

# 使用方法:
# step1: 将该脚本放在工程的根目录下（跟.xcworkspace文件or .xcodeproj文件同目录）
# step2: 根据情况修改下面的参数
# step3: 打开终端，执行脚本。（输入sh，然后将脚本文件拉到终端，会生成文件路径，然后enter就可）

# =============项目自定义部分(自定义好下列参数后再执行该脚本)=================== #

# 是否编译工作空间 (例:若是用Cocopods管理的.xcworkspace项目,赋值true;用Xcode默认创建的.xcodeproj,赋值false)
is_workspace="true"

# .xcworkspace的名字，如果is_workspace为true，则必须填。否则可不填
workspace_name="DigitalBank"

# .xcodeproj的名字，如果is_workspace为true，则必须填。否则可不填
project_name="DigitalBank"

# 指定项目的scheme名称（也就是工程的target名称），必填
scheme_name="DigitalBank"

# 指定要打包编译的方式 : Release,Debug。一般用Release。必填
build_configuration="Release"

# method，打包的方式。方式分别为 development, ad-hoc, app-store, enterprise 。必填
method="development"


#  下面两个参数只是在手动指定Pofile文件的时候用到，如果使用Xcode自动管理Profile,直接留空就好
# (跟method对应的)mobileprovision文件名，需要先双击安装.mobileprovision文件.手动管理Profile时必填
mobileprovision_name=""

# 项目的bundleID，手动管理Profile时必填
bundle_identifier=""


echo "--------------------脚本配置参数检查--------------------"
echo "\033[33;1mis_workspace=${is_workspace} "
echo "workspace_name=${workspace_name}"
echo "project_name=${project_name}"
echo "scheme_name=${scheme_name}"
echo "build_configuration=${build_configuration}"
echo "bundle_identifier=${bundle_identifier}"
echo "method=${method}"
echo "mobileprovision_name=${mobileprovision_name} \033[0m"


# =======================脚本的一些固定参数定义(无特殊情况不用修改)====================== #

# 获取当前脚本所在目录
script_dir="$( cd "$( dirname "$0"  )" && pwd  )"
# 工程根目录
project_dir=$script_dir

# 时间
DATE=`date '+%Y%m%d_%H%M%S'`
# 指定输出导出文件夹路径
export_path="$project_dir/Package/$scheme_name-$DATE"
# 指定输出归档文件路径
export_archive_path="$export_path/$scheme_name.xcarchive"
# 指定输出ipa文件夹路径
export_ipa_path="$export_path"
# 指定输出ipa名称
ipa_name="${scheme_name}_${DATE}"
# 指定导出ipa包需要用到的plist配置文件的路径
export_options_plist_path="$project_dir/ExportOptions.plist"


echo "--------------------脚本固定参数检查--------------------"
echo "\033[33;1mproject_dir=${project_dir}"
echo "DATE=${DATE}"
echo "export_path=${export_path}"
echo "export_archive_path=${export_archive_path}"
echo "export_ipa_path=${export_ipa_path}"
echo "export_options_plist_path=${export_options_plist_path}"
echo "ipa_name=${ipa_name} \033[0m"

# =======================自动打包部分(无特殊情况不用修改)====================== #

echo "------------------------------------------------------"
echo "\033[32m开始构建项目  \033[0m"
# 进入项目工程目录
cd ${project_dir}

# 指定输出文件目录不存在则创建
if [ -d "$export_path" ] ; then
    echo $export_path
else
    mkdir -pv $export_path
fi

# 判断编译的项目类型是workspace还是project
if $is_workspace ; then
# 编译前清理工程
xcodebuild clean -workspace ${workspace_name}.xcworkspace \
                 -scheme ${scheme_name} \
                 -configuration ${build_configuration}

xcodebuild archive -workspace ${workspace_name}.xcworkspace \
                   -scheme ${scheme_name} \
                   -configuration ${build_configuration} \
                   -archivePath ${export_archive_path}
else
# 编译前清理工程
xcodebuild clean -project ${project_name}.xcodeproj \
                 -scheme ${scheme_name} \
                 -configuration ${build_configuration}

xcodebuild archive -project ${project_name}.xcodeproj \
                   -scheme ${scheme_name} \
                   -configuration ${build_configuration} \
                   -archivePath ${export_archive_path}
fi

#  检查是否构建成功
#  xcarchive 实际是一个文件夹不是一个文件所以使用 -d 判断
if [ -d "$export_archive_path" ] ; then
    echo "\033[32;1m项目构建成功 🚀 🚀 🚀  \033[0m"
else
    echo "\033[31;1m项目构建失败 😢 😢 😢  \033[0m"
    exit 1
fi
echo "------------------------------------------------------"

echo "\033[32m开始导出ipa文件 \033[0m"


# 先删除export_options_plist文件
if [ -f "$export_options_plist_path" ] ; then
    #echo "${export_options_plist_path}文件存在，进行删除"
    rm -f $export_options_plist_path
fi
# 根据参数生成export_options_plist文件
/usr/libexec/PlistBuddy -c  "Add :method String ${method}"  $export_options_plist_path
/usr/libexec/PlistBuddy -c  "Add :provisioningProfiles:"  $export_options_plist_path
/usr/libexec/PlistBuddy -c  "Add :provisioningProfiles:${bundle_identifier} String ${mobileprovision_name}"  $export_options_plist_path


xcodebuild  -exportArchive \
            -archivePath ${export_archive_path} \
            -exportPath ${export_ipa_path} \
            -exportOptionsPlist ${export_options_plist_path} \
            -allowProvisioningUpdates

# 检查ipa文件是否存在
if [ -f "$export_ipa_path/$scheme_name.ipa" ] ; then
    echo "\033[32;1mexportArchive ipa包成功,准备进行重命名\033[0m"
else
    echo "\033[31;1mexportArchive ipa包失败 😢 😢 😢     \033[0m"
    exit 1
fi

# 修改ipa文件名称
mv $export_ipa_path/$scheme_name.ipa $export_ipa_path/$ipa_name.ipa

# 检查文件是否存在
if [ -f "$export_ipa_path/$ipa_name.ipa" ] ; then
    echo "\033[32;1m导出 ${ipa_name}.ipa 包成功 🎉  🎉  🎉   \033[0m"
    open $export_path
else
    echo "\033[31;1m导出 ${ipa_name}.ipa 包失败 😢 😢 😢     \033[0m"
    exit 1
fi

# 删除export_options_plist文件（中间文件）
if [ -f "$export_options_plist_path" ] ; then
    #echo "${export_options_plist_path}文件存在，准备删除"
    rm -f $export_options_plist_path
fi

# 输出打包总用时
echo "\033[36;1m使用AutoPackageScript打包总用时: ${SECONDS}s \033[0m"

exit 0


```

-------

****iOS 应用的证书选择****  直通- [蒲公英](https://www.pgyer.com/doc/view/app_developer_account)

对于一个未上线 App Store 的应用，一般来说，开发者如果需要将应用安装到某些用户的设备上，就需要将应用导出为这些设备可以直接安装的安装包（.ipa文件），安装包能否正确导出，是决定了应用能否被正确安装到设备上的关键因素。其中，最关键的一个因素是，导出安装包时，应用所使用的证书（即：签名方式）。

开发者可以选择如下两种方式的证书签名方式，来导出应用安装包：

Ad-hoc 方式
In-house 方式
其中，具体使用哪种方式，取决于开发者拥有苹果开发者账号的类型。例如，如果开发者拥有的是苹果个人开发者账号，则可以使用 Ad-hoc 方式；如果拥有的是苹果企业开发者账号，则可以使用 In-house 方式。关于苹果开发者账号支持的证书类型，请见下表：

|账号类型	|价格	|可以发布AppStore?|	可以通过[蒲公英](https://www.pgyer.com/doc/view/app_developer_account)安装?	|支持安装设备数量|申请条件|证书类型|
|:---|:---:|---:|:---|:---:|---:|---:|
|个人账号	|$99|	可以	|可以	|100	|无限制	|Ad Hoc, App Store|
|公司账号	|$99	|可以	|可以	|100	|DUNS编码	|Ad Hoc, App Store|
|企业账号	|$299	|不可以	|可以	|无限制	|DUNS编码|	Ad Hoc, In House|
|教育账号	|$0|	可以	|可以	|100	|教育机构	|Ad Hoc, App Store|
[关于导出时，具体的操作方式，请查看：打包 iOS 的 IPA 文件](https://www.pgyer.com/doc/view/build_ipa)

三种证书签名的区别

到目前为止，苹果为 iOS 应用共提供了三种类型的证书签名方式，每一种都有独特的用途。这三种分别是：

Ad-hoc
In-house
App-Store
[蒲公英](https://www.pgyer.com/doc/view/app_developer_account)会根据打包证书的不同，分别显示为内测版、企业版、App-Store版。

关于这三种类型的证书，区别如下表所示：

|证书名称	|[蒲公英](https://www.pgyer.com/doc/view/app_developer_account)显示|	[蒲公英](https://www.pgyer.com/doc/view/app_developer_account)支持的安装范围|	支持的苹果开发者类型|
|:---|:---:|---:|:---|
|Ad-hoc	|内测版	|需要把设备UDID添加到证书才可安装	|个人账号、公司账号、教育账号、企业账号|
|In-house|	企业版	|任何iOS设备均可安装	|企业账号|
|App-Store|	App-Store|	只能通过App Store安装	|个人账号、公司账号、教育账号

*![结束-输入文件](https://github.com/cuishengxi/AutomaticPackaging-of-ipa/blob/master/屏幕快照%202018-10-30%20下午3.58.43.png)


其中，
ipa文件就是我们需要的安装包。
DistributionSummary.plist文件是一些详细的签名信息。
ExportOptions.plist文件其实就是我们在exportArchive命令时要用的，但在exportArchive之后会自动生成一个完整的文件。如果不知道该怎么写这个配置文件的话，可以直接参考我demo中生成的这个plist文件。
Packaging.log这个文件就是打包的时候产生的log了，可以查看日志记录。

至此，自动化打包已经完成了。
