# android

                                      Android Studio 批量打包，apk重命名
        
1.在productFlavors添加你需要的所有渠道
  android {

    productFlavors {  //在这里添加你所有需要打包的渠道
        dev {}
        google {}
        myapp {}
        xiaomi {}
        app360 {}
        wandoujia {}
    }
    //添加如下代码
    productFlavors.all { flavors->
	flavors.manifestPlaceholders=[CHANNEL_VALUE:name]
    }
}



同时修改androidManifest.xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    package="cn.op.zdf"
    android:versionCode="25"
    android:versionName="3.1.2">


	<application 
		android:name=".myApplication">
	
		<-- 在manifest中添加如下代码--->
		<meta-data
            		android:name="UMENG_CHANNEL"
            		android:value="${CHANNEL_VALUE}"/>


	</application>


</manifest>           


2.如何给apk重命名

发布产品的时候我们需要如下的命名规则

release版本的命名规则如下：

产品名称-版本号-渠道号-release.apk

在build.gradle中添加如下代码

//获取时间戳
def getDate() {
    def date = new Date()
    def formattedDate = date.format('yyyyMMddHHmm')
    return formattedDate
}
//从androidManifest.xml中获取版本号
def getVersionNameFromManifest(){
    def manifestParser = new com.android.builder.core.DefaultManifestParser()
    return manifestParser.getVersionName(android.sourceSets.main.manifest.srcFile)
}
android{

    //修改生成的apk名字
    applicationVariants.all{ variant->
        variant.outputs.each { output->
            def oldFile = output.outputFile
            def newName = '';
            if(variant.buildType.name.equals('release')){
//                println(variant.productFlavors[0].name)
                def releaseApkName = '产品名称-v' + getVersionNameFromManifest() + '-' + variant.productFlavors[0].name + '-release.apk'
                output.outputFile = new File(oldFile.parent, releaseApkName)
            }
           
            if(variant.buildType.name.equals('debug')){
        newName = oldFile.name.replace(".apk", "-v" + getVersionNameFromManifest() + "-build" + getDate() + ".apk")
                output.outputFile = new File(oldFile.parent, newName)
            }
        }
    }
}

注：
     我的项目是从eclipse中迁移过来的，所以我是从manifest文件中读取的版本号，就是上面的那个函数getVersionNameFromManifest()
  如果你的版本号定义在build.gradle中，那defaultConfig.versionName就是你的版本号。
