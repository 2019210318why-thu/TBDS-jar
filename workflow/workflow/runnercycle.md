# 任务调度设置

**调度设置**  
指定任务的运行周期，每一个运行周期生成一个运行实例。

调度设置需要指定如下几个部分：周期类型，起始数据时间，调度时间以及自身依赖关系。

## 1.周期类型

任务的调度策略支持周期类型和非周期类型。  
周期类型支持分钟，小时，天，周，月四种周期类型。 非周期即为一次性非周期任务，任务整个生命周期只有一个实例。

## 2.起始数据时间

指定任务开始生成实例的时间点，以天为单位。  
今天生成昨天的实例，所以默认的起始数据时间是昨天，这样运行任务之后，就会创建一个昨天的实例。  
起始数据时间还受到周期类型的限制。如周期类型为周，那么起始数据时间只能是周一。如果周期类型是月，那么起始数据时间只能是每月一号。

## 3.调度时间

该属性跟周期类型相关（portal页面会根据周期类型，自动调整）。

```text
如果任务周期类型是月周期，调度时间可以选择某天中，某个小时，某分钟。  
如果任务周期类型是week周期，调度时间可以选择一个礼拜中的某天，某小时，某分钟。  
如果任务周期类型是天，调度时间可以选择一天中的某小时，某分钟。  
如果任务周期类型是小时，调度时间可以选择某分钟。
分钟任务不设置调度时间。
```

**需要注意：**  
1. 系统会根据任务设置的调度时间，在一个周期内，生成一个实例。只有当前时间大于当前所在周期的设置的调度时间，实例才会生成。 2. 调度时间设置的时，分属性值，会决定当前周期内的实例是否会被下发，如果当前时间在设置的时，分之前，实例不会被下发。

  比如有一个任务周期类型是月，设置的调度时间是1号，2点30分。  
  系统会为这个任务每个月生成一个实例，生成实例的时间是每个月的1号 2:30  
  如果当前时间是10月，系统会在10月1号 2:30 生成一个数据时间为9月的实例。  
  
 **举例说明：**

  现在我们假设当前时间是2018.03.08 11:06:00，现在我们创建一个小时任务，并指定调度时间为10分。调度设置如下图 ![](../../.gitbook/assets/cycle2.png) 1. 当前会马上生成2018.03.08号，数据时间为0点到10的实例。并且初始化时间为当前时间（相当于补录任务），实例开始运行时间是当前时间。  
2. 如果现在将调度时间修改为30分钟，系统会在12:30:00 左右生成数据时间为11:00:00 的实例。  
3. 只要实例的数据时间+调度时间大于当前时间，实例就会开始运行。  
4. 如果是月任务或是天任务，设置的小时（如上图的 10点），也会限制任务实例的运行时间。但调度时间的分钟，不会有额外限制。

## 4.自身依赖关系

这里的自身依赖关系是指实例之间的依赖关系，选项有：是，否，并行。  
**是** ：当前实例依赖前一个周期的实例的状态（前一个周期实例运行成功，当前实例才会运行）。  
**否**：当前实例和前一个周期实例没有依赖关系，如果一个任务同时存在多个实例，系统随机选取一个实例运行。同时只有一个实例是运行状态。  
**并行**：前一个周期实例和后一个周期实例之间没有依赖关系，如果一个任务同时存在多个实例，多个实例会同时运行。

## 5.终止时间

系统会按照任务的相关设置生成实例。如果当前时间超过终止时间，系统就会不再为任务生成实例。也就是说终止时间决定任务再什么时间点不再生成实例。

**特别注意：** 终止时间不会影响已有实例的运行。也就是说及时当前时间超过设置的终止时间，实例可以运行，也可以重跑。

## 6.是否可重试

决定实例是否可以重试。默认值为是。  
选是，任务生成的实例可以重跑。  
选择否，任务实例只能运行一次，不能重跑。

## 7.重试次数

