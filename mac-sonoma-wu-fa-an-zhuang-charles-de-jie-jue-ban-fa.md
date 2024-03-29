# Mac Sonoma 无法安装 Charles 的解决办法

今天下载了 Charles 想要安装在 Mac，但却报了以下错误：

<div align="left">

<figure><img src=".gitbook/assets/image (3).png" alt="" width="375"><figcaption></figcaption></figure>

</div>

刚开始还以为是因为新系统不兼容 Charles，但在下载最新 beta 版写明支持 **Apple Silicon** 后，还是报同样的错误，在搜了网上不少方法之后才解决。

先说结论，根本原有是 Charles 是通过 DiskImageMounter 来启动的，但 DiskImageMounter 没有系统磁盘权限，鬼知道我之前的 dmg 文件是怎么安装上的 😓

<figure><img src=".gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

所以要解决这个问题，就是给 DiskImageMounter.app 完全磁盘访问权限。

由于 DiskImageMounter 并不是在应用程序中，正常的访达路径也比较难找到，我们可以在 iTerm 中，先找到对应的路径

```bash
cd /System/Library/CoreServices/
open .
```

打开对应的文件夹，再将 CoreServices 文件夹放置在访达左侧的快捷访问中

<figure><img src=".gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

打开系统设置 -> 隐私与安全性 -> 完全磁盘访问权限，点击新增将 DiskImageMounter 添加到列表里

<figure><img src=".gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

再次安装 Charles 就可以了。



