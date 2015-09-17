# IPALane
#ipa打包、分发测试

##1.打包

###[xcodebuild](https://developer.apple.com/library/prerelease/mac/documentation/Darwin/Reference/ManPages/man1/xcodebuild.1.html)
Apple的命令行打包工具
#####下载[xcodebuild(Command Line Tools Package)](https://developer.apple.com/downloads/) 或 命令行执行`xcode-select --install`
#####使用举例：导出ipa包
* xcodebuild clean install
* xcodebuild -project marathon.xcodeproj -scheme marathon archive -archivePath ./
* xcodebuild -exportArchive -exportFormat IPA -archivePath ../marathon.xcarchive -exportPath ./

###[xctool](https://github.com/facebook/xctool)
* xctool是Apple的xcodebuild基础上的封装；操作更简便

###[shenzhen](https://github.com/nomad/shenzhen)
* 命令行生成、分发ipa包到各平台(iTunes Connect、fir.im等)
* 支持命令行方式查看ipa信息

###[gym](https://github.com/fastlane/gym)
* [Fastlane](https://github.com/KrauseFx/fastlane)的部分工具之一；可以替代[shenzhen](https://github.com/nomad/shenzhen)
* 创建打包有签名的ipa包，支持[TestFlight(iTunes Connect)](https://developer.apple.com/library/prerelease/ios/documentation/LanguagesUtilities/Conceptual/iTunesConnect_Guide/Chapters/BetaTestingTheApp.html#//apple_ref/doc/uid/TP40011225-CH35-SW2)方式分发测试

##2.分发平台

###[蒲公英](http://www.pgyer.com)
* 支持命令行分发ipa

###[fir.im](http://fir.im)
* 非企业账号安装app需要添加UDID
* 支持命令行生成、分发ipa

###[TestFlight(iTunes Connect)](https://developer.apple.com/library/prerelease/ios/documentation/LanguagesUtilities/Conceptual/iTunesConnect_Guide/Chapters/BetaTestingTheApp.html#//apple_ref/doc/uid/TP40011225-CH35-SW2)
* 内部测试者：最多25个
* 外部测试者：最多1000个
	* 不需要添加UDID，只需要提供邮箱 

##3.分发
###[Fastlane](https://github.com/KrauseFx/fastlane)
* 命令行打包、分发、邀请tester的工具链 
	* [deliver](https://github.com/KrauseFx/deliver) •	  [snapshot](https://github.com/KrauseFx/snapshot) •	  [frameit](https://github.com/KrauseFx/frameit) •	  [PEM](https://github.com/KrauseFx/PEM) •	  [sigh](https://github.com/KrauseFx/sigh) •	  [produce](https://github.com/KrauseFx/produce) •	  [cert](https://github.com/KrauseFx/cert) •	  [codes](https://github.com/KrauseFx/codes) •	  [spaceship](https://github.com/fastlane/spaceship) •	  [pilot](https://github.com/fastlane/pilot) •	  [boarding](https://github.com/fastlane/boarding) •
   	[gym](https://github.com/fastlane/gym)
	* deliver
		* 将ipa包命令行方式分发到iTunes Connect中
	* snapshot
		* 自动截图
	* spaceship
		* ...	
	* pilot
		* 管理builds和测试者
	* boarding
		* 邀请外部测试者				
		
* 支持TestFlight
* 支持多航线分发，可选择分发到不同平台：[蒲公英](http://www.pgyer.com)、[fir.im](http://fir.im)、[TestFlight(iTunes Connect)](https://developer.apple.com/library/prerelease/ios/documentation/LanguagesUtilities/Conceptual/iTunesConnect_Guide/Chapters/BetaTestingTheApp.html#//apple_ref/doc/uid/TP40011225-CH35-SW2)

![ipa](https://cloud.githubusercontent.com/assets/3256113/9937003/3edd1e40-5d91-11e5-907c-6bba3a95b4db.png)


##4.方案
###非TestFlight分发测试
* [shenzhen](https://github.com/nomad/shenzhen)
* [shenzhen](https://github.com/nomad/shenzhen) + [deliver](https://github.com/KrauseFx/deliver) 

###TestFlight分发测试
* [gym](https://github.com/fastlane/gym) + [deliver](https://github.com/KrauseFx/deliver) 
* [Fastlane](https://github.com/KrauseFx/fastlane)


##5.举例([Fastlane](https://github.com/KrauseFx/fastlane))
###5.1 安装以下工具
* [xcodebuild(Command Line Tools Package)](https://developer.apple.com/downloads/)
* [gym](https://github.com/fastlane/gym)
* [deliver](https://github.com/KrauseFx/deliver)
* [spaceship](https://github.com/fastlane/spaceship)
* [Fastlane](https://github.com/KrauseFx/fastlane)

###5.2 配置Xcode工程文件
* 选择TARGET下Build Settings中Code Signing选项
* Code Signing Identity 选为`iOS Distribution`;Provisioning Profile 选为`Automatic`,值填`$(PROFILE_UDID)`
* 参照[CodeSigning->Best Solution: Using environment variables](https://github.com/KrauseFx/fastlane/blob/master/docs/CodeSigning.md)

###5.3 运行[Fastlane](https://github.com/KrauseFx/fastlane)命令后，配置`Fastfile`
* lane beta(TestFlight) 配置如下
	
	```
 	lane :beta do
    sigh
    ENV["PROFILE_UDID"] = lane_context[SharedValues::SIGH_UDID]
    gym(scheme: "marathon")
    deliver(beta: true)    
  end
	```
* lane pgyer(蒲公英) 配置如下
	
	```
 	lane :pgyer do
    ENV["PROFILE_UDID"] = lane_context[SharedValues::SIGH_UDID]
    gym(scheme: "marathon")
    sh "../pgyer.sh"

  end
	```
		
* lane fir(fir.im) 配置如下
	
	```
 	lane :fir do
    ENV["PROFILE_UDID"] = lane_context[SharedValues::SIGH_UDID]
    gym(scheme: "marathon")
    sh "../fir.sh"

  end
	```	

* 参照[CodeSigning->Best Solution: Using environment variables](https://github.com/KrauseFx/fastlane/blob/master/docs/CodeSigning.md)

###5.4 配置渠道脚本

####pgyer.sh([蒲公英](http://www.pgyer.com))

```	
	pwd
	#切换到目标目录
	cd ..
	pwd
	
	#替换ipa文件名
	curl -F "file=@{$filePath}" \
	#替换uKey（User Key）和_api_key（API Key）
	-F "uKey={$uKey}" \
	-F "_api_key={$apiKey}" \
	http://www.pgyer.com/apiv1/app/upload
    
```

其中：

* {$filePath}是应用安装包文件的路径
* {$uKey}是开发者的用户 Key，在应用管理-API中查看
* {$apiKey}是开发者的 API Key，在应用管理-API中查看

见[蒲公英](http://www.pgyer.com/doc/view/upload_one_command)

####fir.sh([fir.im](http://fir.im))

```	
	pwd
	#切换到目标目录
	cd ..
	pwd
	
	#发布应用到fir.im(配置生成的ipa名和fir.im API  Token)
	fir p path/to/application -T YOUR_FIR_TOKEN
    
```
见[fir-cli](http://blog.fir.im/fir_cli/)

###5.5 工程目录结构

* projectName	
	* framework
	* projectName	
	* projectName.xcodeproj 
	* projectNameTests
	* fastlane
		* Appfile
		* Deliverfile
		* Fastfile
	* fir.sh
	* pgyer.sh

##6.常见问题
###6.1 Ruby类库安装不了？选择[RubyGems 镜像(淘宝网)](http://ruby.taobao.org)
###6.2 fastlane配置文件中带自定义脚本，运行命令时出现` Permission denied - ../xx.sh`
* 是脚本文件权限不够，修改脚本权限：`sudo chmod a+x  ./xx.sh` 

###TODO
* deliver/fastlane分发ipa到iTunes Connect不支持TestFlight?
	* fastlane已支持TestFlight 
* 管理provision