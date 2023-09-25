# Add GCC for node-canvas in Centos7.x

Node环境做canvas绘画时，依赖CXXABI\_1.3.9，但目前有一些centOS7的环境本身并不带此依赖。 在常规服务器上，我们只需要手动安装即可。但如果是通过ci/cd做镜像部署的时候，需要在项目中添加postinstall来实现。

#### 问题描述: <a href="#wen-ti-miao-shu" id="wen-ti-miao-shu"></a>

```bash
Error: /lib64/libstdc++.so.6: version `CXXABI_1.3.9' not found
```

centos7.x 缺少 CXXABI\_1.3.9

\


#### 解决方案： <a href="#jie-jue-fang-an" id="jie-jue-fang-an"></a>

1. package依赖canvas版本在2.8以上
2.  新增npm script：

    ```json
    "postinstall": "yum install -y gcc-c++ cairo-devel libjpeg-turbo-devel pango-devel giflib-devel librsvg2-devel build && npm rebuild canvas --build-from-source"
    ```
3.  项目 script 文件夹下新增 `image_hook.sh`:

    ```bash
    yum install -y cairo-devel libjpeg-turbo-devel pango-devel giflib-devel librsvg2-devel
    ```

\


#### TODO <a href="#todo" id="todo"></a>

另外这里还有一个问题，由于yum本身在 `Windows/Mac` 并不存在，因此在postinstall时需要考虑判断操作系统。

\


#### 相关问题 <a href="#xiang-guan-wen-ti" id="xiang-guan-wen-ti"></a>

[Cannot get canvas running on CentOS 7](https://github.com/Automattic/node-canvas/issues/1796)
