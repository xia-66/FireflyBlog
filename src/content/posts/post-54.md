---
title: "Axios在Vue3中的封装使用及配置代理"
published: 2024-03-04
updated: 2024-09-20
description: ""
tags: ["Vite","Vue3","Axios"]
category: "前端相关"
draft: false
---

**在utils中新建request.ts文件进行封装**
    import axios from \"axios\";
    // import { BASEURL } from \"@/config/index\";
    //暴露出request函数
    export default (config: any) => {
      //创建axios实例
      const service: any = axios.create({
        // baseURL: `${BASEURL}`,
        baseURL: `/api`,
        //超时时间
        timeout: 12000,
      });

      //请求拦截
      service.interceptors.request.use(
        (config: any) => {
          //设置请求头
          config.headers[\"Content-Type\"] =
            config.headers[\"Content-Type\"] || \"application/json\";
          //每次请求带上token令牌
          if (config.url !== \"/WfwGetUserInfo\") {
            config.headers[\"Authorization\"] = sessionStorage.getItem(\"token\");
          }
          return config;
        },
        (error: any) => {
          return Promise.reject(error);
        }
      );

      //响应拦截：后端返回来的结果
      service.interceptors.response.use(
        (response: any) => {
          return response.data;
        },
        (error: any) => {
          return Promise.reject(error);
        }
      );
      return service(config);
    };
   **在apis文件夹下新建index.ts中模块化管理api**

    import request from \"@/utils/request\";
    // 微服务对接统一授权
    const WfwGetUserInfo = (code: string, state: string) => {
      return request({
        url: `/WfwGetUserInfo`,
        method: \"\",
        params: {
          code: code,
          state: state,
        },
        data: formdata,
      });
    };
**在具体页面中使用**

    WfwGetUserInfo().then((res) => {});

**在vite.config.ts中配置代理解决跨域问题**
 

    server: {
    // 配置host，局域网可访问
    host: \"0.0.0.0\",
    port: 8080,
    // 配置代理
    proxy: {
      \"/api\": {
        //target为本地后端端口
        target: \"http://127.0.0.1:97\",
        changeOrigin: true,
        // 若有https证书，即可将secure设为true
        secure: false,
        rewrite: (path) => path.replace(\"/api\", \"\"),
      },
     },
    },