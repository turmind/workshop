# MSK Evnet

## Health Dashboard 查看事件信息

- 通过health dashboard的Event log栏目，可以查看到近期的相关事件
- 通过搜索栏筛选，可以看到相关的近期的KAFKA的事件
- 从事件中，可以查看相关的升级维护事件，以及涉及到的更改

![KAFKA Event](./img/MSKEvent/Screen%20Shot%202022-07-22%20at%203.22.21%20PM.png)

## 部署企业微信通知

- 通过解决方案链接，可以简单部署相关需要的解决方案，见地址[serverless-alert-notifier](https://www.amazonaws.cn/solutions/serverless-alert-notifier/)，直接通过点击从亚马逊科技海外区域控制台启动方案，按步骤点击即可
- 部署过程中，需要相关的企业微信的信息，可以通过[部署指南](https://aws-gcr-solutions.s3.amazonaws.com/serverless-alert-notifier/v1.0.1/docs.pdf)获取
- 无服务通知部署在是Region级别，对于不同Region的通知需要在不同的Region中部署

![解决方案](./imag/../img/MSKEvent/Screen%20Shot%202022-07-22%20at%203.32.39%20PM.png)

## 增加健康事件监控

- 通过event bridge中新建rule，增加MSK的相关通知，创建中相关的步骤参考以下：

![创建rule](./img/MSKEvent/Screen%20Shot%202022-07-22%20at%203.36.43%20PM.png)

- 设置kafka的健康信息监听
![监听kafka信息](./img/../img/MSKEvent/Screen%20Shot%202022-07-22%20at%203.37.26%20PM.png)

- 设置需要发送的的sns的topic,该topic由上一步的部署中创建

![设置策略](./img/MSKEvent/Screen%20Shot%202022-07-22%20at%203.38.04%20PM.png)

## MSK升级导致业务访问失败

- MSK升级时，理论上也是具有高可用性的，在重复因子和链接上需要多考虑，参考链接<https://docs.aws.amazon.com/msk/latest/developerguide/bestpractices.html#ensure-high-availability>
