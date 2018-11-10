---
title: MPVUE和node的ENV的问题
date: 2018-11-10 16:50:59
categories:
 - mpvue
tags:
 - mpvue
 - node
 - env
---
小程序选定了mpvue作为开发框架，搭建开发环境和构建环境。自从用了Travis和Jenkins之后，再也回不到手工构建的时代了。

# 目的－自动构建
    web项目中，自从前后台分离的结构形成，就形成了一个要求，前后台的连接URL需要根据部署环境进行切换，线上的URL和测试的URL肯定不同；这点类似于java应用连接数据库的连接切换。
    后台URL的定义要能够多环境构建是自动构建的目标。
# 构建简介
    mpvue的框架基于vue.js构建，利用webpack的扩展工具将vue源码转换为小程序的源码。vue的源码是基于node构建的，理论将node构建生态的env模式也能带入mpvue构建过程。
    process.env是nodejs提供的官方api。官方定义是：__process.env属性返回一个包含用户环境信息的对象。__

# process.env
    process.env.NODE_ENV是默认的全局定义的全局变量.process.env是一个定义对象，可以自定义扩展。
    比如：
    ``` js
    process.env = {
        NODE_ENV : 'dev',
        api : 'http://example.com'
    }
    ```
    这样子就实现了自定义的过程，将定义分别放到env.dev.js,env.prod.js,env.test.js即可实现多环境实践。
# mpvue中使用
    mpvue的quickStart提供的构建脚手架，env的定义在config目录中，通过prod.env.js和dev.env.js实现对env的定义。
    ``` js prod.env.js
        module.exports = {
            NODE_ENV: '"production"',
            api: '"example.com"'
        }
    ```
    如何使用呢？
    因为process是一个node的全局变量，使用Process对象在vue源码中应该是任意使用的。测试下：
    ``` js 
    // App.vue
    <script>
        export default {
            created () {
                // 调用API从本地缓存中获取数据
                const logs = wx.getStorageSync('logs') || []
                logs.unshift(Date.now())
                wx.setStorageSync('logs', logs)
                console.log('app created and cache logs by setStorageSync')
            }
        }
    </script>
    //pages/index/index.vue
     <script>
        export default {
            methods: {
                clickHandle (msg, ev) {
                    console.log(process.env)
                    console.log('clickHandle:', msg, ev)
                }
            }
        }
    </script>
    ```
    启动构建工具，打开微信开发工具，可以正确输出定义结果，目标达成。