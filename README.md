# Study-HarmonyOS
HarmonyOS application develop design material,鸿蒙应用开发学习资料

HarmonyOS开发工具:

DevEco Studio 3.1.1 Release

下载地址：https://developer.harmonyos.com/cn/develop/deveco-studio/#download
是idea的社区定制版，相当于Android Studio

Command Line Tools for HarmonyOS： 鸿蒙的命令行工具集合了HarmonyOS应用开发所用到的系列工具，
包括SDK管理sdkmgr、代码检查codelinter、三方库的包管理ohpm 相当于Android sdk manager


HarmonyOS开发语言：ArkTS

ArkTS是鸿蒙生态的应用开发语言。它在保持TypeScript（简称TS）基本语法风格的基础上，对TS的动态类型特性施加更严格的约束，
引入静态类型。同时，提供了声明式UI、状态管理等相应的能力，让开发者可以以更简洁、更自然的方式开发高性能应用。

文档地址：https://developer.harmonyos.com/cn/docs/documentation/doc-guides-V3/arkts-get-started-0000001504769321-V3

HarmonyOS/OpenHarmony应用开发-ArkTS语言声明式UI描述：https://developer.huawei.com/consumer/cn/forum/topic/0207121358743773095?fid=0101610563345550409

HarmonyOS页面开发框架：ArkUI

文档地址：https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V2/application-model-description-0000001493584092-V2

鸿蒙的UI框架 分为两种模式model

FA（Feature Ability）模型：HarmonyOS早期版本开始支持的模型，已经不再主推。

Stage模型：HarmonyOS 3.1 Developer Preview版本开始新增的模型，是目前主推且会长期演进的模型。在该模型中，
由于提供了AbilityStage、WindowStage等类作为应用组件和Window窗口的“舞台”，因此称这种应用模型为Stage模型。

Stage模型与FA模型最大的区别在于：Stage模型中，多个应用组件共享同一个ArkTS引擎实例；而FA模型中，每个应用组件
独享一个ArkTS引擎实例。因此在Stage模型中，应用组件之间可以方便的共享对象和状态，同时减少复杂应用运行对内存的占用。
Stage模型作为主推的应用模型，开发者通过它能够更加便利地开发出分布式场景下的复杂应用。

Ability和Stage相当于Android中的Activity

数据沙箱存储 相当于Android的 Sharedpreference

import dataPreferences from '@ohos.data.preferences';
import UIAbility from '@ohos.app.ability.UIAbility';

class EntryAbility extends UIAbility {
  onWindowStageCreate(windowStage) {
    try {
      dataPreferences.getPreferences(this.context, 'mystore', (err, preferences) => {
        if (err) {
          console.error(`Failed to get preferences. Code:${err.code},message:${err.message}`);
          return;
        }
        console.info('Succeeded in getting preferences.');
        // 进行相关数据操作
      })
    } catch (err) {
      console.error(`Failed to get preferences. Code:${err.code},message:${err.message}`);
    }
  }
}



请求一个权限：

import bundleManager from '@ohos.bundle.bundleManager';
import abilityAccessCtrl, { Permissions } from '@ohos.abilityAccessCtrl';

async function checkAccessToken(permission: Permissions): Promise<abilityAccessCtrl.GrantStatus> {
  let atManager = abilityAccessCtrl.createAtManager();
  let grantStatus: abilityAccessCtrl.GrantStatus;

  // 获取应用程序的accessTokenID
  let tokenId: number;
  try {
    let bundleInfo: bundleManager.BundleInfo = await bundleManager.getBundleInfoForSelf(bundleManager.BundleFlag.GET_BUNDLE_INFO_WITH_APPLICATION);
    let appInfo: bundleManager.ApplicationInfo = bundleInfo.appInfo;
    tokenId = appInfo.accessTokenId;
  } catch (err) {
    console.error(`getBundleInfoForSelf failed, code is ${err.code}, message is ${err.message}`);
  }

  // 校验应用是否被授予权限
  try {
    grantStatus = await atManager.checkAccessToken(tokenId, permission);
  } catch (err) {
    console.error(`checkAccessToken failed, code is ${err.code}, message is ${err.message}`);
  }

  return grantStatus;
}

