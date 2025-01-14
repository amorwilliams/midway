---
title: MNS 触发器（消息队列）
---

:::info
请务必注意，阿里云消息队列会对 Topic 和 Queue 产生一定的费用。
:::

## 使用方式

```typescript
import { Provide, Inject, ServerlessTrigger, ServerlessTriggerType } from '@midwayjs/decorator';
import { Context, FC } from '@midwayjs/faas';

@Provide()
export class HelloAliyunService {
  @Inject()
  ctx: Context;

  @ServerlessTrigger(ServerlessTriggerType.MQ, {
    topic: 'test-topic',
    tags: 'bbb',
  })
  async handleMNSEvent(event: FC.MNSEvent) {
    // ...
  }
}
```

在 `npm run deploy` 后，即可。

:::info
注意，在阿里云下，midway faas 提供的默认消息队列格式为 JSON
:::

## MNS 触发器配置

| 属性名   | 类型                                        | 描述                                                         |
| -------- | ------------------------------------------- | ------------------------------------------------------------ |
| topic    | string                                      | 接收消息的 topic                                             |
| tags     | string                                      | 可选，描述了该订阅中消息过滤的标签（标签一致的消息才会被推送） |
| strategy | 'BACKOFF_RETRY' \|'EXPONENTIAL_DECAY_RETRY' | 调用函数的重试策略，可选值：BACKOFF_RETRY, EXPONENTIAL_DECAY_RETRY, 默认值为: BACKOFF_RETRY |
| region   | string                                      | 可选，topic 所在的 region，如果不填，默认为和函数一样的 region |

示例：

**监听 MQ 消息**

```typescript
@ServerlessTrigger(ServerlessTriggerType.MQ, {
  topic: 'test-topic',
  region: 'cn-shanghai'
  strategy: 'BACKOFF_RETRY'
})
```

## 事件结构

MNS 消息返回的结构如下，在 `FC.MNSEvent` 类型中有描述。

```json
{
  "Context": "user custom info",
  "TopicOwner": "1186202104331798",
  "Message": "hello topic",
  "Subscriber": "1186202104331798",
  "PublishTime": 1550216302888,
  "SubscriptionName": "test-fc-subscibe",
  "MessageMD5": "BA4BA9B48AC81F0F9C66F6C909C39DBB",
  "TopicName": "test-topic",
  "MessageId": "2F5B3C281B283D4EAC694B7425288675"
}
```

## 本地开发

事件类型的函数本地无法使用 dev 开发，只能通过运行 `npm run test` 进行测试执行。

## 本地测试

和 HTTP 测试不同，通过 `createFunctionApp` 创建函数 app，通过 `getServerlessInstance` 获取整个类的实例，从而调用到特定方法来测试。

可以通过 `createMNSEvent` 方法快速创建平台传入的结构。

```typescript
import { createFunctionApp, close } from '@midwayjs/mock';
import { Framework, Application } from '@midwayjs/serverless-app';
import { HelloAliyunService } from '../src/function/hello_aliyun';
import { createMNSEvent, createInitializeContext } from '@midwayjs/serverless-fc-trigger';
import { join } from 'path';

describe('test/hello_aliyun.test.ts', () => {
  let app: Application;
  let instance: HelloAliyunService;

  beforeAll(async () => {
    // create app
    app = await createFunctionApp<Framework>(join(__dirname, '../'), {
      initContext: createInitializeContext(),
    });
    instance = await app.getServerlessInstance<HelloAliyunService>(HelloAliyunService);
  });

  afterAll(async () => {
    await close(app);
  });

  it('should get result from oss trigger', async () => {
    expect(await instance.handleMNSEvent(createMNSEvent())).toEqual('hello world');
  });
});
```
