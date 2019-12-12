# 如何在roboApp部署storybook

> 大部分操作参考自 https://storybook.js.org/docs/basics/introduction/

官网已经写得很详细了，但是无奈我太笨，总是忘记自己做了些啥，最好还是记录下

最主要还是记录下过程

下载，安装，用最初的默认配置

阿咧不行？好像是因为组件是tsx的关系，因为走的并不是dev的webpackconfig，所以之前的loader都炸了？我不信

先删掉之前的东西再试一次吧

## 先安装storybook

自动化安装命令是这个，我就不管他参数到底是干嘛的了

~~~bash
npx -p @storybook/cli sb init --type react
~~~

这个过程有点长，请耐心等待

安装完成后多了几个dependence依赖

* "@babel/core": "^7.7.5",
* "@dyc/eslint-config-dyc-app": "^0.2.7",
* "@storybook/addon-actions": "^5.2.8",
* "@storybook/addon-links": "^5.2.8",
* "@storybook/addons": "^5.2.8",
* "@storybook/react": "^5.2.8",
* "babel-loader": "^8.0.6",

也多了两个命令：

* storybook
* build-storybook 这个我改成了 build:storybook

npm run storybook一下，ok没问题

## 试着导入一下components

~~~javascript
import React from 'react';
import Comp from 'components/Comp.tsx'; // 要加后缀的

export default {
  title: 'Comp',
};

export const testComp = () => <Comp />;

~~~

好吧报错了，Module parse failed: Unexpected token (17:4)，好像是识别不了jsx

把Comp.tsx换成js文件看看，嗯？竟然可以了，而且还不用加.js后缀了。

看来默认设置对ts的支持有点问题

## 让storybook支持ts

查了下storybook官网，让用storybook的present，好吧那就先试试

按照他的要求，添加了present.js，添加了库@storybook/preset-typescript

但是还是报错了，这次是ts-loader react-docgen-typescript-loader NOT find，emmmm...

先安装下ts-loader和react-docgen-typescript-loader试试

build失败了，无法

不走ts-loader了，试试手动安装，

之后又研究了一天，发现两个关键的东西

参考了这篇文章

> https://medium.com/@dandobusiness/setting-up-a-react-typescript-storybook-project-5e4e9f540568

~~~json
// .storybook/tsconfig.json

"skipLibCheck": true,
~~~

~~~javascript
config.module.rules.push({
  test: /\.(ts|tsx)$/,
  // include: [COMPONENTS_PATH],
  use: [
    {
      loader: require.resolve('awesome-typescript-loader'),
      options: {
        configFileName: './.storybook/tsconfig.json',
      },
    },
    { loader: require.resolve('react-docgen-typescript-loader') },
  ],
});
~~~

设置完后跑是能跑起来了，但是.......

alias问题，antd的问题，还有我不想要的报错问题一大堆

## 事后

我感觉我自己是写不出来能够兼容所有配置的配置的，只能想办法让storybook去读喻哥写好的配置

我感觉我也得去研究下配置...要不然一脸懵逼都不知道自己在写什么

![一堆不明所以的报错](/Users/sijie.chang/workspace/@blog/imgs/一堆不明所以的报错.png)

感觉这个issue里应该能找到些线索

> https://github.com/storybookjs/presets/issues/65

还有这个mr

> https://github.com/storybookjs/storybook/pull/8517



## 逐步找到方案

> 照这个走！！！http://www.xefan.com/archives/84174.html 可以用！！！

> 这个pull需要babel-loader https://github.com/hugos29dev/the-Project-Alpha/pull/18

## 总结

经过了4天的奋斗，也许最后的代码不多，但是这次我真的学到了很多东西，tsconfig，babel，webpack配置，我都或多或少看了一点点，感慨自己效率还是太低，这么多天只干了这么点事情。

可能最后还会出点问题，比如某某控件不支持啦，ts又罢工啦～，这些都不管了，反正能跑起来了，后面接着优化就可以了。

果然去百度查问题就是自暴自弃，以后要善用谷歌了

来说一下怎么配置的吧

首先纯净安装一下react-storybook

~~~bash
npx -p @storybook/cli sb init --type react
~~~

然后在 `.storybook` 下面创建 `webpack.config.js` 文件，在里面填入下面的代码

~~~javascript
module.exports = ({ config }) => {
  config.module.rules.push(
    {
      test: /\.(ts|tsx)$/,
      loader: require.resolve('babel-loader'),
      options: {
        presets: [['react-app', { flow: false, typescript: true }]],
      },
    },
    {
      test: /\.less$/,
      use: [
        { loader: 'style-loader' },
        { loader: 'css-loader' },
        {
          loader: 'less-loader',
          options: {
            // modifyVars: antdTheme, // 如果要自定义主题样式
            javascriptEnabled: true,
          },
        },
      ],
    }
  );
  config.resolve.extensions.push('.ts', '.tsx');

  return config;
};
~~~

然后安装下依赖

~~~bash
npm install -D babel-loader style-loader css-loader less-loader
~~~

别问我为什么要装css的东西，问就是antd坑

然后把 `xxx.stories.js` 改成 `xxx.stories.tsx  ` 就可以了

完了

没了

什么就这么点东西花了4天？？？

喂，我为了查错你知道我经历了些什么吗，（其实中间有段时间烦得不行去刷掘金了233）

嘛，总之可喜可贺可喜可贺，有时间把这篇文章整理下吧（虽然没什么内容）