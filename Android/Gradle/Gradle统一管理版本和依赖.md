#### Gradle统一管理版本和依赖

#### 问题和痛点

Android开发中，构建使用的是Gradle脚本。一般我们会将不同的模块划分，一个项目中会有多个module，每个module都会有共同的依赖，例如v4、v7，compileSdkVersion、minSdkVersion、targetSdkVersion等，那么如果我们每个地方都拷贝一遍，以后需要升级某个依赖或者版本就会很麻烦，10个module就需要改10次（这显然是不合理的！）。

在代码中我们对相同的变量、常量会定义在一个常量类中，Gradle中也是可以这样的。

#### 解决方案

1. 每个module都有project对象的引用，叫rootProject。
2. 单独新建一个version.gradle文件存放我们的依赖和版本。
3. 我们给project挂载一个ext属性。
4. ext属性中，定义一个键值对Map对象，存放我们依赖名和具体依赖。
5. 在根build.gradle文件中引入version.gradle文件。
6. 在需要依赖的module中引入即可。

#### 具体步骤

1. 新建一个version.gradle文件，放到项目根目录即可。
2. 增加ext属性。
	- 例如依赖Map：deps。（格式 键名:值名）
	- 编译版本相关Map：android。（格式 键名:值名）

```
ext {
	//对于v4、v7都依赖一个support包的版本，单独抽一个版本常量
	support_version = "28.0.0"

	deps = [
	   ...
	
		v4: "com.android.support:support-v4:$support_version",
		v7: "com.android.support:appcompat-v7:$support_version",
		rv: "com.android.support:recyclerview-v7:$support_version"
		//第三方库
		mmkv: "com.tencent:mmkv-static:1.0.15",
		multitype: "me.drakeet.multitype:multitype:3.4.4",
		...
	]
	
	android = [
            compileSdkVersion       : 28,
            minSdkVersion           : 19,
            targetSdkVersion        : 28,
            androidSupportSdkVersion: "27.1.1"
    ]
}
```

3. 根build.gradle引入version.gradle。

```
buildscript {
	...
}

//引入version.gradle文件
apply from: 'version.gradle'

allprojects {
	...
}
```

4. 在对应需要依赖的module中使用。写法是根据我们摆放的位置来的，所以例如：
	- 编译版本，我们是放在ext -> android下，路径就是rootProject.ext.android，例如我们需要compileSdkVersion，那么就是rootProject.ext.android.compileSdkVersion。
	- 依赖，我们是放在ext -> deps下，路径就是rootProject.ext.deps，例如依赖v7，那么就是rootProject.ext.deps.v7。

```
android {
	compileSdkVersion rootProject.ext.android.compileSdkVersion
	defaultConfig {
	   //编译版本、目标版本从rootProject中开始引用
		minSdkVersion rootProject.ext.android.minSdkVersion
		targetSdkVersion rootProject.ext.android.targetSdkVersion
	}
	...
}

dependencies {
	//依赖统一从rootProject中开始引用，后续名字就是我们定义的ext块中的deps的具体依赖名
	api rootProject.ext.deps.v7
	api rootProject.ext.deps.rv
	//其他第三方库
	api rootProject.ext.deps.mmkv
	api rootProject.ext.deps.multitype
	...
}
```

#### 总结

统一版本和依赖变量有助于我们管理项目，所以还是有必要了解一个Gradle脚本的。