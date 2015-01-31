aar包是Android库工程的二进制分发包

文件的扩展名是aar，所以maven artifact类型最好也是aar，但是它是一个包含以下条目的简单ZIP压缩包：

* /AndroidManifest.xml (必须的)
* /classes.jar (必须的)
* /res/ (必须的)
* /R.txt (必须的)
* /assets/ (可选的)
* /libs/\*.jar (可选的)
* /jni/<abi>/\*.so (可选的)
* /proguard.txt (可选的)
* /lint.jar (可选的)

以上这些条目是直接在ZIP文件的根目录下的。
s
R.txt文件是aapt命令带有 --output-text-symbols标识选项输出的
 
