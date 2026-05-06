仅作工作留痕（wds代码审计部分）

#### 一、src/src/server

​	公钥颁发，主要程序认证机构，类似CA机构（RSA）。

#### 二、src/src/client

​	主要程序包含 证书认证、（管理员/录入员）登入系统、svs图像导入导出系统。

##### 1、src/src/client/main.py

衔接证书认证系统和（管理员/录入员）登入系统的主干

PasswordManager：负责（管理员/录入员）登入系统与数据库密码传参时的加解密工作，加密算法为sha256.

MainWindowThread：svs图像导入导出系统的线程窗口调用。

LoginWindow(QMainWindow):负责（管理员/录入员）登入系统与数据库密码的ui调用和监听应用级别的用户操作实现超时退出的功能。





##### 2、