async function checkPermissions(): Promise<void> {
  const permissions: Array<Permissions> = ['ohos.permission.READ_CALENDAR'];
  let grantStatus: abilityAccessCtrl.GrantStatus = await checkAccessToken(permissions[0]);

  if (grantStatus === abilityAccessCtrl.GrantStatus.PERMISSION_GRANTED) {
    // 已经授权，可以继续访问目标操作
  } else {
    // 申请日历权限
  }
}


加载一个webview

    // xxx.ets
    import web_webview from '@ohos.web.webview';

    @Entry
    @Component
    struct WebComponent {
      webviewController: web_webview.WebviewController = new web_webview.WebviewController();

      build() {
        Column() {
          Button('loadUrl')
            .onClick(() => {
              try {
                // 点击按钮时，通过loadUrl，跳转到www.example1.com
                this.webviewController.loadUrl('www.example1.com');
              } catch (error) {
                console.error(`ErrorCode: ${error.code},  Message: ${error.message}`);
              }
            })
          // 组件创建时，加载www.example.com
          Web({ src: 'www.example.com', controller: this.webviewController})
        }
      }
    }

发起一个网络请求：
    // 引入包名
    import http from '@ohos.net.http';

    // 每一个httpRequest对应一个HTTP请求任务，不可复用
    let httpRequest = http.createHttp();
    // 用于订阅HTTP响应头，此接口会比request请求先返回。可以根据业务需要订阅此消息
    // 从API 8开始，使用on('headersReceive', Callback)替代on('headerReceive', AsyncCallback)。 8+
    httpRequest.on('headersReceive', (header) => {
        console.info('header: ' + JSON.stringify(header));
    });
    httpRequest.request(
        // 填写HTTP请求的URL地址，可以带参数也可以不带参数。URL地址需要开发者自定义。请求的参数可以在extraData中指定
        "EXAMPLE_URL",
        {
            method: http.RequestMethod.POST, // 可选，默认为http.RequestMethod.GET
            // 开发者根据自身业务需要添加header字段
            header: {
                'Content-Type': 'application/json'
            },
            // 当使用POST请求时此字段用于传递内容
            extraData: {
                "data": "data to send",
            },
            expectDataType: http.HttpDataType.STRING, // 可选，指定返回数据的类型
            usingCache: true, // 可选，默认为true
            priority: 1, // 可选，默认为1
            connectTimeout: 60000, // 可选，默认为60000ms
            readTimeout: 60000, // 可选，默认为60000ms
            usingProtocol: http.HttpProtocol.HTTP1_1, // 可选，协议类型默认值由系统自动指定
        }, (err, data) => {
            if (!err) {
                // data.result为HTTP响应内容，可根据业务需要进行解析
                console.info('Result:' + JSON.stringify(data.result));
                console.info('code:' + JSON.stringify(data.responseCode));
                // data.header为HTTP响应头，可根据业务需要进行解析
                console.info('header:' + JSON.stringify(data.header));
                console.info('cookies:' + JSON.stringify(data.cookies)); // 8+
            } else {
                console.info('error:' + JSON.stringify(err));
                // 取消订阅HTTP响应头事件
                httpRequest.off('headersReceive');
                // 当该请求使用完毕时，调用destroy方法主动销毁
                httpRequest.destroy();
            }
        }
    );

 国际化开发基本和Android相同

    import I18n from '@ohos.i18n';
    try {
       let rtl = I18n.isRTL("zh-CN"); // rtl = false
       rtl = I18n.isRTL("ar"); // rtl = true
    } catch(error) {
       console.error(`call i18n.System interface failed, error code: ${error.code}, message: ${error.message}`);
    }

HarmonyOS 完整设计规范 文档地址：https://developer.huawei.com/consumer/cn/doc/design-guides/overview-0000001053563071

    通用设计基础 https://developer.huawei.com/consumer/cn/doc/design-guides/service-overview-0000001139795693
    分布式设计
    元服务设计
    AI 设计
    无障碍设计
    隐私设计
    全球化设计


HarmonyOS 鸿蒙学堂地址：https://hmxt.org/documents


