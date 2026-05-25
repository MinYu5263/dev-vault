本篇文章为译文，[原文地址](https://docs.oracle.com/en/java/javase/21/docs/specs/man/jpackage.html)
## 名称

`jpackage` - 用于打包独立运行的Java 应用程序的工具。

## 概要

`jpackage` \[_选项_\]

_选项_

命令行选项，以空格分隔。请参阅 [](.md#jpackage%20选项)。

## 描述

`jpackage` 工具会将 Java 应用程序和 Java 运行时映像作为输入，并生成包含所有必要依赖项的 Java 应用程序映像。它还能生成特定平台格式的本地包，如 Windows 上的 exe 或 macOS 上的 dmg。每种格式都必须在其运行的平台上构建，不支持跨平台。该工具将提供选项，允许以各种方式定制打包的应用程序。


## jpackage 选项

### 通用选项：

`@`_文件名_

从文件中读取选项。
此选项可以多次使用。

`--type` 或 `-t` _类型_

  

要创建的软件包类型

  

有效值为：{"app-image", "exe", "msi", "rpm", "deb", "pkg", "dmg"}

  

如果未指定此选项，将创建一个依赖于平台的默认类型。

  

`--app-version` _版本号_

  

应用程序和 / 或软件包的版本

  

`--copyright` _版权信息_

  

应用程序的版权信息

  

`--description` _描述信息_

  

应用程序的描述

  

`--help` 或 `-h`

  

将当前平台的每个有效选项的列表及描述的使用文本打印到输出流，然后退出。

  

`--icon` _路径_

  

应用程序软件包图标的路径

  

（绝对路径或相对于当前目录的路径）

  

`--name` 或 `-n` _名称_

  

应用程序和 / 或软件包的名称

  

`--dest` 或 `-d` _目标路径_

  

生成的输出文件放置的路径

  

（绝对路径或相对于当前目录的路径）。

  

默认为当前工作目录。

  

`--temp` _目录_

  

用于创建临时文件的新目录或空目录的路径

  

（绝对路径或相对于当前目录的路径）

  

如果指定了此选项，任务完成后临时目录不会被删除，必须手动删除。

  

如果未指定，将创建一个临时目录，并在任务完成后删除。

  

`--vendor` _供应商_

  

应用程序的供应商

  

`--verbose`

  

启用详细输出。

  

`--version`

  

将产品版本打印到输出流，然后退出。

### 创建运行时镜像的选项：

`--add-modules` _模块名称_ [`,`_模块名称_...]

  

以逗号（","）分隔的要添加的模块列表

  

此模块列表将与主模块（如果已指定）一起作为 `--add-module` 参数传递给 `jlink`。如果未指定，那么要么仅使用主模块（如果指定了 `--module`），要么使用默认的模块集（如果指定了 `--main-jar`）。

  

此选项可以多次使用。

  

`--module-path` 或 `-p` _模块路径_ [`,`_模块路径_...]

  

以 `File.pathSeparator` 分隔的路径列表

  

每个路径要么是模块所在的目录，要么是模块化 JAR 的路径，可以是绝对路径或相对于当前目录的路径。

  

此选项可以多次使用。

  

`--jlink-options` _选项_

  

要传递给 `jlink` 的以空格分隔的选项列表

  

如果未指定，默认值为 `--strip-native-commands --strip-debug --no-man-pages --no-header-files`

  

此选项可以多次使用。

  

`--runtime-image` _目录_

  

将被复制到应用程序镜像中的预定义运行时镜像的路径

  

（绝对路径或相对于当前目录的路径）

  

如果未指定 `--runtime-image`，`jpackage` 将运行 `jlink`，使用 `--jlink-options` 指定的选项来创建运行时镜像。

### 创建应用程序镜像的选项：

`--input` 或 `-i` _目录_

  

包含要打包文件的输入目录的路径

  

（绝对路径或相对于当前目录的路径）

  

输入目录中的所有文件都将被打包到应用程序镜像中。

  

`--app-content` _附加内容_[,_附加内容_...]

  

要添加到应用程序有效负载中的文件和 / 或目录路径的逗号分隔列表。

  

此选项可以使用多次。

### 创建应用程序启动器的选项：

`--add-launcher` _名称_=_路径_

  

启动器的名称，以及一个属性文件的路径，该属性文件包含一系列键值对

  

（绝对路径或相对于当前目录的路径）

  

可以使用的键有 “module”、“main-jar”、“main-class”、“description”、“arguments”、“java-options”、“app-version”、“icon”、“launcher-as-service”、“win-console”、“win-shortcut”、“win-menu”、“linux-app-category” 和 “linux-shortcut”。

  

这些选项将添加到原始命令行选项中，或用于覆盖原始命令行选项，以构建一个额外的备用启动器。主应用程序启动器将根据命令行选项构建。可以使用此选项构建额外的备用启动器，并且此选项可以多次使用以构建多个额外的启动器。

  

`--arguments` _参数_

  

如果启动器未接收到命令行参数，则传递给主类的命令行参数

  

此选项可以多次使用。

  

`--java-options` _选项_

  

要传递给 Java 运行时的选项

  

此选项可以多次使用。

  

`--main-class` _类名_

  

要执行的应用程序主类的全限定名

  

只有在指定了 `--main-jar` 时才能使用此选项。

  

`--main-jar` _主 JAR 文件_

  

应用程序的主 JAR 文件；包含主类（指定为相对于输入路径的路径）

  

可以指定 `--module` 或 `--main-jar` 选项，但不能同时指定两者。

  

`--module` 或 `-m` _模块名称_\(/_主类名称_\)

  

应用程序的主模块（可选择包含主类）

  

此模块必须位于模块路径上。

  

当指定此选项时，主模块将被链接到 Java 运行时镜像中。可以指定 `--module` 或 `--main-jar` 选项，但不能同时指定两者。

### 创建应用程序启动器的特定于平台的选项：

#### Windows 平台选项（仅在 Windows 上运行时可用）：

`--win-console`

  

为应用程序创建一个控制台启动器，对于需要控制台交互的应用程序应指定此选项

#### macOS 平台选项（仅在 macOS 上运行时可用）：

`--mac-package-identifier` _标识符_

  

一个唯一标识 macOS 应用程序的标识符

  

默认为主类名。  
只能使用字母数字字符（A-Z、a-z、0-9）、连字符（-）和句点（.）。

  

`--mac-package-name` _名称_

  

应用程序在菜单栏中显示的名称

  

这可以与应用程序名称不同。  
此名称长度必须小于 16 个字符，并且适合在菜单栏和应用程序信息窗口中显示。默认为应用程序名称。

  

`--mac-package-signing-prefix` _前缀_

  

对应用程序包进行签名时，此值会添加到所有需要签名且没有现有包标识符的组件之前。

  

`--mac-sign`

  

请求对软件包或预定义的应用程序镜像进行签名。

  

`--mac-signing-keychain` _钥匙串名称_

  

用于搜索签名身份的钥匙串名称

  

如果未指定，则使用标准钥匙串。

  

`--mac-signing-key-user-name` _名称_

  

Apple 签名身份中的团队或用户名部分

  

`--mac-app-store`

  

表示 `jpackage` 的输出是为 Mac App Store 准备的。

  

`--mac-entitlements` _路径_

  

包含在对捆绑包中的可执行文件和库进行签名时使用的授权信息的文件路径

  

`--mac-app-category` _类别_

  

用于在应用程序的 `plist` 文件中构造 `LSApplicationCategoryType` 的字符串  
默认值为 “utilities”（实用工具）。

### 创建应用程序包的选项：

`--about-url` _网址_

  

应用程序主页的 URL

  

`--app-image` _目录_

  

用于构建可安装软件包（在所有平台上）或进行签名（在 macOS 上）的预定义应用程序镜像的位置  
（绝对路径或相对于当前目录的路径）

  

`--file-associations` _路径_

  

包含键值对列表的属性文件的路径  
（绝对路径或相对于当前目录的路径）

  

键 “extension”（扩展名）、“mime-type”（MIME 类型）、“icon”（图标）和 “description”（描述）可用于描述关联关系。  
此选项可以多次使用。

  

`--install-dir` _路径_

  

应用程序安装目录的绝对路径（在 macOS 或 Linux 上），或者安装目录的相对子路径，如 “Program Files”（程序文件）或 “AppData”（应用数据）（在 Windows 上）

  

`--license-file` _路径_

  

许可证文件的路径  
（绝对路径或相对于当前目录的路径）

  

`--resource-dir` _路径_

  

覆盖 `jpackage` 资源的路径  
（绝对路径或相对于当前目录的路径）

  

通过向此目录添加替换资源，可以覆盖 `jpackage` 的图标、模板文件和其他资源。

  

`--runtime-image` _路径_

  

要安装的预定义运行时镜像的路径  
（绝对路径或相对于当前目录的路径）  
创建运行时安装程序时此选项是必需的。

  

`--launcher-as-service`

  

请求创建一个安装程序，该安装程序会将主应用程序启动器注册为后台服务类型的应用程序。

### 创建应用程序包的特定于平台的选项：

#### Windows 平台选项（仅在 Windows 上运行时可用）：

`--win-dir-chooser`

  

添加一个对话框，使用户能够选择安装产品的目录。

  

`--win-help-url` _网址_

  

用户可以获取更多信息或技术支持的 URL

  

`--win-menu`

  

请求为此应用程序添加一个 “开始” 菜单快捷方式

  

`--win-menu-group` _菜单组名称_

  

此应用程序所在的 “开始” 菜单组

  

`--win-per-user-install`

  

请求按每个用户进行安装

  

`--win-shortcut`

  

请求为此应用程序创建一个桌面快捷方式

  

`--win-shortcut-prompt`

  

添加一个对话框，使用户能够选择安装程序是否创建快捷方式

  

`--win-update-url` _网址_

  

可用的应用程序更新信息的 URL

  

`--win-upgrade-uuid` _ID_

  

与此软件包的升级相关联的 UUID（通用唯一识别码）

#### Linux 平台选项（仅在 Linux 上运行时可用）：

`--linux-package-name` _名称_

  

Linux 软件包的名称  
默认为应用程序名称。

  

`--linux-deb-maintainer` _电子邮件地址_

  

`.deb` 捆绑包的维护者

  

`--linux-menu-group` _菜单组名称_

  

此应用程序所在的菜单组

  

`--linux-package-deps`

  

应用程序所需的软件包或功能

  

`--linux-rpm-license-type` _类型_

  

许可证类型（RPM `.spec` 文件中的 “License: _值_”）

  

`--linux-app-release` _版本号_

  

RPM `<name>.spec` 文件的版本值或 DEB 控制文件的 Debian 修订版本值

  

`--linux-app-category` _类别值_

  

RPM `/`.spec 文件的组值或 DEB 控制文件的 “Section”（部分）值

  

`--linux-shortcut`

  

为应用程序创建一个快捷方式。

#### macOS 平台选项（仅在 macOS 上运行时可用）：

`--mac-dmg-content` _附加内容_[,_附加内容_...]

  

将所有引用的内容包含在 dmg 文件中。  
此选项可以使用多次。

## jpackage 示例

```plaintext
生成适合主机系统的应用程序包：
```

```plaintext
对于模块化应用程序:
    jpackage -n name -p modulePath -m moduleName/className
对于非模块化应用程序:
    jpackage -i inputDir -n name \
        --main-class className --main-jar myJar.jar
对于预构建的应用程序镜像:
    jpackage -n name --app-image appImageDir
```

```plaintext
生成应用程序镜像：
```

```plaintext
对于模块化应用程序：
    jpackage --type app-image -n name -p modulePath \
        -m moduleName/className
对于非模块化应用程序:
    jpackage --type app-image -i inputDir -n name \
        --main-class className --main-jar myJar.jar
要为 jlink 提供您自己的选项，请单独运行 jlink:
    jlink --output appRuntimeImage -p modulePath \
        --add-modules moduleName \
        --no-header-files \[<additional jlink options>...\]
    jpackage --type app-image -n name \
        -m moduleName/className --runtime-image appRuntimeImage
```

```plaintext
生成 Java 运行时包：
```

```plaintext
jpackage -n name --runtime-image <runtime-image>
```

```plaintext
对预定义的应用程序镜像进行签名（在 macOS 上）:
```

```plaintext
jpackage --type app-image --app-image <app-image> \
    --mac-sign \[<additional signing options>...\]

Note: the only additional options that are permitted in this mode are:
      the set of additional mac signing options and --verbose
```

## jpackage 资源目录

通过向该目录添加替换资源，可以覆盖 jpackage 的图标、模板文件和其他资源。jpackage 会在资源目录中按特定名称查找文件。

### 仅在 Linux 上运行时会考虑的资源目录文件：

`<启动器名称>.png`

  

应用程序启动器图标  
默认资源是 _JavaApp.png_

  

`<启动器名称>.desktop`

  

要与 `xdg-desktop-menu` 命令一起使用的桌面文件  
在为文件关联注册的应用程序启动器以及 / 或者具有图标的情况下会被考虑  
默认资源是 _template.desktop_

#### 仅在构建 Linux DEB/RPM 安装程序时会考虑的资源目录文件：

`<软件包名称>-<启动器名称>.service`

  

注册为后台服务类型应用程序的应用程序启动器的 systemd 单元文件  
默认资源是 _unit-template.service_

#### 仅在构建 Linux RPM 安装程序时会考虑的资源目录文件：

`<软件包名称>.spec`

  

RPM spec 文件  
默认资源是 _template.spec_

#### 仅在构建 Linux DEB 安装程序时会考虑的资源目录文件：

`control`

  

控制文件  
默认资源是 _template.control_

  

`copyright`

  

版权文件  
默认资源是 _template.copyright_

  

`preinstall`

  

预安装 shell 脚本  
默认资源是 _template.preinstall_

  

`prerm`

  

预删除 shell 脚本  
默认资源是 _template.prerm_

  

`postinstall`

  

后安装 shell 脚本  
默认资源是 _template.postinstall_

  

`postrm`

  

后删除 shell 脚本  
默认资源是 _template.postrm_

### 仅在 Windows 上运行时会考虑的资源目录文件：

`<启动器名称>.ico`

  

应用程序启动器图标  
默认资源是 _JavaApp.ico_

  

`<启动器名称>.properties`

  

应用程序启动器可执行文件的属性文件  
默认资源是 _WinLauncher.template_

#### 仅在构建 Windows MSI/EXE 安装程序时会考虑的资源目录文件：

`<应用程序名称>-post-image.wsf`

  

构建应用程序镜像后要运行的 Windows 脚本文件 (WSF)

  

`main.wxs`

  

主 WiX 项目文件  
默认资源是 _main.wxs_

  

`overrides.wxi`

  

覆盖 WiX 项目文件  
默认资源是 _overrides.wxi_

  

`service-installer.exe`

  

服务安装程序可执行文件  
如果某些应用程序启动器被注册为后台服务类型的应用程序，则会被考虑

  

`<启动器名称>-service-install.wxi`

  

服务安装程序 WiX 项目文件  
如果某些应用程序启动器被注册为后台服务类型的应用程序，则会被考虑  
默认资源是 _service-install.wxi_

  

`<启动器名称>-service-config.wxi`

  

服务安装程序 WiX 项目文件  
如果某些应用程序启动器被注册为后台服务类型的应用程序，则会被考虑  
默认资源是 _service-config.wxi_

  

`InstallDirNotEmptyDlg.wxs`

  

用于安装程序用户界面对话框的 WiX 项目文件，该对话框检查安装目录是否不存在或为空  
默认资源是 _InstallDirNotEmptyDlg.wxs_

  

`ShortcutPromptDlg.wxs`

  

用于配置快捷方式的安装程序用户界面对话框的 WiX 项目文件  
默认资源是 _ShortcutPromptDlg.wxs_

  

`bundle.wxf`

  

包含应用程序镜像组件层次结构的 WiX 项目文件

  

`ui.wxf`

  

用于安装程序用户界面的 WiX 项目文件

#### 仅在构建 Windows EXE 安装程序时会考虑的资源目录文件：

`WinInstaller.properties`

  

安装程序可执行文件的属性文件  
默认资源是 _WinInstaller.template_

  

`<软件包名称>-post-msi.wsf`

  

为 EXE 安装程序构建嵌入式 MSI 安装程序后要运行的 Windows 脚本文件 (WSF)

### 仅在 macOS 上运行时会考虑的资源目录文件：

`<启动器名称>.icns`

  

应用程序启动器图标  
默认资源是 _JavaApp.icns_

  

`Info.plist`

  

应用程序属性列表文件  
默认资源是 _Info-lite.plist.template_

  

`Runtime-Info.plist`

  

Java 运行时属性列表文件  
默认资源是 _Runtime-Info.plist.template_

  

`<应用程序名称>.entitlements`

  

签名授权属性列表文件  
默认资源是 _sandbox.plist_

#### 仅在构建 macOS PKG/DMG 安装程序时会考虑的资源目录文件：

`<软件包名称>-post-image.sh`

  

构建应用程序镜像后要运行的 shell 脚本

#### 仅在构建 macOS PKG 安装程序时会考虑的资源目录文件：

`uninstaller`

  

卸载 shell 脚本  
如果某些应用程序启动器被注册为后台服务类型的应用程序，则会被考虑  
默认资源是 _uninstall.command.template_

  

`preinstall`

  

预安装 shell 脚本  
默认资源是 _preinstall.template_

  

`postinstall`

  

后安装 shell 脚本  
默认资源是 _postinstall.template_

  

`services-preinstall`

  

服务软件包的预安装 shell 脚本  
如果某些应用程序启动器被注册为后台服务类型的应用程序，则会被考虑  
默认资源是 _services-preinstall.template_

  

`services-postinstall`

  

服务软件包的后安装 shell 脚本  
如果某些应用程序启动器被注册为后台服务类型的应用程序，则会被考虑  
默认资源是 _services-postinstall.template_

  

`<软件包名称>-background.png`

  

背景图片  
默认资源是 _background_pkg.png_

  

`<软件包名称>-background-darkAqua.png`

  

深色背景图片  
默认资源是 _background_pkg.png_

  

`product-def.plist`

  

软件包属性列表文件  
默认资源是 _product-def.plist_

  

`<软件包名称>-<启动器名称>.plist`

  

注册为后台服务类型应用程序的应用程序启动器的 launchd 属性列表文件  
默认资源是 _launchd.plist.template_

#### 仅在构建 macOS DMG 安装程序时会考虑的资源目录文件：

`<软件包名称>-dmg-setup.scpt`

  

设置 AppleScript 脚本  
默认资源是 _DMGsetup.scpt_

  

`<软件包名称>-license.plist`

  

许可证属性列表文件  
默认资源是 _lic_template.plist_

  

`<软件包名称>-background.tiff`

  

背景图片  
默认资源是 _background_dmg.tiff_

  

`<软件包名称>-volume.icns`

  

卷图标  
默认资源是 _JavaApp.icns_