任务实例运行失败，可能是因为环境组件正在重启等外部临时原因，所以我们添加重试次数，当第一次运行失败，可以由系统将实例重新调起，直到任务运行成功，或者失败次数达到设置的重试次数。  
默认值为重试5次。  
**ps:** 每一次重试对应的优先级都是不一样的。也就是说前一次重试跟下一次重试，被调度的优先级不一样。第一次运行和第二次运行被调度优先级不一样，也就是说系统会优先调度第一次运行的任务。

## 8.步长

步长是周期生成实例的一种扩展。比如可能存在这样一种需求，我们希望实例生成周期是每两天生成一个实例，我们可以将任务设置为天任务，步长设置为2。再比如我们有一个统计功能是按照季度统计，我们可以将任务设置为月任务，步长设置为4。

## 9.代理IP

指定该任务所有实例只能在某些节点运行。使用节点的ip 地址，多个节点ip之间通过逗号\(英文逗号\)分隔

## 10.任务调度优先级

任务调度优先级只对同一个类型的任务有效。不同的任务类型之间该设置是分离的。  
如果任务设置的任务调度优先级一样，则依次判断如下设置来判断优先级先后：  
1. 重试次数（重试次数越大优先级越低,优先没有跑过的实例先跑），  
2. 任务调度类型（分钟任务优先跑，然后是小时任务，天任务，周任务，月任务）， 3. 是否重跑，没有重跑的任务优先跑 4. 是否是补录，不是补录的任务优先跑 5. 数据时间（实例关键字），时间距离当前时间越近优先级越高， 6. 任务创建时间，时间距离当前时间越近优先级越高。

更多说明

### 一、有关非相同周期类型的父子依赖关系说明

子任务在某个周期类型内的所有实例都依赖父任务在相同周期类型内的所有实例。  
比如  
1. 父任务调度类型为周，子任务调度类型为天 假设父任务在某周一的实例是a,子任务在该星期内所有实例为b1-b7。则b1-b7 所有实例都要等a实例成功，才会调度。 2. 父任务调度类型为天，子任务调度类型为周 假设子任务在某周一的实例是a,父任务在该星期内所有实例为b1-b7。则a要等b1-b7 都运行成功，才会调度。

### 二、有关非周期类型依赖的说明

1. 周期性任务依赖一次性非周期任务 子任务所有实例的运行都依赖一次性非周期的父任务实例（只有一个实例） 例如： 某个任务A（周期性任务）依赖父任务B（一次性非周期任务），则任务A的所有实例都要等任务B生成实例并执行成功才会下发运行。
2. 一次性非周期任务依赖周期性任务 子任务（只有一个实例）依赖父任务对应所在天的零时零分零秒的实例。 比如: 某个任务A（一次性非周期任务）依赖父任务B（周期性任务），假设任务A 实例（只有一个实例）数据时间是2018-09-10 00:00:00（都是起始时间对应的当天零时零分零秒）。则A任务的实例要等B任务在2018-09-10 00:00:00的实例运行成功方能下发运行。如果任务B在2018-09-10 00:00:00没有对应实例，则A始终在等待运行。
3. 一次性非周期任务依赖一次性非周期任务 子任务所有实例（只有一个实例）的运行都依赖一次性非周期的父任务实例（只有一个实例） 比如： 某个任务A（一次性非周期性任务）依赖父任务B（一次性非周期任务），则任务A的实例要等任务B生成实例并执行成功才会下发运行。

### 三、有关在某个时间段，只有子任务有实例，父任务没有实例的情况说明

这种情况发生在，子任务的开始时间比父任务的开始时间早。比如同样父子任务都是天任务，父任务开始时间是2018-02-01,子任务的开始时间是2018-01-01 。子任务比父任务早开始一个月，那么子任务会比父任务多一个月的实例。这一个月的实例，因为找不到对应的父实例，就会一直等待调度。  
这个等待调度会浪费系统资源，所以在创建子任务时候需要用户多注意。  
但如果用户有这个需求怎么办？可以复制一个子任务，开始时间选2018-01-01 ，结束时间选2018-02-01 。原来子任务开始时间选在2018-01-01。这样复制出的子任务就会直接执行。

