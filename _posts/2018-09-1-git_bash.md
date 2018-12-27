##  Git中git commit -m ""与git commit -a -m "" 的区别

##### 一般仓库中的文件可能存在于这三种状态：

+ 1）Untracked files → 文件未被跟踪；

+ 2）Changes to be committed → 文件已缓存，这是下次提交的内容；

+ 3）Changes bu not updated → 文件被修改，但并没有添加到缓存区。



> git commit -m "xx"  只会提交添加到缓存区的文件（只提交添加的）
>
> git commit -a -m "xx"  能提交修改过，但是没有添加到缓存区的文件（修改过的就能提交）

