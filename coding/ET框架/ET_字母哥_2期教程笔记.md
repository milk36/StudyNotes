# ET_字母哥_2期教程笔记

## 角色数值属性

### 数值组件

1. ET6中的数值组件更新事件通知逻辑: `NumericNoticeComponent`
1. 数值更新协议: `M2C_NoticeUnitNumeric`
1. `UnitFactory.Create` 创建Unit时如果类型为 `Player` 将初始化添加 `PlayerNumericConfig` 配置表中的数值属性

### 数据定时回写组件逻辑

UnitDBSaveComponentSystem

## 挂机战斗逻辑

### 放置闯关系统

* 放置闯关系统(四,五)

  `AdventureComponent` 放置挂机战斗组件

  `TimerType.BattleRound` 战斗定时器类型

  `AdventureBattleRound` 战斗攻击回合事件

  `AdventureBattleRoundView` 战斗攻击回合表现事件

  `AdventureBattleRoundEvent_CalculateDamage` 处理攻击逻辑

  `AdventureBattleRoundView_PlayAnimation` 处理攻击动画播放逻辑

  `AdventureCheckComponentSystem.SimulationBattle` 服务端战斗结果模拟

  `SRandom` 伪随机逻辑(线性同余)

* 红点系统(血条飘血) -- 放置闯关系统(六)
  
  `EventType.ShowAdventureHpBar` 处理血条显示事件

  `FlyDamageValueViewComponent` 飘血组件

  `AfterCreateZoneScene_AddComponent` 处理战斗场景创建添加相关组件

  `RedDotHelper` 红点帮助类

## 背包

### 背包系统(一二三)

* `BagComponent` 背包组件
  
  `Item` 数据的存储逻辑:

  1. `Item` 实现 `ISerializeToEntity` 接口
  1. `Item` 以 `Entity.AddChild()` 的方式添加到 `BagComponent` 组件 `Children` 中
  1. `BagComponent` 下的字段虽然都标记成 `[BsonIgnore]` , 但是 `Item` 回被作为 `Entity.childrenDB` 中的内容被保存到DB中
  1. `BagComponent` 还实现了 `IDeserialize` 接口, 用于对反序列化操作, 其中会把 `Entity.childrenDB` 映射到相关`BagComponent`的业务字段中

      ```c#
      public class BagComponentDeserializeSystem: DeserializeSystem<BagComponent>
      {
          public override void Deserialize(BagComponent self)
          {
              foreach (Entity entity in self.Children.Values)
              {
                  self.AddContainer(entity as Item);
              }
          }
      }
      ```

* `Item` 物品实体
  1. `M2C_ItemUpdateOpInfo` 更新单个物品消息  
     1. `ItemHelper.RandomQuality` 随机生成物品品质
     2. `ItemUpdateNoticeHelper.SyncAddItem` 通知 `Client` 添加物品消息 -> `M2C_ItemUpdateOpInfo`
  2. `M2C_AllItemsList` 同步所以道具信息
     1. `ItemUpdateNoticeHelper.SyncAllBagItems` 同步全部背包物品信息
     2. `ItemUpdateNoticeHelper.SyncAllEquipItems` 同步全部角色装备信息

## Actor消息的转发

ET6中的 `SessionStreamDispatcherServerInner` 逻辑对应到 ET7的 `ActorHandleHelper`

## 设计范式

### Unit不要挂继承ISerializeToEntity的组件

[Unit不要挂孩子，Unit上的组件才能挂孩子](https://et-framework.cn/d/448-20cnyiserializetoentity)
