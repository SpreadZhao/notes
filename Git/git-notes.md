# git使用技巧

## 已经推送过的文件，但是本地发现他不用提交(比如clion的cmake-build-debug)

* 可使用如下代码来操作：

  ```git
  git rm --cached <files>
  ```

* 如果删除的文件不在当前目录下，而在子目录下，需要递归操作：

  ```git
  git rm -r --cached <d1/d2/files>
  ```

