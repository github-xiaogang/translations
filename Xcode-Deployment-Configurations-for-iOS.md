# iOS Xcode部署配置

几乎所有移动应用，包括我正在编写的一款iOS应用 [Emmerge][1] 都需要与服务器交互。从第一次向团队成员演示应用开始，我们就开始找一种更快构建不同环境产品的方法。一开始我们只需要修改不同环境下的服务器地址，不过经过几天不停的在配置文件中来回切换本地和远程服务器地址后，我彻底厌倦了，于是到Google查找解决的方法。搜索一段时间后，我却没用找到如何在Xcode中为每一种构建环境设置自己的配置文件的方法，于是在大量搜索stack overflow和博客后，我总结了一种相当简洁有效的方法，这里分享给需要的人。

## 我们什么时候需要不同的部署环境

首先，几乎所有需要服务器的应用至少需要配置服务器地址；又比如你的应用使用了第三方平台登录，如Facbook，Google等，并且在每种构建环境下可能又需要不同的第三方平台，那么就需要为不同构建环境设置不同的平台App IDs；又比如你的应用需要收集分析数据，那么你可能就需要配置不同的 [mixpanel][2] ID。

## 具体设置方法

以上面为不同环境配置服务器地址为例，假设我们Development和Release两个构建环境，我们的目标是为每种环境创建自己的配置文件(Setting.plist)，然后在应用运行时只加载相应环境的配置文件。

你可以下载 [Demo程序][3] 操作一遍。

1.创建Development和Release构建环境

在Project Navigator下，选择项目名，然后在下拉菜单中选择Project（注意不是Target），然后选中Info标签栏

![创建不同环境配置][4]

在Configurations下点击 "+" ，然后选择 Duplicate "Release" Configuration，然后重命名为"Production"

![创建Production配置][5]

然后和上面一样步骤复制"Debug"配置，并修改为"Development"

接着，删除默认的Debug和Release配置。

![创建Development配置][6]

(注意：如果你使用CocoaPods，看下面关于CocoaPods的注意项)

2.为两种构建环境分别创建配置文件"Settings-Development.plist" 和 "Settings-Production.plist"

![创建Development配置文件][7]

![创建Production配置文件][8]

3.现在你可以向配置文件中添加任意的配置项，但注意两个配置文件中的配置key要保持相同

![添加配置内容][9]

4.打开Project Navigator，选中项目名，然后在下拉菜单中选中Target（注意不是Project）

![设置Target][10]

在Build Phases区域，在Copy Bundle Resources区域移除Target对两个配置文件的引用

![移除引用][11]

5.添加一个Run Script Build Phase ，选择Editor -> Add Build Phase -> Add Run Script Build Phase

![添加Run Script][12]

6.拷贝下面脚本到Run Script

```
if [ “${CONFIGURATION}” == “Development” ]; then
    
cp -r “${PROJECT_DIR}/Settings-Development.plist” “${BUILT_PRODUCTS_DIR}/${PRODUCT_NAME}.app/Settings.plist”

elif [ “${CONFIGURATION}” == “Production” ]; then
    
cp -r “${PROJECT_DIR}/Settings-Production.plist” “${BUILT_PRODUCTS_DIR}/${PRODUCT_NAME}.app/Settings.plist”
    
fi
```

7.下面我们就可以在运行时读取配置文件中的内容：

```
var serverUrl: String = ""
if let filePath = NSBundle.mainBundle().pathForResource("Settings", ofType: "plist") {
    let contentsOfFile = NSDictionary(contentsOfFile: filePath)
    serverUrl = contentsOfFile?.objectForKey("ServerUrl") as! String
} else {
    // no settings!
}
```

8.接下来选择运行时的构建环境，选择Product -> Scheme -> Edit Scheme菜单，选中左侧Action列表栏中的"Run"，在右侧选择Development或者Production环境

![选择配置1][13]

![选择配置2][14]

9.同样，你需要为其他的Action如Test，Profile，Analyze和Archive选择构建环境。也就是说，如果在你构建版本到TestFlight之前，你需要将"Archive" Action的配置环境设置为Production!

CocoaPods注意事项：

如果你在创建Devlopment和Production配置之前，你已经使用CocoaPods创建了项目，你可能会碰到错误提示：`[!] CocoaPods did not set the base configuration of your project because your project already has a custom config set. In order for CocoaPods integration to work at all, please either set the base configurations of the target … in your build configuration.
` 或者你会在应用链接时会看到其他pods相关错误。

解决这些错误，你需要到Project setting区域的info栏（和步骤1相同位置）,将所有新增配置的"Based on Configuration File" 设置为 "None"，接着再次运行"pod install"强制pod工具为新创建的环境重新生成配置。同样如果后面你又添加了配置环境，你需要重复上面的操作。

![cocoapods][15]

##接下来

使用上面的方法，你可以在项目配置文件中添加任何配置内容，并在运行时读取他们。当然，如果你的应用需要多种演示测试环境，你可以根据需要创建任意多构建环境。同样你还可以添加多个项目配置文件-记住，你需要为每种构建环境创建相同格式的配置文件，另外你需要在Build Phases脚本内容中将这些配置文件拷贝到程序包中（参考步骤6）。

##反馈

我非常乐意收到您的意见，尤其是可以简化这个过程或者有其他思路的建议。




  [1]: http://emmerge.com/
  [2]: https://mixpanel.com/
  [3]: https://github.com/nsolter/DeploymentConfigurationsDemo
  [4]: https://cdn-images-1.medium.com/max/800/1*bq8nwaYr_VHSUpg6IFw_zw.png
  [5]: https://cdn-images-1.medium.com/max/800/1*3kHjSJQvu4q5-ee2Cv8A-g.png
  [6]: https://cdn-images-1.medium.com/max/800/1*RjY0NjSZ-XhJKX09HnH03A.png
  [7]: https://cdn-images-1.medium.com/max/800/1*JO3L5eOso9ckPesTPGhFNg.png
  [8]: https://cdn-images-1.medium.com/max/800/1*hGaMWtSo53ZGCdp94auWaw.png
  [9]: https://cdn-images-1.medium.com/max/800/1*Fo3R1a6GmV7BFhYBeei1KQ.png
  [10]: https://cdn-images-1.medium.com/max/800/1*kUfaw6X1urdSjb_VccF--w.png
  [11]: https://cdn-images-1.medium.com/max/800/1*8T5E1yUEtei3iD5WbV9rYQ.png
  [12]: https://cdn-images-1.medium.com/max/800/1*Gj6TGRRPuUwq_7atHfi5SA.png
  [13]: https://cdn-images-1.medium.com/max/800/1*-uoHFkxGa1obj5zsncDChQ.png
  [14]: https://cdn-images-1.medium.com/max/800/1*zb5ykn6CHQeKqO2X7ClKRQ.png
  [15]: https://cdn-images-1.medium.com/max/800/1*F7yDDktEEAnXDZs5DpE4Rg.png
