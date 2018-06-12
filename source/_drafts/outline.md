---
title: 大纲
categories:
 - 其他
date: 2018-05-09 09:41:47
---

1.  前后端分离
    后端专注于：后端控制层（Restful API） & 服务层 & 数据访问层；
    前端专注于：前端控制层 & 视图层

    -   项目设计阶段，前后端架构负责人将项目整体进行分析，讨论并确定 API 风格、职责分配、开发协助模式，确定人员配备；设计确定后，前后端人员共同制定开发接口。
    -   项目开发阶段，前后端分离是各自分工，协同敏捷开发，后端提供 Restful API，并给出详细文档说明，前端人员进行页面渲染前台的任务是发送 API 请(GET,PUT,POST,DELETE 等)获取数据（json，xml）后渲染页面。
    -   项目测试阶段，API 完成之前，前端人员会使用 mock server 进行模拟测试，后端人员采用 junit 进行 API 单元测试，不用互相等待；API 完成之后，前后端再对接测试一下就可以了，当然并不是所有的接口都可以提前定义，有一些是在开发过程中进行调整的。

2.  前端框架分析
    主流三大框架：[优缺点分析，适用场景选择 ](https://juejin.im/post/5a0d5df1f265da43062a542f)
    React Vue.js AngularJS

3.  Yarn 代替 NPM

4.  PWA ‘代替’ 原生 App
    PWA（Progressive Web App）是 Google 于 2016 年提出的概念，2017 年已被迅速采用。PWA 旨在增强 Web 体验，可显著提高加载速度、可离线工作、可被添加至主屏、全屏执行、推送通知消息等等。这些特性将使得 Web 应用渐进式接近原生 App。

    相关阅读： [您的第一个 Progressive Web App](https://developers.google.com/web/fundamentals/codelabs/your-first-pwapp/?hl=zh-cn)、[Chrome Web App 已被谷歌干掉 未来将主推 PWA](https://www.oschina.net/news/91277/google-removes-chrome-apps-from-chrome-web-store)

5.  [GraphQL](https://www.graphql.com/)
    GraphQL 并不是一个面向图数据库的查询语言，而是一个数据抽象层，包括数据格式、数据关联、查询方式定义与实现等等一揽子的东西。GraphQL 也并不是一个具体的后端编程框架，如果将 REST 看做适合于简单逻辑的查询标准，那么 GraphQL 可以做一个独立的抽象层，通过对于多个 REST 风格的简单的接口的排列组合提供更多复杂多变的查询方式。与 REST 相比，GraphQL 定义了更严格、可扩展、可维护的数据查询方式。

6.  [sketch](https://www.sketchapp.com/)
    草图 & 线框图：简单，速度快 ---> 适用逻辑思维较强的 pm
    skecht：直观，明了，减少后期 ui 来回修稿的成本，在制作过程中 pm 就能发现逻辑的不合理性

7.  app/web 泛滥的时代，如何在交互上抓住用户的眼球 [principle](http://principleformac.com/)
    div + table ---> html5 + css3
    百分比布局 ---> 响应式
    2D 的页面 ---> 3/4D 的界面

    用户一直在满足基本需求的基础上寻求更好(炫)的用户体验，好的用户体验是产品的根本，'炫'技提高用户口碑

    炫技 or 过炫？ 过度的效果、动画加重载体负荷

    [交互分享实例库](http://principleux.com/)
