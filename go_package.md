# 包
## 工作空间
工作空间需要bin,pkg,src三个目录组成。
```
workspace
    |
    +--- bin // go install 安装⺫⽬目录。
    |   |
    |   +--- learn
    |
    |
    +--- pkg // go build ⽣生成静态库 (.a) 存放⺫⽬目录。
    |       |
    |       +--- darwin_amd64
    |       |
    |       +--- mylib.a
    |       |
    |       +--- mylib
    |       |
    |       +--- sublib.a
    |
    +--- src // 项⺫⽬目源码⺫⽬目录。
        |
        +--- learn
        |       |
        |       +--- main.go
        |
        |
        +--- mylib
                |
                +--- mylib.go
                |
                +--- sublib
                         |
                         +--- sublib.go

```