以下内容由[飞雪无情](http://www.flysnow.org)提供翻译

原文地址 <http://tools.android.com/tech-docs/tools-attributes>

## Tools 属性

为了在XML文件中记录一些信息，Android专门定义了名为tools的XML命名空间。在应用打包的时候这些信息会被自动去掉，所以不会影响运行和下载的包大小。命名空间的URI是`http://schemas.android.com/tools`，一般以tools作为前缀：


	<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    	xmlns:tools="http://schemas.android.com/tools"
    	android:layout_width="match_parent"
    	android:layout_height="match_parent" >
    ....

该文档记录了我们当前tools属性的用法.(** 注意：这可能会随时改变 **)

### tools:ignore

这个属性可以在任何XML元素上设置，其值是一个lint问题ID的逗号分割的列表，设置后该XML元素以及其子元素都将被递归的忽略。

	<string name="show_all_apps" tools:ignore="MissingTranslation">All</string>

用途: Lint

### tools:targetApi

该属性和Java类里的 `@TargetApi` 注解的作用是一样的：它可以让你指定元素使用的API的级别，其值既可以是整数也可以是代号名称

    <GridLayout tools:targetApi="ICE_CREAM_SANDWICH" >

用途: Lint

### tools:locale

该属性可以在资源文件的根元素上设置，可以设置一个合适的语言以及一个可选的地区。这样可以让tools知道资源文件里的字符串应用的是什么语言。比如，`values/strings.xml` 可以有如下根元素：

	<resources xmlns:tools="http://schemas.android.com/tools" tools:locale="es">

现在我们知道，默认values文件里的字符串使用的是西班牙语，而不是英语。
用途: Lint, Studio (可以在非英语的资源文件中禁用拼写检查)

### tools:context

该属性通常被设置在布局文件的根元素上，记录布局文件所关联的Activity（设计时，一个布局可能会被多个部门引用）。这可以用来让布局编辑器知道其默认的主题，因为主题一般都是在清单文件里和与之关联的Activity里定义，而不是在布局文件里。和在清单文件中指定activity的类一样，你也可以使用.开头设置。

	<android.support.v7.widget.GridLayout
    	xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:tools="http://schemas.android.com/tools"
        tools:context=".MainActivity" ... >

用途：Studio & Eclipse中的布局编辑器以及Lint。

#### tools:layout

此属性通常设置在&lt;fragment&gt;标签中，用来记录在设计时，你想看到的呈现的布局（运行时，将会由标签中给出的fragment类来决定）。

	<fragment android:name="com.example.master.ItemListFragment"
    	tools:layout="@android:layout/list_content" />

用途: Studio & Eclipse的布局编辑器

### tools:listitem / listheader / listfooter

这些属性可以被用在一个&lt;ListView&gt;(或者&lt;GridView&gt;，&lt;ExpandableListView&gt;这些AdapterView的子View)上，用于在设计时指定list元素、list头、list底的布局。工具就会填充一些虚拟的数据显示一个有代表性内容的列表。

    <ListView
        android:id="@android:id/list"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:listitem="@android:layout/simple_list_item_2" />

用途: Studio & Eclipse的布局编辑器

### tools:showIn

该属性需要设置在被另外一个布局包含的一个布局的根元素中。允许你设置包含该布局的布局文件，并且在设计时，这个被包含的布局将会在其外部的布局里渲染呈现。这允许你在上下文里查看和编辑布局。需要Studio 0.5.8及其以后版本支持。更多信息请参考[发布公告](http://tools.android.com/recent/androidstudio058released)

    <?xml version="1.0" encoding="utf-8"?>
    <TextView xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:tools="http://schemas.android.com/tools"
        android:text="@string/hello_world"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        tools:showIn="@layout/activity_main" />

用途: Studio布局编辑器

### tools:menu

该属性设置在布局的根元素上，作用是配置在Action Bar显示的菜单。Android Studio通过和该布局关联的Activity（通过tools:context找到）的onCreateOptionsMenu()方法尝试找出在Action Bar使用的菜单。者允许你覆盖搜索和已确认状态的菜单。该属性值是一个逗号分割的id列表(不需要@id和其他任何前缀)。你也可以用不带.xml扩展名的xml菜单的文件名。必须是0.8.0及其之后的Studio版本才支持。

    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:tools="http://schemas.android.com/tools"
        android:orientation="vertical"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:menu="menu1,menu2" />

用途: Studio布局编辑器

### tools:actionBarNavMode

概述行设置在布局的根元素上，以配置Action Bar的导航模式。有"standard", "list" 以及 "tabs"这三个值可供选择，需要0.8.0及其之后的Studio版本支持。

    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:tools="http://schemas.android.com/tools"
        android:orientation="vertical"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:actionBarNavMode="tabs" />

用途: Studio布局编辑器

### 其他: 设计时属性

在布局中，任何一个属性都有一个与之对应的内置的Android属性。比如，你能设置一个只在设计时显示的替代文本，但是在实际运行的时候却不显示。要了解更多相信，请参考[设计时布局属性](http://tools.android.com/tips/layout-designtime-attributes)
