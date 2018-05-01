# 打包文档

> Boss项目是由create-react-app脚手架生成,由webpack打包,分为开发环境,测试环境和生产环境,分别对应package.json中的npm的三个命令。
> 1. 首先从gitlab上拉取代码,git clone 链接地址 -b 版本名称
> 2. 然后使用npm i/npm install或者使用yarn/yarn install拉取外部依赖包。
> 3. 开发环境是用webpack-dev-server本地服务器搭建而成的,命令为: npm start/yarn start。
> 4. 测试环境是用webpack无压缩(为了提高打包速度)打包而成的,命令为: npm run pre/yarn run pre
> 5. 生产环境是用webpack打包而成(包括js ES6语法编译压缩,css打包,html打包,图片文字打包,为了防止重复引入第三方依赖包,导致打包体积过大,打包速度过慢,打包生成manifest json api),命令为: npm run build/yarn run build
> 6. 开发环境webpack-dev-server默认端口号为: 3001,假如要与开发环境的后台进行连调或者是启用前端服务给后台使用,需要与nginx进行配合,nginx配置如下:
    
    upstream csn_boss {
        server 后台的boss服务器ip:后台的boss服务器端口号;
    }
    
    upstream csn_upms {
        server 后台的upms服务器ip:后台的upms服务器端口号;
    }
    
    server {
        listen 本地端口号;
        server_name 随便起一个前端本地服务器名字;
        
        location / {
            proxy_pass http://localhost:3001;
            # 还有一些请求头部,请求时间的配置,这里不一一描述
            # ...
            # ...
        }
        
        //用于拦截对boss服务器的请求
        location ~^ /xqy-portal-web/boss/ {
            proxy_pass http://csn_boss/xqy_boss/boss/;
            # 还有一些请求头部,请求时间的配置,这里不一一描述
            # ...
            # ...
        }
        
        //用于拦截对upms服务器的请求
        location ~^ /xqy-portal-web/upms/ {
            proxy_pass http://csn_upms/xqy_upms/upms/;
            # 还有一些请求头部,请求时间的配置,这里不一一描述
            # ...
            # ...
        }
    }
    
> 7. 测试环境和生产环境都可在本地生成打包文件,我称之为"稳定的环境",与后台进行联调或者是启用前端服务给后台使用时,最好不要使用开发环境,由于开发环境需要研发人员修改,且修改是热加载,每一次修改都会引起页面的刷新,所以使用本地的生产环境或者测试环境更佳,在本地打包后会生成build打包目录。不过需要修改package.json中的homePage属性,将"/maintain"更置为"",由于在真正的生产环境和测试环境,外层都有一层maintain文件夹,而本地没有,所以在本地要去掉"/maintain",而提交测试环境、release环境和生产环境代码时,需要将"/maintain"加上。nginx配置如下:

    upstream csn_boss {
        server 后台的boss服务器ip:后台的boss服务器端口号;
    }
    
    upstream csn_upms {
        server 后台的upms服务器ip:后台的upms服务器端口号;
    }
    
    server {
        listen 本地端口号;
        server_name 随便起一个前端本地服务器名字;
        
        location / {
            root E:/项目文件夹名/build/;
            index index.html index.htm;
            # try_files $uri /index.html;
        }
        
        //用于拦截对boss服务器的请求
        location ~^ /xqy-portal-web/boss/ {
            proxy_pass http://csn_boss/xqy_boss/boss/;
            # 还有一些请求头部,请求时间的配置,这里不一一描述
            # ...
            # ...
        }
        
        //用于拦截对upms服务器的请求
        location ~^ /xqy-portal-web/upms/ {
            proxy_pass http://csn_upms/xqy_upms/upms/;
            # 还有一些请求头部,请求时间的配置,这里不一一描述
            # ...
            # ...
        }
    }
