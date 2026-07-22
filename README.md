# 项目说明

本项目是OnePrinter打印软件的Vue3版演示。

本项目演示了通过前端调用OnePrinter的2个api接口实现批量静默打印的整个过程，可以直接挪到项目中使用。

本项目适用于订单、票据、小票、标签、处方、二维码……等一切需要通过web端或者其他软件发起的打印需求。

# 前置条件

必须在windows10及以上电脑上安装OnePrinter，并确保可以运行，下载地址：

<https://docs.wangwei.ltd/guide/oneprinter/>

# 代码说明

1、通过onMounted调用OnePrinter的getPrinter接口获取到打印机列表；

2、使用html2canvas生成名片；

3、调用OnePrinter的print发送打印请求；

# 安装方法

## 安装

```sh
npm install
```

## 开发

```sh
npm run dev
```

## 打包

```sh
npm run build
```

# 作者信息

北京网为科技有限公司
<https://www.wangwei.ltd>
