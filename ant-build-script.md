以下内容由[飞雪无情](http://www.flysnow.org)提供翻译

原文地址 <http://tools.android.com/tech-docs/ant-build-script>

## Ant构建脚本

该文档的目的是介绍如何修改默认的build.xml脚本以满足开发者的需求

*持续更新中。*

当前执行ant help命令的输出 (很明显不是最新的).

     [echo] Android Ant 构建. 可用的任务:
     [echo]    help:      显示帮助.
     [echo]    clean:     删除被其他任务创建的所有输出文件.
     [echo]               所有的任务都可以同时依赖clean任务（包括测试工程和库工程）
     [echo]               用法: 'ant all clean'
     [echo]    debug:     构建一个应用并且用debug密钥签名.
     [echo]               'nodeps' 任务可以让你仅仅构建当前工程而忽略其他库
     [echo]               用法:'ant nodeps debug'
     [echo]    release:   构建应用. 生成的apk文件必须在发布之前签名.
     [echo]               'nodeps' 任务可以让你仅仅构建当前工程而忽略其他库
     [echo]               用法：'ant nodeps release'
     [echo]    instrument:构建一个instrumented安装包并且使用debug密钥签名
     [echo]    test:      运行所有测试. 运行的工程必须是一个测试工程并且已经构建完成
     [echo]               用法：ant [emma] debug installt test
     [echo]    emma:      为后续的任务临时启用代码覆盖率
     [echo]    install:   安装构建新生成的包. 该安装包必须是构建任务生成
     [echo]               (debug/release/instrument).如果之前已经安装过，那么在签名
     [echo]               一样的情况下将被重新安装。
     [echo]    installd:  只安装debug包.
     [echo]    installr:  只安装release包.
     [echo]    installi:  只安装instrumented包.
     [echo]    installt:  只安装test包.
     [echo]    uninstall: 从一个模拟器或者设备上卸载应用. 如果可以的话也可以用来卸载test包

统计的任务依赖关系图

nodeps : 

test : -test-project-check > 

debug : -set-mode-check > -set-debug-files > -set-debug-mode > -debug-obfuscation-check > -setup > -build-setup > -pre-build > -code-gen > -pre-compile > -compile > -post-compile > -obfuscate > -dex > -crunch > -package-resources > -package > -post-package > -do-debug > -post-build > 

emma : 

help : 

clean : -pre-clean > 

uninstall : 

all : -setup > 

installr : -set-mode-check > -set-release-mode > install > 

install : 

instrument : -set-mode-check > -set-instrumented-mode > -set-debug-mode > -debug-obfuscation-check > -setup > -build-setup > -pre-build > -code-gen > -pre-compile > -compile > -post-compile > -obfuscate > -dex > -crunch > -package-resources > -package > -post-package > -do-debug > 

installt : -test-project-check > -set-mode-check > -set-debug-files > install > installd > 

installi : -set-mode-check > -set-instrumented-mode > install > 

release : -set-mode-check > -set-release-mode > -release-obfuscation-check > -setup > -build-setup > -pre-build > -code-gen > -pre-compile > -compile > -post-compile > -obfuscate > -dex > -crunch > -package-resources > -package > -post-package > -release-prompt-for-password > -release-nosign > -release-sign > -post-build > 

installd : -set-mode-check > -set-debug-files > install >