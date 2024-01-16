### UBuntu-Git And GitHub

> - *什么是SSL？什么是公钥和密钥？*
> 
> > - SSH (Secure Shell) 密钥是用于身份验证和加密通信的一对加密密钥。
> >   它由两个部分组成：私钥（private key）和公钥（public key）。这对密钥是通过非对称加密算法生成的，其中私钥用于加密数据，而公钥用于解密数据
> 
> 1. 在 SSH 中，私钥应该保持在你的本地计算机上，并且必须保持安全和保密。
> 
> 2. 公钥则可以被分享给其他人或服务器。当你连接到一个远程服务器时，你可以将你的公钥添加到服务器上，以便服务器可以使用该公钥对你的身份进行验证。
> 
> 3. 当你使用 SSH 协议连接到远程服务器时，身份验证过程如下：
> 
> > 你的本地计算机向服务器发送请求。
> > 
> > 服务器要求提供身份验证凭据。
> > 
> > 你的本地计算机将使用你的私钥对一个随机生成的数字进行加密，并将加密后的数字发送给服务器。
> > 
> > 服务器使用你之前提供的公钥对加密后的数字进行解密。
> > 如果解密后的数字与服务器生成的数字匹配，服务器将验证你的身份并允许你登录。

##### Git 配置SSH_Key

1. ```git
   $ git config --global user.name "guanxingbusuan"
   $ git config --global user.email "guanxingbusuan@qq.com"
   ```

2. **生成SSH Key**
   
   ```git
   # Github绑定的邮箱
   $ ssh-keygen -t rsa -C "guanxingbusuan@qq.com"
   ```

3. **获取SSH Key**
   
   根据命令行提示，进入文件夹，获取以`ssh-rsa`的字符串（包括`ssh-rsa`)
   
    Enter file in which to save the key (/home/guanxingbusuan/.ssh/id_rsa):
- 复制SSH Key 添加到GitHub

- 检验配置：

- ```git
  $ ssh -T git@github.com
  Hi guanxinbusuan! You've successfully authenticated, but GitHub does not provide shell access.
  ```

> - [x] 使用SSH协议传输：
>   
>   1. 本地化提交完成之后:`git remote add origin 远程仓库的SSH地址`完成本地仓库与远程相关联
>      
>      ```git
>      $ git remote add origin 远程仓库的SSH地址(git@github.com:guanxinbusuan/Study_Notes.git)
>      #remote是远程仓库名
>      #SSH的url在github-repository-code里面查到
>      $ git remote -v
>      #查看远仓的信息
>      origin    git@github.com:guanxinbusuan/Study_Notes.git (fetch)
>      origin    git@github.com:guanxinbusuan/Study_Notes.git (push)
>      ```

> 2. 执行如下命令，将文件上传到 GitHub 的远程仓库。
>    
>    ```git
>    $ git push -u origin main
>    ```

> > ##### 第二类方法：
> > 
> > 复制远程仓库的 SSH 地址，执行如下命令，将远程仓库克隆到本地。
> > 
> > ```git
> > $ git clone 远程仓库的SSH地址
> > ```

> > 本地上多出了一个仓库（自带隐藏文件夹 `.git`），这个本地仓库是通过 `git clone` 而来的，它已经跟 GitHub 上的远程仓库相关联了，所以就省去了 `git init`、`git remote add` 等操作。
> > 
> > - 剩下的步骤与第一类方法相同，跟踪，提交，再`git push -u origin main`

###### **git push -u origin main错误：源引用规格 main 没有匹配：**

> - 因为本地分支名称不是 `main`，而是其他的名称，比如 `master`。
> 
> - 将本地分支推送到远程仓库中，并且与远程仓库中的某个分支进行关联，可以使用以下命令：
> 1. `git push -u <远程主机名> <本地分支名>:<远程分支名>`
> 
> 其中，`<远程主机名>` 是前面添加的远程主机名称，`<本地分支名>` 是你当前所在的本地分支名称，`<远程分支名>` 是想要与之关联的远程分支名称。例如：
> 
> 2. ```git
>    $ git push -u origin master:main
>    ```
> 
> 这条命令会将当前所在的 `master` 分支推送到 `origin` 这个远程主机上，并且与其上面的 `main` 分支进行关联。

###### 
