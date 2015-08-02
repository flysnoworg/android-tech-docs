以下内容由[飞雪无情](http://www.flysnow.org)提供翻译

原文地址 <http://tools.android.com/tech-docs/ant-tasks>

## Ant任务

此功能还在积极开发过程中，可以到ADT-dev中讨论

当前自定义的任务列表：

* AaptExecTask
* AidlExecTask
* ApkBuilderTask
* BuildConfigTask
* DexExecTask
* IfElseTask
* PropertyByReplaceTask
* RenderScriptTask
* SignApkTask
* XPathTask
* ZipAlignTask

r20版本新增的任务：

* CheckEnvTask
* ComputeDependencyTask
* ComputeProjectClasspathTask
* GetEmmaFilterTask
* GetLibraryListTask
* GetTargetTask
* GetTypeTask
* ManifestMergerTask

被以上主要任务所使用的一些基本任务：

* BuildTypedTask
* MultiFilesTask
* SingleDependencyTask
* SingleInputOutputTask

### com.android.ant.CheckEnvTask:&lt;checkenv&gt;

对如下的环境信息作一些简单的校验:

* Ant的版本
* Android SDK里的platform-tool文件夹是否存在

该任务没有属性可配置

### com.android.ant.GetTypeTask &lt;gettype&gt;

获取项目工程的类型，其取值可能是：

* "app"
* "library"
* "test"
* "test-app"

|属性名|描述|是否必须|
|---------|-----------|--------|
|projectTypeOut|用于存储获得的项目类型|是

### com.android.ant.GetTargetTask &lt;gettarget&gt;

解析项目target，返回构建所必需依赖的target的值

|属性名|描述|是否必须|
|---------|-----------|--------|
|androidJarFileOut|存储获得的构建工程的target android.jar位置|是
|androidAidlFileOut|存储获得的构建工程的target framework.aidl位置|是
|bootClassPathOut|存储获得的一个Path对象，该对象包含所有启动jar(android.jar + add-on APIs)|是
|targetApiOut|存储获得的构建工程的target API级别|是
|minSdkVersionOut|存储获得的应用的minSdkVersion的值|是

### com.android.ant.GetLibraryListTask &lt;getlibs&gt;

统计工程所依赖的所有库工程的列表。该列表包含所有直接和间接的依赖，并且按指定的顺序排列。

|属性名|描述|是否必须|
|---------|-----------|--------|
|libraryFolderPathOut|存储获得的一个Path对象，该对象包含所有的库工程文件夹，并且以编译顺序排序|是

### com.android.ant.ComputeDependencyTask &lt;dependency&gt;

统计工程的依赖。主要是关于库工程相关的细节，当然也包括和主工程和它的库工程有关的所有jar库。
它会使用 [这里](http://tools.android.com/recent/dealingwithdependenciesinandroidprojects) 提到的依赖方案进行处理。

|属性名|描述|是否必须|
|---------|-----------|--------|
|libraryFolderPathOut|存储获得的一个Path对象，该对象包含所有的库工程文件夹，并且以编译顺序排序|是
|libraryPackagesOut|存储获得的所有库工程的包名，并且以逗号分割|是
|libraryManifestFilePathOut|存储获得的一个Path对象，该对象包含所有库工程的Manifest文件|是
|libraryResFolderPathOut|存储获得的一个Path对象，该对象包含所有库工程的资源文件夹。<br/它>使用aapt的顺序排序，正好和libraryFolderPathOut的顺序相反|是
|libraryNativeFolderPathOut|存储获得的一个Path对象，该对象包含所有的库工程本地文件夹|是
|jarLibraryPathOut|存储获得的一个Path对象，该对象包含所有构建所需的jar库|是
|targetApi|项目构建的target API级别|是
|verbose|项目构建显示的明细级别|否

### 其他的任务将在以后引入补充