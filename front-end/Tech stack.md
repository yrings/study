# Tech stack

首先是一些基础必备的[技术栈](https://www.zhihu.com/search?q=技术栈&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A2903471260})。

1. HTML/HTML5、JavaScript（ES5、ES6等）、CSS/CSS3、[浏览器原理](https://www.zhihu.com/search?q=浏览器原理&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A2903471260})、浏览器兼容。
2. 网络，对HTTP、TCP等协议原理有深刻的理解。
3. React、Vue学会一种就行了。
4. 熟练使用[浏览器工具](https://www.zhihu.com/search?q=浏览器工具&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A2903471260})，Firebug、Chrome Dev Tools等，擅用搜索引擎，[抓包工具](https://www.zhihu.com/search?q=抓包工具&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A2903471260})Fiddler、Charles等。
5. 理解构建，会用打包工具（webpack、rollup等），预处理和模版引擎（Sass、Jade等）。
6. 使用[单元测试](https://www.zhihu.com/search?q=单元测试&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A2903471260})，保证代码的质量，业务的准确。

- **图形编辑器**，如 [GIMP](https://www.gimp.org/) 、[Figma](https://www.figma.com/) 、[Paint.NET](https://www.getpaint.net/) 、[Photoshop](https://www.adobe.com/products/photoshop.html) 、[Sketch](https://www.sketch.com/) 或 [XD](https://www.adobe.com/products/xd.html) ，为你的网页制作图像或图形。



- `<meta charset="utf-8">`——该元素指明你的文档使用 UTF-8 字符编码，UTF-8 包括世界绝大多数书写语言的字符。它基本上可以处理任何文本内容。以它为编码还可以避免以后出现某些问题，没有理由再选用其他编码。
- `<meta name="viewport" content="width=device-width">`——[视口元素](https://developer.mozilla.org/zh-CN/docs/Web/CSS/Viewport_concepts#移动设备的视口)可以确保页面以视口宽度进行渲染，避免移动端浏览器上因页面过宽导致缩放。

## [通过 GitHub 发布](https://developer.mozilla.org/zh-CN/docs/Learn/Getting_started_with_the_web/Publishing_your_website#通过_github_发布)

现在，让我们通过 Github 页面告诉你公布的你的代码是如此的简单。

1. 首先， [注册一个 GitHub 账号，](https://github.com/join) 并确认你的邮箱地址。

2. 接下来，你需要创建一个新的资源库 ( repository ) 来存放你的文件。

3. 在这个页面上，在 *Repository name* 输入框里输入 *username*.github.io，username 是你的用户名。比如，我们的朋友 bobsmith 会输入 *bobsmith.github.io。同时勾选* *Initialize this repository with a README*，然后点击 *Create repository*。![img](https://developer.mozilla.org/zh-CN/docs/Learn/Getting_started_with_the_web/Publishing_your_website/github-create-repo.png)

4. 然后，将你的网站文件夹里的内容拖拽到你的资源库 ( repository )，再点击Commit changes

   **备注：** 确保你的文件夹有一个 *index.html* 文件。

5. 现在将你的浏览器转到 *username*.github.io 来在线查看你的网站。比如，*如果用户名为 chrisdavidmills*, 请转到 [chrisdavidmills.github.io](http://chrisdavidmills.github.io/)。

> ### 什么是Node.js

node.js 一种javascript的运行环境，能够使得javascript能够脱离浏览器运行。以前js只能在浏览器基础上运行，能够操作的也知识浏览器，比如浏览器上的放大缩小操作，前提是浏览器开启的基础上进行操作（浏览器是客户端）。有了node.js之后，js可以在服务端进行操作，直接在系统上进行操作，可以打开、关闭浏览器等操作。

简单的说 Node.js 就是运行在服务端的 JavaScript。Node.js 是一个基于Chrome JavaScript 运行时建立的一个平台。Node.js是一个事件驱动I/O服务端JavaScript环境，基于Google的V8引擎，V8引擎执行Javascript的速度非常快，性能非常好。


> ### npm 是世界上最大软件包仓库

最后来聊 npm 。Node.js 引发了前后端开发的爆发，尤其是前端。 JS 开发者众多，所以贡献开源代码的人就非常多，所有这些凝结成了 npm 这个世界上最大的软件包仓库。

npm 是 Node Package Manager 的缩写，意思是 Node 的包管理系统。Nodejs 现在如日中天，其中 npm https://www.npmjs.com/ 这个功不可没。在这里，我们要实现各种功能几乎都能找到现成的别人写好的包，直接拿了用就好了。

很多 npm 包都对应一个 Github 项目，但是如果只有代码，那么使用起来还不是特别方便。而当系统上安装好了 Node.js 之后，就会配套安装一个命令，叫做 npm 。

npm install moment
执行 npm install moment 就可以把 moment 这个包从 npm 的软件包仓库中下载这个包，然后安装到本地了。而 npm 的软件包仓库中，有数以万计的 moment 这样的包。

关于 npm ，我们就聊到这里。

