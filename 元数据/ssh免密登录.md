1. 进入服务器1的用户目录下的使用以下命令

   ```shel
   (1)  cd ~    // 进入用户目录
   (2)  cd .ssh    //进入.ssh目录
   (3)  ssh-keygen -t dsa  // 一直按回车就行，最后会生成两个文件id_dsa（私钥）  id_dsa.pub（公钥）
   (4)  使用scp ~/.ssh/id_dsa.pub 用户名@主机名:/tmp/  // 使用scp在服务器之间进行复制，第一个是当前文件的路径，第二个是
                                                    //  另一个服务器的路径，在这里我的是root@10.20.0.138:/tmp/
   (5)  在服务器2上使用以下命令  
        cat /tmp/id_dsa.pub >> ~/.ssh/authorized_keys      //将公钥追加到这个文件中   /根目录  ~用户目录
   (6)  回到服务器1，进行测试
        ssh root@10.20.0.138                      // 如果配置成功则不需要登陆密码就能连接
   (7)  exit                                      //断开连接
   ```

2. 这个只是单向连接，想要配置服务器2，也要在服务器2使用同样的方法

   ```shell
   (1)  cd ~    // 进入用户目录
   (2)  cd .ssh    //进入.ssh目录
   (3)  ssh-keygen -t dsa  // 一直按回车就行，最后会生成两个文件id_dsa（私钥）  id_dsa.pub（公钥）
   (4)  使用scp ~/.ssh/id_dsa.pub 用户名@主机名:/tmp/  // 使用scp在服务器之间进行复制，第一个是当前文件的路径，第二个是
                                                    //  另一个服务器的路径，在这里我的是wantb@10.22.128.163:/tmp/
   (5)  在服务器1上使用以下命令  
        cat /tmp/id_dsa.pub >> ~/.ssh/authorized_keys      //将公钥追加到这个文件中   /根目录  ~用户目录
   (6)  回到服务器2，进行测试
        ssh wantb@10.22.128.163                      // 如果配置成功则不需要登陆密码就能连接
   (7)  exit                                      //断开连接
   ```

   

3. 同样的道理 ，多个服务器的话也是相同的配置方法，以上只是两个服务器的免密登陆。