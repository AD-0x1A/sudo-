#  sudo权限绕过漏洞分析与复现

## 一、漏洞介绍

* CVE官方于2019年10月14日发布了CVE-2019-14287漏洞，该漏洞为Linux 近期曝出的一个提权漏洞，直指 sudo 的一个安全策略隐患 ，即便配置中明确不允许 root 用户访问，该漏洞仍可允许恶意用户或程序，在目标 Linux 系统上以 root 用户身份执行任意命令。

## 二、漏洞原理

* Sudo 特指“超级用户”。作为一个系统命令，其允许用户以特殊权限来运行程序或命令，而无需切换使用环境通常以 root 用户身份运行命令。
* 在大多数 Linux 发行版中，/ etc / sudoers 的 RunAs 规范文件中的 ALL 关键字，允许 admin 或 sudo 
* 分组中的所有用户，以系统上任何有效用户的身份运行任何命令。由于特权分离是 Linux 中最基本的安全范例之一，因此管理员可以配置 sudoers 文件，来定义哪些用户可以运行哪些命令。
* 但1.8.28 之前的sudo版本存在一个BUG，如果将用户 ID 转换为用户名的函数，会将 -1（或无效等效的 4294967295）误认为 0，而这正好是 root 用户 User ID 。该BUG可允许用户绕过安全策略，并完全控制系统

## 三、漏洞复现

* 实验环境：kali虚拟机、1.8.28版本之前的sudo版本

### 1、查看sudo版本

* 命令 ：sudo -V

![Image and Preview Themes on the toolbar](https://markdownmonster.west-wind.com/docs/images/EditorPreviewThemeUi.png) 

* 可以看到，sudo版本为1.8.27，可以实现sudo权限绕过漏洞的复现

### 2、创建新用户cn001

* 命令 ： （1）useradd cn001（添加用户）（2）passwd cn001（设置用户密码）

![Image and Preview Themes on the toolbar](https://markdownmonster.west-wind.com/docs/images/EditorPreviewThemeUi.png) 

### 3、修改/etc/sudoers文件

* leafpad  /etc/sudoers（打开/etc/sudoers）
* 添加 cn001 ALL=(ALL,!root) ALL
![Image and Preview Themes on the toolbar](https://markdownmonster.west-wind.com/docs/images/EditorPreviewThemeUi.png) 
* 1.其中的cn001表示用户名 
* 2.第一个ALL表示允许该用户在任意机器或者终端中使用sudo
* 3.括号里面的（ALL,!root）表示命令可以被除了root以外的任意用户身份去执行
* 4.最后一个ALL表示被允许执行

### 4、实现sudo权限绕过

![Image and Preview Themes on the toolbar](https://markdownmonster.west-wind.com/docs/images/EditorPreviewThemeUi.png) 

* 可以看到，cn001用户本身无权以root身份执行id命令
* 但是，-u#-1 将用户uid改为0，即为root，所以用户可以在无root密码的情况下执行root命令

## 五、总结
* 本次漏洞复现过程总体来说比较顺利，本次实验让我感触最深的是，编写程序时一定要时刻注意程序的安全性，避免留下漏洞。
