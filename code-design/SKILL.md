---
name: code-design
description: Use when 在 Java/Spring 后端项目写或改业务代码之前——新增 Service/DTO/适配器/外部客户端/工具类、写实体到响应对象的组装转换、新业务接入流程引擎或审批流、对接第三方系统、以及 review 自己的 diff 时。适用于 Controller-Service-Mapper 等分层架构的中后台项目。覆盖单一职责(SRP)类设计、贫血/充血模型分层、设计模式选型、存量代码对齐纪律。
---

# 代码设计：职责纯粹的 Java 后端类设计

## 总纲（判断框架）

**一个类只回答一个问题；依赖方向单向；每层只见能力、不见细节。**

写码前先问：这个类该回答什么问题？职责之外的逻辑，向对应对象下沉或委托给对应层。这是单一职责原则（SRP）的可操作表述——不数「修改原因」，数「回答的问题」。

## 第 0 条：先找活例子，再动手

1. 写新代码前，先在仓库（或所依赖脚手架/框架的源码）里找**同构先例**对齐——一个能跑的活例子的引导力大于任何规范文字
2. 新增基建类（分布式锁/线程池/工具类/开关注解）前，先确认依赖的框架或脚手架是否已提供，不重复造轮子
3. **先例≠都对**：存量代码里混着好先例和差先例。逐字段 setter 灌值、同源散参长列表、信息三处重复的注释属于差先例，模仿前先对照下方分层职责表甄别——AI 和人都天然倾向照抄眼前代码，这是被差代码带偏的最大来源

## 分层职责表（每层只回答一个问题）

| 层 | 只回答 | 硬纪律 |
|----|--------|--------|
| Entity/DO | 持久化映射 | 保持贫血；充血会污染持久化职责 |
| DTO/Resp/VO | 展示组装 | 轻量充血：`Resp.of(entity, xxMap...)` 静态工厂承担组装；**绝不 import Service/Mapper/上下文工具**，保持纯 POJO |
| Service | 取数+编排 | 批量取数成 Map 喂给组装方法；不写过程式 setter 灌值 |
| Adapter | 协议翻译 | 只做事件路由/类型映射/格式转换/日志；业务逻辑一行委托 Service；实现细节不外泄给对接方 |
| 外部 Client | 能力封装 | 配置属性类只在实现内注入；「可不可用」的复合判断（开关+必填配置）封装为接口方法如 `isEnabled()`；业务层只见能力不见配置 |
| Registry/Factory | 发现分发 | 自动注册 + 冲突快速失败（重复注册抛异常），使用方零 if-else 分发 |
| Mapper/DAO | 数据访问 | 逻辑删除等全局过滤在自定义 SQL 中同样生效，别漏 |

## 一个完整例子（自包含）

需求：订单列表要展示下单人昵称和商品名（都需关联查询）。

```java
/** 展示组装内聚在 Resp，纯 POJO 不依赖任何基础设施 */
public static OrderResp of(OrderDO order, Map<Long, String> nicknameMap, Map<Long, String> productNameMap) {
    OrderResp resp = BeanUtil.toBean(order, OrderResp.class);
    // 外键可能为空的关联字段用 Optional 链兜底，不写 if 判空
    resp.setNickname(Optional.ofNullable(order.getUserId())
        .map(nicknameMap::get).orElse("未知用户"));
    resp.setProductName(Optional.ofNullable(order.getProductId())
        .map(productNameMap::get).orElse("-"));
    return resp;
}

/** Service 只做取数 + 编排：两个维度各批量查一次防 N+1，空集合短路防空 IN */
public List<OrderResp> listDetail(List<OrderDO> orders) {
    if (CollUtil.isEmpty(orders)) {
        return List.of();
    }
    Map<Long, String> nicknameMap = userMapper.listNicknameByIds(distinctNonNull(orders, OrderDO::getUserId));
    Map<Long, String> productNameMap = productService.listNameByIds(distinctNonNull(orders, OrderDO::getProductId));
    return orders.stream().map(o -> OrderResp.of(o, nicknameMap, productNameMap)).toList();
}
```

## 模式选型（新场景即查）

| 遇到什么 | 用什么 | 要点 |
|----------|--------|------|
| 可插拔能力（登录方式/存储后端/支付渠道） | 策略 + 模板方法 + 工厂 | 公共流程放抽象父类复用，新方式=加一个实现类 |
| 多业务类型接入同一流程（审批流/工作流/审核通道） | 适配器 + 注册器 | 新类型=加一个 @Component 自动注册，使用方零改动；businessId 语义在类注释声明 |
| 流程完成/导入完成后的联动 | Spring 事件 | 事件方与监听方解耦；注意 @TransactionalEventListener 在无事务上下文会丢事件 |
| 实体→展示对象组装 | 静态工厂 | `Resp.of()`，见上例 |
| 框架默认行为需替换 | 条件装配 | @ConditionalOnMissingBean 定向替换，不动全局配置 |

## 代码风格

- Optional/Stream 链式优先：`Optional.ofNullable().map().orElse()`、`.filter().ifPresent()`；少写 if-else 判空和显式 for
- private 方法同源参数传对象不拆散：多个入参来自同一对象时传对象本身，独立入参才保留为参数
- record/不可变值对象跨类使用 = 独立文件放 dto 包，不藏在 Service 实现类里
- 注释 2~4 行讲「为什么」，不与参数校验注解/API 文档注解重复同一信息
- 关联取数防 N+1（批量查成 Map）、防空 IN（空集合短路）

## Red Flags（出现即回头改）

- for 循环 + `if (xxx != null)` + setter 逐字段灌值
- 配置属性类或「开关+密钥」判断出现在业务 Service
- Adapter 里出现业务判断或直接写 SQL
- private 方法参数 ≥5 个且多数来自同一对象
- ServiceImpl 里定义 private record / 内部 DTO
- 同一段说明在 Javadoc、API 文档注解、@param 三处重复
