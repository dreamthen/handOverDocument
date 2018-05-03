# 请求文档

> Boss项目使用的是axios xhr请求工具,统一对xsrf、请求、响应、异常和错误分别提供属性或者回调函数,并对异步问题进行解决处理。
    
    import axios from "axios";
    import * as uiUtils from "../helpers/uiUtils";
    //在根目录底下的src/configs/axiosConfig.js文件中
    //首先对xsrf进行配置,返回一个配置好xsrf的axios
    const bossAxios = axios.create({
        //获取到请求头部的header头部中的xsrf-token属性值
        xsrfHeaderName: "xsrf-token",
        //获取到Cookie信息存储工具中的xsrf-token属性值
        xsrfCookieName: "xsrf-token"
    });
    
    //用来判别是否是get请求的标识符
    const GET = "GET";
    
    /**
      * 暴露axiosConfig对象,具体的配置是在暴露的对象中的axiosRequest方法中
      * url 请求的接口地址
      * method 请求方式
      * data get or post请求的参数
      * done 回调函数,fulfilled请求成功之后的状态管理等其他操作
      * headers 请求头部配置
      * isForm 是否为Form表单请求
      * error 回调函数,rejected异常处理,显示提示语等其他操作
      * catchError 回调函数,错误处理,后台50x错误,404接口查询不到,301重定向问题等其他操作
      * loading 是否显示全局loading gif加载图片
      */
    axiosRequest(url, method, data, done, headers, isForm, error, catchError, loading) {
        bossAxios({
            url,
            method,
            //params用于请求方式为get或者Form表单请求时,请求的参数。
            params: (method.toUpperCase() === GET || isForm) ? data : {},
            //data用于请求方式为post时,请求的参数,假如是Form表单请求,不请求post参数
            data: Object.assign({}, method.toUpperCase() === GET ? {} : isForm ? {} : data, loading),
            headers: Object.assign({}, headers ? headers : {}, {
                "ContentType": "application/json",
                "X-Requested-With": "XMLHttpRequest"
            }),
            //返回数据类型为json类型
            responseType: "json",
            //是否设置请求的域与当前域在"same-origin"下,用来设置请求跨域
            withCredentials: true
        })
        //fulfilled请求成功之后的状态管理等其他操作,rejected异常处理,显示提示语等其他操作
        .then(done, error)
        //错误处理,后台50x错误,404接口查询不到,301重定向问题等其他操作
        .catch(catchError)
    }
    
    //接口地址请求统一处理
    bossAxios.interceptors.request.use(function resolve(request) {
        //获取到请求对象中的data参数对象
        let data = request.data;
        //获取到传递的loading属性,假如loading为true,就显示全局loading gif加载图片
        if (data.loading) {
            uiUtils.showLoading();
        }
        return request;
    }, function reject() {
        
    });
    
    //接口地址响应统一处理
    bossAxios.interceptors.response.use(function resolve(response) {
        //后台返回的主体信息部分
        let data = response.data,
            //后台返回的主体信息头部
            head = data.head,
            //后台返回的主体信息头部code码
            code = head.code,
            //后台返回的主体信息头部异常提示语
            msg = head.msg ? head.msg : head.message,
            //后台返回的主体信息的配置部分
            config = response.config,
            //获取到配置部分的请求参数isManager,isManager的作用是防止在开户页面后台做的会话过期重定向到首页的问题
            isManager = config.params && config.params.isManager,
            //后台返回的主体信息主体部分
            body = head.body;
        //隐藏全局loading gif加载图片
        uiUtils.hideLoading();
        //假如后台返回的主体信息头部code码为0000000时,说明请求响应成功,直接返回主体信息的主体部分
        if(head && code === "00000000") {
            return body;
        }
        
        //假如后台返回的主体信息头部code码为11111114且不是开户页面的情况时,说明会话过期,提示异常:登录已失效，请重新登录,重定向到首页
        if (head && code === "11111114" && !isManager) {
            uiUtils.warning("登录已失效，请重新登录");
            window.location.href = "./login.html";
        }
        
        //其他异常情况直接返回异常主体信息部分,与error回调函数,rejected异常处理相对应
        return Promise.reject(response);
        
     
    }
    //错误处理rejected,后台50x错误,404接口查询不到,301重定向问题等其他操作,提前于catchError回调函数处理错误,错误一般不会暴露到catchError回调函数中
    , function reject(response) {
        //隐藏全局loading gif加载图片
        uiUtils.hideLoading();
        //提示错误:服务器异常，请稍后重试
        uiUtils.warnings("服务器异常，请稍后重试");
        //错误情况直接返回错误主体信息部分
        return Promise.reject(response);
    });
    
    //导出axiosConfig对象
    export default axiosConfig;    