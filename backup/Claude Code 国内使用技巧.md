1、Claude Code安装
1.1、先安装一个hermes agent 
        因为我们在自己安装的时候很容易不成功，所以这一步让Agent来帮我们全自动安装，全程不用自己动手。复制这段文字给hermes agent：帮我在windows主机上安装Claude Code，遇到问题自行安装依赖或解决问题。
2、模型设置接入
2.1、安装 cc switch
        我们首先要安装一个插件来跳过Claude Code的登录验证，并且可以接入任意的第三方模型这个插件很强大，叫cc switch ,你可以打开cc switch页面直接安装对应的版本。
2.2、cc switch 配置
        安装cc switch后打开，点击左上角设置
        下滑找到并开启“跳过claude code 初次安装确认”
        接着点击右上角的加号，就可以配置对应的模型API了。
3、开启Claude Code默认跳过权限申请
     在cc switch配置模型的JSON配置里点击编辑通用配置，复制已下内容
`
{
  "permissions": {
    "defaultMode": "bypassPermissions"
    }
}
`
替换文本框里的所有内容，然后点保存即可。
4、如何指定Claude Code工作文件夹
     打开终端，输入cd ,按一下空格，然后把你的文件夹直接拖拽到终端窗口，回车，这时候终端的工作目录就变成了你指定的文件夹，然后输入claude，这时候就能以你指定的文件夹作为工作文件夹了。