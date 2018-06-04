# devdocs-desktop [![explain](http://llever.com/explain.svg)](https://github.com/chinanf-boy/Source-Explain)

「 Desktop client for devdocs.io 」

应该第一个 `electron 应用` explain, 所以选了个相对程度但不复杂的

Explanation

> "version": "1.0.0"

[github source](https://github.com/egoist/devdocs-desktop)

[中文](./readme.md) | ~~[english](./readme.en.md)~~

---

<span> 穷 </span>

<a href="https://patreon.com/yobrave">
<img src="https://c5.patreon.com/external/logo/become_a_patron_button@2x.png" height="30"></a> 
---

<!-- START doctoc -->
<!-- END doctoc -->

---

## package.json

``` js
  "main": "app/index.js", // 主入口
  "scripts": {
    "postinstall": "electron-builder install-app-deps",
    "test": "npm run lint",
    "lint": "xo",
    "app": "DEBUG=devdocs-desktop:* electron app/index.js",
    "pack": "build --dir",
    "dist": "build",
    "release": "build"
  },
```

### 1.  `postinstall`

在安装软件包后运行。 

> `electron-builder install-app-deps` 下载 应用的 依赖

### 2.  [electron-builder](https://github.com/electron-userland/electron-builder)

一个完整的解决方案，打包并构建一个准备发布的电子应用程序，带有“自动更新”支持

### 3.  `build`

提供 使用 `elertron-builder` 的快捷, 同时依赖`package.json`-`build`字段的定义

<details>

<summary> 本应用 build 字段 细节 </summary>

``` js
 "build": {
    "appId": "com.egoistian.devdocs", // id
    "productName": "DevDocs", // name
    "compression": "maximum", // 压缩级别
    "asar": true, // 是否使用 Electro n的存档格式将应用程序的源代码打包到档案中。
    "mac": {
      "category": "public.app-category.developer-tools" // 应用程序类别类型
    },
    "win": {
      "target": [ // 目标封装类型
        "nsis", // Windows的默认目标
        "zip",
        "portable" // 便携式应用，而无需安装
      ]
    },
    "nsis": {
      "oneClick": false // 是创建一键安装还是辅助
    },
    "linux": {
      "synopsis": "DevDocs desktop app", // 简短说明
      "category": "Development", // 应用程序类别
      "target": [ // 目标封装类型
        "AppImage", // 让Linux应用随处运行 
        "deb", // Debian软件包选项
        "tar.xz"
      ]
    }
  }
```

- [build 全配置](https://www.electron.build/configuration/configuration) `en`
- [AppImage](https://appimage.org/)

</details>

## app/index.js

### electron-控制-方式

既然是 `electron` 应用

就有必要说明一下, electron 控制网页 的 方式



