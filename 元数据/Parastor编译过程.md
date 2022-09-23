### 01 安装包的下载

**错误方法：一定不要去windows下面的git下载想要编译的安装包，在编译的时候总是会出现一些莫名其妙的问题。**

![image-20220706090501307](https://tbwan.oss-cn-chengdu.aliyuncs.com/imgs/%E5%85%83%E6%95%B0%E6%8D%AEimage-20220706090501307.png)

**正确方法：**

1. 首先登陆远程桌面SecureCRT软件。

2. 使用git clone http://............下载项目文件。在使用git时，可能会遇到以下错误，这是因为有一个文件的名字太长

   ![image-20220706094952172](https://tbwan.oss-cn-chengdu.aliyuncs.com/imgs/%E5%85%83%E6%95%B0%E6%8D%AEimage-20220706094952172.png)

   解决方法：

   ```shell
   1.先进入下载文件的目录。
   2.使用以下命令。
    git reset
    git config core.protectNTFS false
    git checkout
   3.此时就可以使用git checkout 命令修改分支且不会报错了。
   ```

   

3. 下载完成后，先使用dce命令选择编译环境，在这里选择了第八个，如下图所示。

![image-20220706095702755](https://tbwan.oss-cn-chengdu.aliyuncs.com/imgs/%E5%85%83%E6%95%B0%E6%8D%AEimage-20220706095702755.png)

​        然后进入选择编译环境的选项

![image-20220706095741199](https://tbwan.oss-cn-chengdu.aliyuncs.com/imgs/%E5%85%83%E6%95%B0%E6%8D%AEimage-20220706095741199.png)

### 02 编译

1. 输入以下命令进行编译，这两个命令只需要使用其中一个就行。一般使用最上面的，一般最后两个参数不用添加。

![image-20220704111115072](https://tbwan.oss-cn-chengdu.aliyuncs.com/imgs/%E5%85%83%E6%95%B0%E6%8D%AEimage-20220704111115072.png)

2. 编译成功后会在ofs/code/bin目录下生成一个服务器部署文件

![image-20220706133941589](https://tbwan.oss-cn-chengdu.aliyuncs.com/imgs/%E5%85%83%E6%95%B0%E6%8D%AEimage-20220706133941589.png)

3. 使用`tar xvf OFS***`命令对这个包进行解压