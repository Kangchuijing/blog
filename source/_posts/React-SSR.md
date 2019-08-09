---
title: React_SSR
date: 2017-08-09 17:53:13
tags: React SSR
---

##### 目录结构
    
    client/      // 客户端打包生成的文件夹
        css/
        index.html
        index.css
        
    public/     // 客户端静态资源文件夹
        index.html  // html模版文件
        
    server/     // 服务端打包生成的文件夹
        server.js
```
import express from "express";
import path from "path";
import fs from "fs";

const app = express();

app.use(
  express.static(path.resolve(__dirname, "../client"), {
    index: false        // 防止直接将客户端的index.html发送给前端
  })
);

import React from "react";
import { renderToString } from "react-dom/server";
import {StaticRouter} from 'react-router-dom';
import App from "./App";
app.get("/*", (req, res, next) => {
  let context = {css: []};
  let content = renderToString(
      <StaticRouter context={context} location={req.url}>
          <App />
      </StaticRouter>
  );
  let cssStr = context.css.length ? context.css.join('\n') : '';
  try {
    let template = fs.readFileSync(path.join(process.cwd(), "/client/index.html"), 'utf8');
    template = template.replace(/\/\* css insert here \*\//, cssStr);
    template = template.replace(/<!-- js insert here -->/, content);
    res.status(200).send(template);
  } catch (error) {
    console.log(error);
    res.status(500).send({
      code: "500",
      msg: "系统错误，请稍后重试！"
    });
  }
});
app.listen(8900, () => {
  console.log("服务启动成功，端口号是8900！");
});

```
> 注意 `fs` 读取文件时的路径问题！！！！

```
__dirname：    获得当前执行文件所在目录的完整目录名
__filename：   获得当前执行文件的带有完整绝对路径的文件名
process.cwd()：获得当前执行node命令时候的文件夹目录名 
./：           文件所在目录
```