# Github 仓库操作



### 1. 新建仓库

---

​	在github中选择 **New repository** ，输入仓库名称后新建仓库



### 2. 本地操作

---

​	在本地目录下依次进行：

```
git init
git remote add origin https://github.com/sxsp/docs.git
git branch -M main
git add .
git commit -m "提交内容"
git push -u origin main
```



### 3. 更新仓库

---

​	云端更新本地：

```
git pull
```

​	本地更新云端：

```
git add .
git commit -m "提交内容"
git push
```

