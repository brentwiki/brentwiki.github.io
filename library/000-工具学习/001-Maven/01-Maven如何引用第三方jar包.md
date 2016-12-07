在用maven管理项目时，当我们使用的第三方的jar包在Maven中央仓库没有下载，这个时候有两种解决办法，第一种是将jar包直接放在工程中，以指定systemPath来引用；另一种方法就是采用架设Nexus私服，通过将jar直接发布到私服上来解决引用问题。
## 引用工程内的jar包
举个最常见的例子就是ojdbc的引用，maven中央仓库是没有下载，采用本地引用的配置如下：
```xml
<dependency>
    <groupId>ojdbc</groupId>
    <artifactId>ojdbc</artifactId>
    <version>14</version>
    <scope>system</scope>
    <systemPath>${project.basedir}/lib/ojdbc14.jar</systemPath>
</dependency>
```
注意：
* systemPath必须是具体的jar，而不是jar所在目录.
* groupId, artifactId, version 必须设置，否则pom语法出错
* scope必须是system，当为system的时候maven将从systemPath中查找所需jar包，而不是从repository.
* 虽然jar包的名字叫ojdbc14.jar，但打出来的包命名规则是artifactId-version.jar
## Nexus引用
使用Nexus私服不仅可以管理第三方jar的发布，还可以发布自己项目中的组件，还有另外的好处就是可以节省外网带宽，加快下载速度。
### Nexus搭建
网上很多教程，这里不再重复了
### 创建deploy用户
在nexus上创建一个专门deploy jar包的用户
### 修改本地maven配置，配置发布帐号
比如，我创建一个用户叫deploy,在配置文件中新增以下配置：
```xml
<servers>
    <server>
        <id>xxxx-releases</id>
        <username>deploy</username>
        <password>xxxx</password>
    </server>
</servers>
```
### deploy第三方jar包
最常规的做法是执行maven deploy命令
```shell
mvn deploy:deploy-file -DgroupId=com.xxx -DartifactId=yyyy -Dversion=1.0.1 -Dpackaging=jar -Dfile=D:\User\Desktop\xxxx.jar -Durl=http://10.19.1.22/nexus/content/repositories/thirdparty/ -DrepositoryId=xxxx-releases
```
注意：
* 命令中的repositoryId参数值要和上面server配置中的id要对应
* url表示要发布的私服仓库地址，可以发布到thirdparty，也可以直接发到releases中
### 使用脚本来发布jar包
使用命令行感觉太麻烦，为此我写了一个bat脚本，通过jar包重命名成一定规则名字来完成发布，脚本如下：
```bat
@echo off
setlocal enabledelayedexpansion
set tpath=%1
for /f  %%i in ("%tpath%") do (
    set tname=%%~ni
)
for /f "delims=- tokens=1,*" %%i in ("%tname%") do (
    set jgroup=%%i
)
for /f "delims=- tokens=2,*" %%i in ("%tname%") do (
    set jname=%%i
)
for /f "delims=- tokens=3,*" %%i in ("%tname%") do (
    set jversion=%%i
)
echo 即将执行以下命令，请核对，确认请按1
echo mvn deploy:deploy-file -DgroupId=%jgroup% -DartifactId=%jname% -Dversion=%jversion% -Dpackaging=jar -Dfile=%tpath% -Durl=http://10.19.1.22/nexus/content/repositories/releases/  -DrepositoryId=xxxx-releases
set /p var=是否确认：
if %var%==1 (
    echo deploy jar to Nexus 开始...
    mvn deploy:deploy-file -DgroupId=%jgroup% -DartifactId=%jname% -Dversion=%jversion% -Dpackaging=jar -Dfile=%tpath% -Durl=http://10.19.1.22/nexus/content/repositories/releases/ -DrepositoryId=xxxx-releases >deploy.log
    echo deploy jar to Nexus 结束...
)else (
    echo 取消命令，按任意键退出
)
pause
```
将代码复制到新建文件depoly.bat中。
我们将jar包重命名成groupId-artifactId-version.jar之后，直接拖到deploy.bat上，按1确认后会在当前目录中生成deploy.log日志。
