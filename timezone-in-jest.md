# TimeZone in Jest

jest 和 node 相同，在运行时没有时区的概念，导致某些场景的下和业务实际的场景不同。

要解决这个问题有两种处理方法：

在 jest 启动的时候设置环境变量 TZ

```bash
TZ=UTC jest --config=jest.config.js
```

***

或者在 jest.config.js 中配置 globalSetup

```javascript
let config = {
    ... ...
    // A path to a module which exports an async function that is triggered once before all test suites
    globalSetup: "<rootDir>/global-setup.js",
    ... ...
}
module.exports = config
```

在根目录新建一个 global-setup.js

```javascript
module.exports = async () => {
    process.env.TZ = "Asia/Shanghai";
};
```



这里的 UTC 和 Asia/Shanghai 需要遵循 UTC 标准写法，[点此查看规则](https://www.zeitverschiebung.net/de/)



最后我们可以在控制台打印看到，已经根据对应的时区显示了

<figure><img src=".gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

