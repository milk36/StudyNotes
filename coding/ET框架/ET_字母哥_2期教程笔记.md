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

### 关注 AwakeSystem;DestroySystem;DeserializeSystem

* `AwakeSystem` 初始化操作(随机属性/生成新物品等逻辑)
* `DestroySystem` 销毁回收操作(清理数据/回收内存)
* `DeserializeSystem` 反序列化操作(将 Children 数据填充到对应的业务字段中)

### 背包系统(一二三) 同步数据

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
  1. `ItemFactory.Create` Item 实体的创建逻辑;随机物品品质,增加相关组件等
  1. `M2C_ItemUpdateOpInfo` 更新单个物品消息  
     1. `ItemHelper.RandomQuality` 随机生成物品品质
     2. `ItemUpdateNoticeHelper.SyncAddItem` 通知 `Client` 添加物品消息 -> `M2C_ItemUpdateOpInfo`
  1. `M2C_AllItemsList` 同步所以道具信息
     1. `ItemUpdateNoticeHelper.SyncAllBagItems` 同步全部背包物品信息
     1. `ItemUpdateNoticeHelper.SyncAllEquipItems` 同步全部角色装备信息
  1. `Server.ItemSystem.ToMessage` 将 `Item` 实体数据转换成 `Protobuf` 消息体
  1. `Unity.ItemSystem.FromMessage` 将 `Protobuf` 消息体转换成 `Item` 实体数据

* `ItemContainerType` 物品容器类型

  ```c#
  public enum ItemContainerType
  {
      Bag = 0,  //背包容器
      RoleInfo = 1, //游戏角色装配容器
  }
  ```

### 背包系统(四) 卖出/移除

* `C2M_SellItem` 售卖道具协议

  ```pb
  //ResponseType M2C_SellItem
  message C2M_SellItem // IActorLocationRequest
  {
    int32 RpcId = 1;
    int64 ItemUid = 2;
  }
  ```

* `Unity.ItemApplyHelper.SellBagItem` 客户端发起卖出道具操作
* `C2M_SellItemHandler` 服务端移除道具
* `M2C_ItemUpdateOpInfoHandler` 客户端接收移除道具返回数据

### 背包系统(五六) 装备

* `EquipInfoComponent` 装备信息组件; 也实现了 `ISerializeToEntity` 接口, 也会被序列化保存到数据库中
* `AttributeEntry` 属性词条; 也实现了 `ISerializeToEntity` 接口 ;关联方式 `EquipInfoComponent.AddChild<AttributeEntry>()`
  
  `AttributeEntry.Key` 对应 `NumericType`

* `EquipInfoComponentSystem.CreateEntry` 生成装备词条属性
  
  ```c#
  ///创建词条
  public static void CreateEntry(this EquipInfoComponent self)
  {
    xxx
    //创建普通词条
    EntryConfig entryConfig       = EntryConfigCategory.Instance.GetRandomEntryConfigByLevel((int)EntryType.Common,entryRandomConfig.EntryLevel);
    if (entryConfig == null)
    {
        continue;
    }
    //AddChild添加到EquipInfoComponent组件中
    AttributeEntry attributeEntry = self.AddChild<AttributeEntry>();
    //词条类型:普通
    attributeEntry.Type           = EntryType.Common;
    //词条属性类型
    attributeEntry.Key            = entryConfig.AttributeType;
    //词条随机值
    attributeEntry.Value          = RandomHelper.RandomNumber(entryConfig.AttributeMinValue, entryConfig.AttributeMaxValue + self.GetParent<Item>().Quality);
    self.EntryList.Add(attributeEntry);
  }
  ```

* 装备信息的Proto消息转换

  `Server.EquipInfoComponentSystem.ToMessage` Entity -> ProtoMessage

  `Unity.EquipInfoComponentSystem.FromMessage` ProtoMessage -> Entity

### 背包系统(七) 穿戴装备

* `C2M_EquipItem` 请求穿戴装备协议
  
  ```pb
  //ResponseType M2C_EquipItem
  message C2M_EquipItem // IActorLocationRequest
  {
    int32 RpcId = 1;
    int64 ItemUid = 2;
  }
  ```

* `EquipmentsComponentSystem.EquipItem` 穿戴装备道具
* `EquipmentsComponent` 玩家穿戴装备的信息
  
  `EquipPosition` 装备装配部位枚举

  ```c#
  //对应装备位置上的装备道具信息
  public Dictionary<int, Item> EquipItems = new Dictionary<int, Item>();
  ```

### 背包系统(八) 卸下装备

* `C2M_UnloadEquipItem` 请求卸下装备协议
  
  ```pb
  //ResponseType M2C_UnloadEquipItem
  message C2M_UnloadEquipItem // IActorLocationRequest
  {
    int32 RpcId = 1;
    int32 EquipPosition = 2;
  }
  ```

* `EquipmentsComponentSystem.UnloadEquipItemByPosition` 卸下对应位置的装备道具
* `ChangeEquipItem` 穿戴/卸下装备事件

  `ChangeEquipItemEvent_ChangeNumeric` 对应事件监听,处理玩家 穿戴/卸下装备 后属性的变化

## Actor消息的转发

ET6中的 `SessionStreamDispatcherServerInner` 逻辑对应到 ET7的 `ActorHandleHelper`

## 设计范式

### Unit不要挂继承ISerializeToEntity的组件

[Unit不要挂孩子，Unit上的组件才能挂孩子](https://et-framework.cn/d/448-20cnyiserializetoentity)
