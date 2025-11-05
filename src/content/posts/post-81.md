---
title: "解决 NestJS 项目中接口跨域问题的三种方案"
published: 2024-07-22
updated: 2024-07-22
description: ""
tags: ["NestJs"]
category: "后端相关"
draft: false
---

NestJs允许我们以非常直观的方式启用CORS支持。你可以全局启用它，也可以为特定的路由启用。下面让我们一步一步来看如何操作。

**方法一：全局启用CORS**
在你的 main.ts文件中，你可以在创建Nest应用时启用CORS：

    import { NestFactory } from \'@nestjs/core\';
    import { AppModule } from \'./app.module\';
    
    async function bootstrap() {
      const app = await NestFactory.create(AppModule);
    
      // 启用CORS
      app.enableCors();
    
      await app.listen(3000);
    }
    bootstrap();

这将允许所有不同源的客户端请求，没有任何限制。对于某些情况来说，这可能太宽泛了，因为这样设定后，任何网站都可以访问你的服务。

如果你想限制特定的源能够访问服务，可以传入一个选项对象：

    app.enableCors({
      origin: \'<http://example.com>\', // 只有来自 <http://example.com> 的请求才被允许
    });

还可以传入更多的配置选项，例如允许的HTTP请求方法、响应中包含的头信息等。

**方法二：为特定路由启用CORS**
如果你只希望为特定的路由启用CORS，你可以在对应的Controller上使用 @UseCors装饰器。例如：

    import { Controller, Get, UseCors } from \'@nestjs/common\';
    
    @Controller(\'cats\')
    export class CatsController {
      @Get()
      @UseCors({
        origin: \'<http://example.com>\',
      })
      findAll() {
        // ...
      }
    }

这将确保只有在访问 /cats路由时，才会启用CORS，并且只有来自 http://example.com的请求可以访问。

**方法三：使用中间件来自定义CORS**
如果你需要进行更为复杂的CORS配置，或者你希望有完全的控制权，你可以使用 cors这个NPM包，它提供了丰富的配置选项。这个方法也适用于你想对某些路由使用不同的CORS策略的情况。

首先，你需要安装 cors：

    npm install cors

然后，在你的代码中引入并使用它：

    import { NestFactory } from \'@nestjs/core\';
    import { AppModule } from \'./app.module\';
    import * as cors from \'cors\';
    
    async function bootstrap() {
      const app = await NestFactory.create(AppModule);
    
      // 使用自定义CORS中间件
      app.use(cors({
        origin: \'<http://my-mobile-app.com>\',
        // 更多的配置选项...
      }));
    
      await app.listen(3000);
    }
    bootstrap();

这将允许你把 cors作为一个函数调用并传递配置对象，根据需要进行详细的配置。

高级配置
对于一些更高级的配置，NestJs允许直接传递CORS配置到 NestFactory的 create方法中。例如：

    const app = await NestFactory.create(AppModule, {
      cors: {
        origin: \'<http://example.com>\',
        methods: \'GET,HEAD,PUT,PATCH,POST,DELETE\',
        allowedHeaders: \'Content-Type, Accept\',
        credentials: true,
      },
    });

上面的配置中，我们指定了允许请求的方法、允许的请求头，以及是否允许携带凭证（如cookies）。

**注意事项**
安全性：在配置CORS时，不推荐使用通配符()来允许所有来源，因为这可能会使你的应用更容易受到跨站请求伪造（CSRF）或其他攻击。
环境配置：实际项目中，不同的环境（开发、测试、生产）可能需要不同的CORS设置。推荐将CORS设置存储在环境变量中，并根据当前的运行环境载入。

    app.enableCors({
      origin: process.env.CORS_ORIGIN || \'*\', // 生产环境应指定具体的域名
      methods: process.env.CORS_METHODS || \'GET,HEAD,PUT,PATCH,POST,DELETE\',
      allowedHeaders: process.env.CORS_HEADERS || \'Content-Type, Accept\',
      credentials: process.env.CORS_CREDENTIALS === \'true\', // 请确保实际字符串值是\'true\'或者\'false\'
    });

测试: 一旦你配置了CORS，确保你进行了适当的测试，包括从不同的源发起请求，确保配置按预期工作。

转载自https://blog.csdn.net/m0_37890289/article/details/135469118
