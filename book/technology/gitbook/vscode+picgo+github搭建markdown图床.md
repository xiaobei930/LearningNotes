# vscode+picgo+github 搭建 markdown 图床

vscode 作为轻量级 IDE 神器，在安装 Markdown All in One 作为 markdown 编辑器异常好用，唯一美中不足的是不能上传图片。
这里就要讲到图床插件 picgo

## picgo 介绍 ([github地址](https://github.com/PicGo/vs-picgo))

**You can change all the shortcuts below as you wish.**

| OS           | Uploading an image from clipboard               | Uploading images from explorer                  | Uploading an image from input box               |
| ------------ | ----------------------------------------------- | ----------------------------------------------- | ----------------------------------------------- |
| Windows/Unix | <kbd>Ctrl</kbd> + <kbd>Alt</kbd> + <kbd>U</kbd> | <kbd>Ctrl</kbd> + <kbd>Alt</kbd> + <kbd>E</kbd> | <kbd>Ctrl</kbd> + <kbd>Alt</kbd> + <kbd>O</kbd> |
| OsX          | <kbd>Cmd</kbd> + <kbd>Opt</kbd> + <kbd>U</kbd>  | <kbd>Cmd</kbd> + <kbd>Opt</kbd> + <kbd>E</kbd>  | <kbd>Cmd</kbd> + <kbd>Opt</kbd> + <kbd>O</kbd>  |

## 特性

<details>
<summary>Uploading an image from clipboard</summary>
<img src="https://i.loli.net/2019/04/09/5cac17d2d2265.gif" alt="clipboard.gif">
</details>

<details>
<summary>Uploading images from explorer</summary>
<img src="https://i.loli.net/2019/04/09/5cac17eea0d65.gif" alt="explorer.gif">
</details>

<details>
<summary>Uploading images from input box</summary>
<img src="https://i.loli.net/2019/04/09/5cac17fe52a86.gif" alt="input box.gif">
</details>

<details>
<summary>Use selection text as the uploaded </summary>
<img src="https://i.loli.net/2019/04/09/5cac180fb1dc7.gif" alt="selection.gif">
<b>Notice: These characters: <code>\$</code>, <code>:</code>, <code>/</code>, <code>?</code> and newline will be ignored in the image name. </b>(Because they are invalid for file names.)
</details>

## github 图床优点

1. 稳定，用到微软破产应该不是问题
2. 不花钱，这点很赞
3. 容量大，一个仓库的上限是 100G，用作图床是够用了
4. 用了 cdn 加速之后速度还是可以的

## github 图床设置

1. 注册登录 Github
   略
2. 创建**public**仓库
   略
3. 进入个人设置，选择开发者设置
   settings-->Developer settings-->Personal access tokens,生成新的 token
   ![vscode+picgo+github搭建markdown图床-gitbook+pic搭建图床1-2020-06-18](https://cdn.jsdelivr.net/gh/xiaobei930/LearningNotes@master/assets/images/vscode+picgo+github搭建markdown图床-gitbook+pic搭建图床1-2020-06-18.png)
4. 添加描述，勾选 repo
   ![vscode+picgo+github搭建markdown图床-2](https://cdn.jsdelivr.net/gh/xiaobei930/LearningNotes@master/assets/images/vscode+picgo+github搭建markdown图床-2.png)
5. 保存生成的 token **关闭页面将无法再次看到**
   ![vscode+picgo+github搭建markdown图床-3](https://cdn.jsdelivr.net/gh/xiaobei930/LearningNotes@master/assets/images/vscode+picgo+github搭建markdown图床-3.png)

## picgo 配置

GitHub 设置完之后，需要修改 vscode 的 picgo 插件的设置，配置刚才设置的 github 图床，具体设置如下：

1. vscode 右下角或者 ctrl+，唤出设置页面
   ![vscode+picgo+github搭建markdown图床-4](https://cdn.jsdelivr.net/gh/xiaobei930/LearningNotes@master/assets/images/vscode+picgo+github搭建markdown图床-4.png)
2. 在扩展打开或者搜索 picgo
   ![vscode+picgo+github搭建markdown图床-5](https://cdn.jsdelivr.net/gh/xiaobei930/LearningNotes@master/assets/images/vscode+picgo+github搭建markdown图床-5.png)
3. 具体设置如图
   ![vscode+picgo+github搭建markdown图床-6](https://cdn.jsdelivr.net/gh/xiaobei930/LearningNotes@master/assets/images/vscode+picgo+github搭建markdown图床-6.png)

**配置说明：**

1. **current**选择 GitHub
2. **Branch**为仓库的分支，默认为 master
3. **custom url** 为图片上传的连接，有多种方式可以使用，我们这里用的是 cdn 加速方式
   1. 格式为https://cdn.jsdelivr.net/gh/[用户名]/[仓库名]@[分支名]
   2. 本文https://cdn.jsdelivr.net/gh/xiaobei930/PicBed@master
4. **path**为图片存储在仓库中的路径，比如我的是 learningNotes/pictures/

设置好后就可以愉快的在 vscode 里插图片了！
