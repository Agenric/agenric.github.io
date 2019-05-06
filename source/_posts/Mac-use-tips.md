---
title: Mac使用小技巧
date: 2019-01-18 11:27:03
 - Mac
categories:
 - 效率
---

> 保存一些小的tips

* Mac系统语言为英文的情况下，设置office语言为中文

```bash
defaults write com.microsoft.Word AppleLanguages '("zh-cn")'
defaults write com.microsoft.Excel AppleLanguages '("zh-cn")'
defaults write com.microsoft.Powerpoint AppleLanguages '("zh-cn")'
```

* Mac应用无法打开或文件损坏

```bash
sudo spctl --master-disable
```

* VSCode 插件降级

    1. 找到你需要降级的插件的 [作者：author] 和 [名称：name]
    2. 修改这个 URL : `https://[author].gallery.vsassets.io/_apis/public/gallery/publisher/[author]/extension/[name]/[version]/assetbyname/Microsoft.VisualStudio.Services.VSIXPackage` 然后扔到浏览器下载
    3. 下载完成之后修改扩展名 `.VSIXPackage` 为 `.vsix`
    4. Terminal 执行

        ```shell
        $ code --install-extension Microsoft.VisualStudio.Services.vsix
        Extension 'Microsoft.VisualStudio.Services.vsix' was successfully installed!
        ```

    5. 然后打开 VSCode 你会发现，旧版本回来了。
    6. 对了，记得关闭 VSCode 插件自动更新。