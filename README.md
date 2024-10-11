# tt-cocos-creator-sdk
## 提示

> 在接入前，请先阅读接入前准备
>
> 最新版本：v1.0.0 下载小游戏

------

## 一、集成SDK

下载小游戏CocosCreator SDK，并在代码中引入对应的SDK文件：
[下载小游戏CocosCreator SDK](https://github.com/ABetterChoice/cocos-creator-sdk/archive/refs/heads/master.zip)

如果您的项目是 TypeScript 工程，接入步骤如下：

1. 将声明文件 ABetterChoice.cc.d.ts 放入项目根目录下 assets 下级的 libs 目录，如果 libs 不存在，新建 libs 目录
2. 将 SDK 文件 (abetterchoice.mg.cocoscreator.min.js) 放入 assets/script 目录中

如果您的项目是 JavaScript 工程，您可以直接将 SDK 文件 (abetterchoice.mg.cocoscreator.min.js) 放入 assets/script 目录中

初始化SDK参数

```typescript
const config = {
  gameId: "YOUR_GAME_ID", // 项目游戏ID，必选，可以在ABetterChoice平台管理页查看
  apiKey: "YOUR_SECRET_KEY", // 项目API KEY，必选，可以在ABetterChoice平台管理页查看
  unitId: "login_unitId", // 登陆的用户帐号，必选，若不填，则在login之前都用访客ID做用户ID，可能会导致数据计算有误差
  autoTrack: {   // 可选，自动采集配置，默认全部关闭
    mgShow: true,  // 自动采集，小程序启动，或从后台进入前台，可选
    mgHide: true,  // 自动采集，小程序从前台进入后台，可选
  }
}
```

配置对象**Config**其他可选参数说明：

- **serverUrl**：可选，数据上报地址域名，默认是 https://data.abetterchoice.cn/。
- **attributes**:  可选，实验分流属性条件使用，类型为{ string : string[] }，其中string为条件属性，string[]为对应的条件属性值数组
- **enableAutoExposure**：可选，实验分流使用，默认值为false。如果设置为true，当调用AB实验分流时，曝光数据将自动上报。
- **enableAutoPoll**：可选，实验分流使用，默认值为true。如果设置为true，实验和功能标志数据将每10分钟轮询并更新。

**注意**：

> 在上报数据与实验分流之前，请先在微信公众平台或其他平台的开发设置中，将下面默认请求URL加入到服务器域名的 request 列表中，主要有：
>
> 分流URL：https://mobile.abetterchoice.cn
>
> 数据上报URL：https://data.abetterchoice.cn
>
> 若您使用的是私有化部署版本，请与运维同学确认上报地址

## 二、常用功能

在使用常用功能之前，请先知悉当前的用户生成规则，SDK默认会生成随机数作为访客ID，并持久化存储访客ID在本地；用户未登录之前，会以访客ID作为身份识别ID。若用户登陆之后，将采用登陆ID做为用户的唯一识别标志。注意:访客 ID 在用户清理缓存 以及更换设备时将会发生变更。

### 2.1 设置帐号ID

在用户进行登录时，可调用 `login` 来设置用户的账号 ID， SDK将会以账号 ID 作为身份识别 ID，并且设置的账号 ID 将会在调用 `logout` 之前一直保留。多次调用 `login` 将覆盖先前的账号 ID。

```typescript
// 用户的唯一标识，此数据对应上报数据里的user_id，此时user_id的值为ABC
ABetterChoice.login('ABC');
```

### 2.2 设置公共属性

公共事件属性指的就是每个事件都会带有的属性，您可以调用 `setSuperProperties` 来设置公共事件属性，我们推荐您在发送事件前，先设置公共事件属性。对于一些重要的属性，譬如用户的会员等级、来源渠道等，这些属性需要设置在每个事件中，此时您可以将这些属性设置为公共事件属性。

```typescript
var commonProperties = {
    vipSource: "ABC", //字符串
    vipLevel: 1,//数字
    isVip: true,//布尔
    birthday: new Date(),//对象
    object: { key: "value" },//对象
    object_arr: [{ key: "value" }],//对象组
    arr: ["value"]//数组
};
// 设置公共属性
ABetterChoice.setCommonProperties(commonProperties);
```

公共事件属性将会被保存到缓存中，无需每次启动时调用。如果调用 `setCommonProperties`上传了先前已设置过的公共事件属性，则会覆盖之前的属性。

- Key 为该属性的名称，为字符串类型，规定只能以字母开头，包含数字，字母和下划线 "_"，长度最大为 50 个字符。
- Value 为该属性的值，目前仅支持字符串。

### 2.3 发送事件

您可以调用 `track` 来上传事件，建议您根据先前梳理的埋点文档来设置事件的属性以及发送信息的条件，此处以用户购买某商品作为范例：

```typescript
ABetterChoice.track({
    eventName: "product_buy", // 事件名称
    properties: {
        product_name: "商品名"
    } //事件属性
});
```

### 2.4 获取AB实验

```typescript
// 获取实验分流信息
const experiment = ABetterChoice.getExperiment('abc_layer_name');
if (experiment === undefined) {
    // 无命中，执行默认版本
    handleView('无命中实验，执行默认版本');
    return;
}
// 现在您可以获取参数并在代码中直接使用这些参数。
const shouldShowBanner = experiment.getBoolValue("should_show_banner", true);
```

### 2.5 实验曝光

```typescript
// 当未设置enableAutoExposure时，您可以根据上面获取的实验分流信息进行手动记录曝光
ABetterChoice.logExperimentExposure(experiment);
```

### 2.6 获取配置开关

假设您在配置与开关模块中创建开关与配置流程中没有进行配置条件，那么配置与开关将按照配置的流量比例进行下发；

```typescript
// 获取配置开关参数名称为：new_feature_flag的配置开关值信息
const configInfo = ABetterChoice.getConfig("new_feature_flag");
// 获取对应配置开关的参数值,其中参数为默认值
const boolValue = configInfo?.getBoolValue(false);
// 其它根据平台配置的值类型进而获取不同的类型值
// const stringValue = configInfo?.getStringValue('banner');
// const numberValue = configInfo?.getNumberValue(1000);
```

若配置了条件，如下图创建所示，假设您创造了开关配置参数名称为'new_feature_flag'，配置条件配置参数属性为'city'与'age'，条件参数属性值为'shenzhen'与'18'，则满足这个条件则会进行下发布尔值true。

![IMG](https://cdn.abetterchoice.cn/static/cms/5640e1e9ac.jpeg)

```typescript
// 初始化SDK时需配置条件属性：attributes
var config = {
  .......
  .......
  // 配置条件属性
  attributes: {
    city: ['shenzhen'],
    age: ['18']
  }
}
......
......
// 获取配置开关参数名称为：new_feature_flag的配置开关值信息
const configInfo = ABetterChoice.getConfig("new_feature_flag");
// 获取对应配置开关的参数值,其中参数为默认值
const boolValue = configInfo?.getBoolValue(false);
```

## 三、最佳实践

[下载CocosCreator SDK](https://github.com/ABetterChoice/cocos-creator-sdk/archive/refs/heads/master.zip)，根据上面集成SDK的方式引入对应的文件之后，您就可以在代码中直接使用SDK功能，并开始上报数据了：

```typescript
const config = {
  gameId: "YOUR_GAME_ID", //项目游戏ID，必选，可以在ABetterChoice平台管理页查看
  secretKey: "YOUR_SECRET_KEY"，//项目API KEY，必选，可以在ABetterChoice平台管理页查看
  unitId: "YOUR_LOGGIN_USER_ID", //用户帐号ID，必选，若不填，则在login之前都用访客ID做用户ID，可能会导致数据计算有误差
  autoTrack: {   // 可选，自动采集配置
    mgShow: true,  // 自动采集，小程序启动，或从后台进入前台
    mgHide: true,  // 自动采集，小程序从前台进入后台
    mgShare: true, // 自动采集，小程序分享时自动采集
  }
};
// 打开SDK日志，默认关闭，上线前可关闭
ABetterChoice.setLogLevel(0);
// 初始化
await initResult = ABetterChoice.init(config);
if (initResult) {
  console.log('初始化结果：' + initResult);
}
// 用户的登录帐号唯一标识，此数据对应上报数据里的user_id，此时user_id的值为ABC，相当于配置初始化参数unitId，若上面配置初始化参数已填，可不使用
ABetterChoice.login('ABC');
// 设置公共事件属性
const commonProperties = {
    channel : "ta", //字符串
    age : 1,//数字
    isSuccess : true,//布尔
    birthday :  new Date(),//对象
    object : { key : "value" },//对象
    object_arr : [ { key : "value" } ],//对象组
    arr : [ "value" ]//数组
};
ABetterChoice.setCommonProperties(commonProperties);

// 清除公共事件属性
ABetterChoice.clearCommonProperties();

//发送事件
ABetterChoice.track(
    eventName: "product_buy", // 事件名称
    properties: {
        product_name: "商品名"
    } //事件属性
);

// 获取实验分流信息
const experiment = ABetterChoice.getExperiment('abc_layer_name');
// 获取实验配置的参数值并在代码中选择直接使用这些参数。
const shouldShowBanner = experiment.getBoolValue("should_show_banner", true);
if (shouldShowBanner) {
  handleView('执行显示Banner业务逻辑');
} else {
  handleView('执行不显示Banner业务逻辑');
}
// 根据上面获取到的实验信息主动进行曝光
ABetterChoice.logExperimentExposure(experiment);

// 获取配置开关名为：new_feature_flag的配置开关值信息
const configObj = ABetterChoice.getConfig(["new_feature_flag"]);
```

