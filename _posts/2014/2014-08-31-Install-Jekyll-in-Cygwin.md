前段时间发现Cygwin，可以在windows平台模拟linux下的大部分环境，而且通过其管理git也非常方便，但是在Cygwin下配置jekyll是遇到许多问题，也在网上找了许多方法，尝试了好久最后终于配置成功，在这里记录一下，以后再次配置时能够轻松一点。

安装jekyll需要ruby环境，这里通过rvm进行ruby环境的安装。另外，还需要一些依赖包：patch，zlib-devel，openssl，openssl-devel，libyaml-devel，libyaml0_2，sqlite3，make，libtool，gcc-core，autoconf，automake，bison，m4，mingw64-i686-gcc，mingw64-x86_64-gcc，cygwin32-readline等，如果不想逐个安装，先把Cygwin的包管理软件配置一下。

1. 配置包管理环境：

	curl -L https://get.rvm.io | bash -s stable –ruby


2. 安装rvm
3. 启动rvm
4. 安装ruby（如果ruby版本低于2.1时）
5. 安装jekyll