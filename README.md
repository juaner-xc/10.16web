# Quiz 后台管理作业

这是一个使用 **Vue 2 + Element UI** 完成的 Quiz 管理后台，界面和课堂示例保持相近。

## 已完成功能

- 用户管理：查询、添加、编辑、删除、分页
- 题目管理：查询、添加、编辑、删除、分页
- 题目弹窗包含问题内容、A/B/C/D 四个选项和正确答案
- 默认题库与现有 Quiz 前台 `https://quiz-game-phi-sooty.vercel.app/` 的 8 道题对应
- 数据自动保存在浏览器 `localStorage`，刷新后不会消失
- 自适应手机与电脑屏幕

## 本地运行

先安装 Node.js，然后在项目目录运行：

```bash
npm install
npm run serve
```

浏览器打开终端显示的网址即可。

## 编译

```bash
npm run build
```

编译结果会生成在 `dist` 文件夹中。

## 上传 GitHub

不要上传 `node_modules` 和 `dist`；项目已经提供 `.gitignore`。

## 部署 Vercel

1. 将本项目上传到 GitHub。
2. 在 Vercel 新建项目并导入该仓库。
3. Vercel 会自动识别配置：构建命令为 `npm run build`，输出目录为 `dist`。
4. 点击 Deploy，等待部署完成。
