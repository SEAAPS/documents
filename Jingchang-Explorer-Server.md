<!-- markdownlint-disable MD024 -->
<!-- markdownlint-disable MD033 -->
<!-- markdownlint-disable MD046 -->
<!-- markdownlint-disable MD029 -->

# 井畅浏览器服务

## 现有井通浏览器查询服务的缺点

1. 不能浏览更多的区块列表，只能看到最新的6个区块列表；

2. 所有查询列表只能每次再向前加载10条记录，没有分页功能，若想看最前面的交易，需要很多次翻页；

3. 根据hash查询时，输入一个区块hash，不能查询；

4. 对于创建挂单的交易Hash查询结果，如果有成交，没有显示实际成交金额，如果部分成交，没有显示实际挂单金额；

5. 挂单中买和卖的关系比较混乱；

6. 对于撮合交易，没有显示实际成交金额；

7. 对于被动成交，如果该被动成交只是一个大单交易中的一部分，那么以该被动成交钱包来查看该交易，不能明显看到哪个被动成交才是自己的；

8. 没有钱包的收益分析；

## 浏览器服务的需求

1. 查询交易列表

2. 查询区块概要信息列表

3. 根据HASH查询交易的详细，HASH分2种：交易Hash和区块Hash

   1. 交易Hash。每个交易均具有一个Hash，根据该Hash，可以查询该交易对应的详情。

   2. 区块Hash。每个区块均具有一个Hash，根据该Hash，可以查询该区块基本信息。

4. 根据区块HASH查询其包含的交易列表

5. 根据钱包地址查询该钱包的余额（所有币种的余额和冻结数量）、委托单以及交易列表

## 服务端接口详细设计

### 1. 查询交易列表

#### 1.1 查询最新的6个交易列表

* route

   `/trans/new/:uuid`

* method

   `get`

* 请求参数

   参数|类型|必填|可选值 |默认值|描述
   --|:--:|:--:|:--:|:--:|:--
   uuid|String|是|-|-|唯一id

* 返回结果

   字段|类型|描述|备注|可能值
   :--|:--:|:--|:--|:--
   code|String|-|"0"表示成功|-
   msg|String|消息描述|-|-
   data|Object|查询结果|-|-
     &emsp;list|Array|交易列表|-|-
     &emsp;&emsp;_id|String|交易哈希|-|-
     &emsp;&emsp;type|String|交易类型|-|[Transaction Type](#Transaction-Type)
     &emsp;&emsp;time|Number|交易时间|-|-
     &emsp;&emsp;past|Number|交易距现在过去的秒数|-|-
     &emsp;&emsp;account|String|交易发起方钱包地址|-|-
     &emsp;&emsp;succ|String|交易结果是否成功|"tesSUCCESS"表示交易成功|-

   1. 当type=`Payment`时，`data.list`包含

         字段|类型|描述|备注|可能值
         :--|:--:|:--|:--|:--
         dest|String|转账对方地址|-|-
         amount|[Amount](#Amount-Object)|转账币种和数量|-|-

   2. 当type=`OfferCreate`时，`data.list`包含

         字段|类型|描述|备注|可能值
         :--|:--:|:--|:--|:--
         takerGets|[Amount](#Amount-Object)|挂单付出币种和数量|-|-
         takerPays|[Amount](#Amount-Object)|挂单得到币种和数量|-|-
         realGets|[Amount](#Amount-Object)|除去立即成交之后实际挂单付出币种和数量|-|-
         realPays|[Amount](#Amount-Object)|除去立即成交之后实际挂单得到币种和数量|-|-
         flag|Number|买或卖|-|[Flag Type](#Flag-Type)

   3. 当type=`OfferCancel`时，`data.list`包含

         字段|类型|描述|备注|可能值
         :--|:--:|:--|:--|:--
         takerGets|[Amount](#Amount-Object)|撤消的该挂单付出币种和数量|可能不存在|-
         takerPays|[Amount](#Amount-Object)|撤消的该挂单得到币种和数量|可能不存在|-
         flag|Number|买或卖|存在的前提是`takerGets`和`takerPays`存在|[Flag Type](#Flag-Type)

#### 1.2 查询所有交易列表

* route

   `/trans/all/:uuid?p=&s=`

* method

   `get`

* 请求参数

   参数|类型|必填|可选值 |默认值|描述
   --|:--:|:--:|:--:|:--:|:--
   uuid|String|是|-|-|唯一id
   p|Number|是|-|-|页数，从0开始
   s|Number|是|10/20/50/100|20|每页条数

* 返回结果

  * 同[1.1](#11-查询最新的6个交易列表)

  * 最多只能查询100万条交易记录（按照每个区块100条交易计算，大概一天的交易数据）

### 2. 查询区块信息列表

#### 2.1 查询最新的6个区块

* route

   `/block/new/:uuid`

* method

   `get`

* 请求参数

   参数|类型|必填|可选值 |默认值|描述
   --|:--:|:--:|:--:|:--:|:--
   uuid|String|是|-|-|唯一id

* 返回结果

   字段|类型|描述|备注|可能值
   :--|:--:|:--|:--|:--
   code|String|-|"0"表示成功|-
   msg|String|消息描述|-|-
   data|Object|查询结果|-|-
     &emsp;list|Array|交易列表|-|-
     &emsp;&emsp;_id|Number|区块高度|-|-
     &emsp;&emsp;time|Number|区块关闭时间|-|-
     &emsp;&emsp;past|Number|该区块距现在过去的秒数|-|-
     &emsp;&emsp;transNum|Number|区块内交易个数|-|-
     &emsp;&emsp;hash|String|区块哈希|-|-
     &emsp;&emsp;parentHash|String|上一区块的哈希|-|-

#### 2.2 查询所有区块信息列表

* route

   `/block/all/:uuid?p=&s=`

* method

   `get`

* 请求参数

   参数|类型|必填|可选值 |默认值|描述
   --|:--:|:--:|:--:|:--:|:--
   uuid|String|是|-|-|唯一id
   p|Number|是|-|-|页数，从0开始
   s|Number|是|10/20/50/100|20|每页条数

* 返回结果

  * 同[2.1](#21-查询最新的6个区块)

  * 最多只能查询300万条区块记录（大概接近一年的区块数据）

### 3. 根据哈希查询交易详细

* route

   `/hash/detail/:uuid?h=`

* method

   `get`

* 请求参数

   参数|类型|必填|可选值 |默认值|描述
   --|:--:|:--:|:--:|:--:|:--
   uuid|String|是|-|-|唯一id
   h|String|是|-|-|hash

* 返回结果

   1. 如果是交易哈希

      字段|类型|描述|备注|可能值
      :--|:--:|:--|:--|:--
      code|String|-|"0"表示成功|-
      msg|String|消息描述|-|-
      data|Object|查询结果|-|-
      &emsp;_id|String|交易哈希|-|-
      &emsp;hashType|Number|哈希类型|1:区块哈希<br>2:交易哈希|2
      &emsp;upperHash|String|所属区块哈希|-|-
      &emsp;block|Number|所属区块的区块高度|-|-
      &emsp;time|Number|交易发生的时间|-|-
      &emsp;past|Number|交易距现在过去的秒数|-|-
      &emsp;type|String|交易类型|-|[Transaction Type](#Transaction-Type)
      &emsp;account|String|交易发起方钱包地址|-|-
      &emsp;seq|Number|在交易发起方钱包所有交易中的序号|-|-
      &emsp;fee|Number|交易gas费用|单位是SEAAPS，小数，最小0.00001|-
      &emsp;succ|String|交易是否成功|"tesSUCCESS"表示交易成功|-

      1. 当type=`Payment`时，`data`包含

         字段|类型|描述|备注|可能值
         :--|:--:|:--|:--|:--
         dest|String|转账目的钱包地址|-|-
         amount|[Amount](#Amount-Object)|转账币种和数量|-|-
         memos|Array|转账备注|-|-
         &emsp;Memo|[Memo](#Memo-Object)|备注内容|-|-

      2. 当type=`OfferCreate`时，`data`包含

         字段|类型|描述|备注|可能值
         :--|:--:|:--|:--|:--
         flag|Number|买/卖|-|[Flag Type](#Flag-Type)
         takerGets|[Amount](#Amount-Object)|挂单付出币种和数量|-|-
         takerPays|[Amount](#Amount-Object)|挂单得到币种和数量|-|-
         realGets|[Amount](#Amount-Object)|实际挂单付出币种和数量（即扣除立即成交之后形成Offer的那部分）|若挂单立即全部成交则该字段不存在|-
         realPays|[Amount](#Amount-Object)|实际挂单得到币种和数量（即扣除立即成交之后形成Offer的那部分）|若挂单立即全部成交则该字段不存在|-
         matchGets|[Amount](#Amount-Object)|实际成交付出币种和数量|若没有实际成交则该字段不存在|-
         matchPays|[Amount](#Amount-Object)|实际成交得到币种和数量|若没有实际成交则该字段不存在|-
         matchFlag|Number|撮合标志|若没有撮合，则该字段不存在；数字: 表示多方撮合，比如3表示三方撮合|-
         affectedNodes|Array|挂单立即成交部分（以被动成交钱包的角度）|根据该数组分析出matchGets、matchPays以及matchFlag|-
         &emsp;&emsp;account|String|被动成交的钱包地址|-|-
         &emsp;&emsp;seq|Number|该被动成交的挂单序号|-|-
         &emsp;&emsp;flag|Number|该被动成交的挂单性质|-|[Flag Type](#Flag-Type)
         &emsp;&emsp;previous|Object|被动成交前的交易对币种和数量|该字段可能没有，若没有该字段，表示这个被动成交记录是撤消自己的反向挂单，这种情况在自己新的挂单会吃掉自己以前的反向挂单时会发生，就是说不允许自己吃掉自己的挂单，一旦要出现这种情况时，会先把自己以前的反向挂单撤消，然后再把新单挂上去|-
         &emsp;&emsp;final|Object|被动成交后的数量|-|-

      3. 当type=`OfferCancel`时，`data`包含

         字段|类型|描述|备注|可能值
         :--|:--:|:--|:--|:--
         offerSeq|Number|取消挂单的序号|-|-
         takerGets|[Amount](#Amount-Object)|取消挂单的付出币种和数量|-|-
         takerPays|[Amount](#Amount-Object)|取消挂单的得到币种和数量|-|-

   2. 如果是区块哈希

      字段|类型|描述|备注|可能值
      :--|:--:|:--|:--|:--
      code|String|-|"0"表示成功|-
      msg|String|消息描述|-|-
      data|Object|查询结果|-|-
      &emsp;list|Array|区块中包含的交易列表|字段说明参见交易哈希查询返回结果|-
      &emsp;count|Number|区块中包含的交易总数|-|-
      &emsp;info|Object|区块信息|-|-
      &emsp;&emsp;_id|String|区块哈希|-|-
      &emsp;&emsp;hashType|Number|哈希类型|1:区块哈希；2:交易哈希|1
      &emsp;&emsp;upperHash|String|所属区块哈希|-|""
      &emsp;&emsp;block|Number|区块的区块高度|-|-
      &emsp;&emsp;time|Number|区块的关闭时间|-|-
      &emsp;&emsp;past|Number|区块距现在过去的秒数|-|-
      &emsp;&emsp;transNum|Number|区块中交易数量|-|-
      &emsp;&emsp;parentHash|String|上一区块的哈希|-|-
      &emsp;&emsp;totalCoins|String|SEAAPS的总量|客户端除以1000000后所得即是真实的SEAAPS总量（需要bignumber处理）|-

### 4. 根据区块HASH查询其包含的交易列表

* route

   `/hash/trans/:uuid?p=&s=&n=&h=`

* method

   `get`

* 请求参数

   参数|类型|必填|可选值 |默认值|描述
   --|:--:|:--:|:--:|:--:|:--
   uuid|String|是|-|-|唯一id
   h|String|是|-|-|交易hash
   p|Number|是|-|-|页数，从0开始
   s|Number|是|10/20/50/100|20|每页条数
   n|Number|是|-|-|区块中一共有多少个交易记录

* 返回结果

   字段|类型|描述|备注|可能值
   :--|:--:|:--|:--|:--
   code|String|-|"0"表示成功|-
   msg|String|消息描述|-|-
   data|Object|查询结果|-|-
   &emsp;list|Array|区块中包含的交易列表|绝大部分字段说明参见交易哈希查询返回结果|-
   &emsp;&emsp;index|Number|交易在区块内的序号|index升序排列|-

### 5. 指定钱包查询

#### 5.1 指定钱包的余额查询（包括所有Token的余额、所有Token的冻结数量）

* route

   `/wallet/balance/:uuid?w=`

* method

   `get`

* 请求参数

   参数|类型|必填|可选值 |默认值|描述
   --|:--:|:--:|:--:|:--:|:--
   uuid|String|是|-|-|唯一id
   w|String|是|-|-|钱包地址

* 返回结果

   字段|类型|描述|备注|可能值
   :--|:--:|:--|:--|:--
   code|String|-|"0"表示成功|-
   msg|String|消息描述|-|-
   data|Object|查询结果|-|-
   &emsp;_id|String|钱包地址|-|-
   &emsp;(`token名称`_`发行方`)|Object|-|-|-
   &emsp;&emsp;value|String|余额|-|-
   &emsp;&emsp;frozen|String|冻结数量|-|-

### 5.2 指定钱包的当前委托单查询

* route

   `/wallet/offer/:uuid?p=&s=&c=&bs=&w=`

* method

   `get`

* 请求参数

   参数|类型|必填|可选值 |默认值|描述
   --|:--:|:--:|:--:|:--:|:--
   uuid|String|是|-|-|唯一id
   p|Number|是|-|-|页数，从0开始
   s|Number|否|10/20/50/100|20|每页条数
   w|String|是|-|-|钱包地址
   c|String|否|-|-|交易对（可以不传值，不传值表示查询全部类型交易对的委托单。形如：SEAAPS-CNY或SEAAPS-cny，必须是交易对本来的顺序base-counter的形式，比如这里不能是CNY-SEAAPS，否则买卖关系可能就乱了，另外交易对可以只指定base或counter，如SEAAPS-或-cny，所有的交易对请参见附录）
   bs|Number|否|0:买或卖<br>1:买<br>2:卖|0|委托性质,如果传值必须与c参数一起使用

* 返回结果

   字段|类型|描述|备注|可能值
   :--|:--:|:--|:--|:--
   code|String|-|"0"表示成功|-
   msg|String|消息描述|-|-
   data|Object|查询结果|-|-
   &emsp;count|Number|委托单个数|-|-
   &emsp;list|Array|区块中包含的交易列表|-|-
   &emsp;&emsp;time|Number|委托单的挂单时间（所属区块的时间）|-|-
   &emsp;&emsp;past|Number|距现在过去的秒数|-|-
   &emsp;&emsp;hash|String|挂单哈希|-|-
   &emsp;&emsp;block|Number|区块高度|-|-
   &emsp;&emsp;flag|Number|买/卖|-|[Flag Type](#Flag-Type)
   &emsp;&emsp;takerGets|[Amount](#Amount-Object)|挂单付出币种和数量|-|-
   &emsp;&emsp;takerPays|[Amount](#Amount-Object)|挂单得到币种和数量|-|-

#### 5.3 指定钱包的历史交易查询

* route

   `/wallet/trans/:uuid?p=&s=&b=&e=&t=&bs=&c=&w=`

* method

   `get`

* 请求参数

   参数|类型|必填|可选值 |默认值|描述
   --|:--:|:--:|:--:|:--:|:--
   uuid|String|是|-|-|唯一id
   p|Number|是|-|-|页数，从0开始
   s|Number|是|10/20/50/100|20|每页条数
   b|String|否|-|-|开始日期，形如YYYY-MM-DD
   e|String|否|-|-|结束日期，形如YYYY-MM-DD
   t|String|否|OfferCreate/OfferAffect/OfferCancel/Send/Receive|-|交易类型（多个类型以逗号分隔，可以不传值，不传值表示查询所有类型）
   c|String|否|-|-|交易对或币种（可以不传值，不传值表示币种不作为查询条件。在t=OfferCreate或OfferAffect或OfferCancel时，传值必须形如：SEAAPS-CNY或SEAAPS-cny，必须是交易对本来的顺序base-counter的形式，比如这里不能是CNY-SEAAPS，否则买卖关系可能就乱了，另外交易对可以只指定base或counter，如SEAAPS-或-cny，所有的交易对请参见附录；在t=Send或Receive时，传值必须长度<8，如JJCC）
   bs|Number|否|0:买或卖<br>1:买<br>2:卖|-|只有在t=OfferCreate或OfferAffect或OfferCancel时有效果
   w|String|是|-|-|钱包地址

* 返回结果

   字段|类型|描述|备注|可能值
   :--|:--:|:--|:--|:--
   code|String|-|"0"表示成功|-
   msg|String|消息描述|-|-
   data|Object|查询结果|-|-
   &emsp;list|Array|区块中包含的交易列表|-|-
   &emsp;type|String|交易类型|-|[Transaction Type](#Transaction-Type)
   &emsp;time|Number|交易发生的时间|-|-
   &emsp;past|Number|交易距现在过去的秒数|-|-
   &emsp;hash|String|交易哈希|-|-
   &emsp;block|Number|区块高度|-|-
   &emsp;fee|String|交易gas费用|OfferAffect和Receive时，fee=""|-
   &emsp;success|String|交易是否成功|"tesSUCCESS"表示成功|-
   &emsp;seq|Number|交易序号|对于OfferAffect和Receive，该字符无意义|-

   1. 当type=`Send`或`Receive`时，`data.list`包含

         字段|类型|描述|备注|可能值
         :--|:--:|:--|:--|:--
         account|String|对方钱包地址|-|-
         amount|[Amount](#Amount-Object)|支付或收到的币种和数量|-|-

   2. 当type=`OfferCreate`时，`data.list`包含

         字段|类型|描述|备注|可能值
         :--|:--:|:--|:--|:--
         flag|Number|买/卖|-|[Flag Type](#Flag-Type)
         matchFlag|Number|撮合标志|若没有撮合，则该字段不存在；数字: 表示多方撮合，比如3表示三方撮合|-
         takerGets|[Amount](#Amount-Object)|创建挂单时付出的币种和数量|-|-
         takerPays|[Amount](#Amount-Object)|创建挂单时得到的币种和数量|-|-
         takerGetsFact|[Amount](#Amount-Object)|立即成交剩余的实际挂单部分的付出币种和数量|如果挂单全部成交，则没有该字段|-
         takerPaysFact|[Amount](#Amount-Object)|立即成交剩余的实际挂单部分的得到币种和数量|如果挂单全部成交，则没有该字段|-
         takerGetsMatch|[Amount](#Amount-Object)|立即成交部分的付出币种和数量|如果没有立即成交，则没有该字段|-
         takerPaysMatch|[Amount](#Amount-Object)|立即成交部分的得到币种和数量|如果没有立即成交，则没有该字段|-

   3. 当type=`OfferAffect`时，`data.list`包含

        字段|类型|描述|备注|可能值
        :--|:--:|:--|:--|:--
        flag|Number|买/卖|-|[Flag Type](#Flag-Type)
        matchFlag|Number|撮合标志|若没有撮合，则该字段不存在；数字: 表示多方撮合，比如3表示三方撮合|-
        takerGets|[Amount](#Amount-Object)|被动成交前挂单的付出币种和数量|-|-
        takerPays|[Amount](#Amount-Object)|被动成交前挂单的得到币种和数量|-|-
        takerGetsFact|[Amount](#Amount-Object)|被动成交剩余部分的付出币种和数量|如果全部被动成交，则没有该字段|-
        takerPaysFact|[Amount](#Amount-Object)|被动成交剩余部分的得到币种和数量|如果全部被动成交，则没有该字段|-
        takerGetsMatch|[Amount](#Amount-Object)|被动成交部分的付出币种和数量|-|-
        takerPaysMatch|[Amount](#Amount-Object)|被动成交部分的得到币种和数量|-|-

   4. 当type=`OfferCancel`时，`data.list`包含

        字段|类型|描述|备注|可能值
        :--|:--:|:--|:--|:--
        offerSeq|Number|被撤消挂单的序号|-|-
        flag|Number|被撤消挂单的交易性质|-|[Flag Type](#Flag-Type)
        takerGets|[Amount](#Amount-Object)|被撤消挂单的付出币种和数量|账本中经常出现一个挂单被多次撤消的情况，所以该字段可能没有|-
        takerPays|[Amount](#Amount-Object)|被撤消挂单的得到币种和数量|账本中经常出现一个挂单被多次撤消的情况，所以该字段可能没有|-

#### 5.4 指定钱包收益分析

* route

   `/wallet/profit/:uuid?t=&v=&w=`

* method

   `get`

* 请求参数

   参数|类型|必填|可选值 |默认值|描述
   --|:--:|:--:|:--:|:--:|:--
   uuid|String|是|-|-|唯一id
   t|Number|是|1:日<br>2:周<br>3:月<br>4:年|-|类型
   w|String|是|-|-|钱包地址

### 6. 持仓排行

* 设计说明：后台设计一个数据库 skywell_sum目前存放两个表
   1. tokens: 存放所有tokens列表
   2. tokensSort: 存放所有tokens的前100数据，不够100显示所有，每天生成 一次数据，目前160多种tokens，每天增长7000多条，数据量不大，放在一个表

#### 6.1 获取所有tokens列表

* route

   `/sum/all/:uuid?t=`

* method

   `get`

* 请求参数

   参数|类型|必填|可选值 |默认值|描述
   --|:--:|:--:|:--:|:--:|:--
   uuid|String|是|-|-|唯一id
   t|String|否|-|-|token名称

* 返回结果

   字段|类型|描述|备注|可能值
   :--|:--:|:--|:--|:--
   code|String|-|"0"表示成功|-
   msg|String|消息描述|-|-
   data|Object|查询结果|-|-
   &emsp;(key为token首字母或`num`)|Array|-|-|-
   &emsp;&emsp;-|String|token名称和发行方地址，下划线连接|-|-

### 7. 获取某种token的100排名

* route

   `/sum/list/:uuid?t=&p=&s=&b=&e=`

* method

   `get`

* 请求参数

   参数|类型|必填|可选值 |默认值|描述
   --|:--:|:--:|:--:|:--:|:--
   uuid|String|是|-|-|唯一id
   t|String|否|-|-|token名称（SWT无需发行方，值为SEAAPS_，其他token需要带发行方，币种大写, 如c=CNY_jGa9J9TkqtBcUoHe2zqhVFFbgUVED6o9or）
   p|Number|是|-|-|页数，从0开始
   s|Number|否|10/20/50/100|20|每页条数
   b|String|否|-|-|开始日期，形如YYYY-MM-DD
   e|String|否|-|-|结束日期，形如YYYY-MM-DD

* 返回结果

   字段|类型|描述|备注|可能值
   :--|:--:|:--|:--|:--
   code|String|-|"0"表示成功|-
   msg|String|消息描述|-|-
   data|Array|查询结果|-|-
   &emsp;tokens|String|token名称和发行方地址，下划线连接|-|-
   &emsp;totalsupply|Number|总发行量|-|-
   &emsp;holders|Number|对应钱包持有数量|-|-
   &emsp;circulation|Number|流通量|-|-
   &emsp;issueDate|Number|token发行日期|-|-
   &emsp;flag |Number|是否跨链标志|-|0没有跨链，1表示跨链token

### 8. 查看自己排名

* route

   `/sum/self/:uuid?t=&w=`

* method

   `get`

* 请求参数

   参数|类型|必填|可选值 |默认值|描述
   --|:--:|:--:|:--:|:--:|:--
   uuid|String|是|-|-|唯一id
   t|String|是|-|-|token名称（SWT无需发行方，值为SEAAPS_，其他token需要带发行方，币种大写, 如c=CNY_jGa9J9TkqtBcUoHe2zqhVFFbgUVED6o9or）
   w|String|是|-|-|钱包地址

* 返回结果

   字段|类型|描述|备注|可能值
   :--|:--:|:--|:--|:--
   code|String|-|"0"表示成功|-
   msg|String|消息描述|-|-
   data|Number|个人排名| 如果没有持有，则data返回空|-

### 9. 查看银关地址发行过tokens

* route

   `/wallet/fingate_tokenlist/:uuid?w=`

* method

   `get`

* 请求参数

   参数|类型|必填|可选值 |默认值|描述
   --|:--:|:--:|:--:|:--:|:--
   uuid|String|是|-|-|唯一id
   w|String|是|-|-|银关地址

* 返回结果

   字段|类型|描述|备注|可能值
   :--|:--:|:--|:--|:--
   code|String|-|"0"表示成功|-
   msg|String|消息描述|-|-
   data|Object|查询结果|-|-
   &emsp;(key为银关地址)|Array|-|-|-
   &emsp;&emsp;currency|String|token名称|-|-
   &emsp;&emsp;issuer|String|发行方地址|-|-

### 10. 查询某个token在某个时间段内的转账交易hash信息

* route

   `/hash/payment_trans/:uuid?p=&s=&b=&e=&t=&c=`

* method

   `get`

* 请求参数

   参数|类型|必填|可选值 |默认值|描述
   --|:--:|:--:|:--:|:--:|:--
   uuid|String|是|-|-|唯一id
   p|Number|是|-|-|页数，从0开始
   s|Number|否|10/20/50/100|20|每页条数
   b|String|否|-|-|开始日期，形如YYYY-MM-DD
   e|String|否|-|-|结束日期，形如YYYY-MM-DD
   t|String|是|Payment|-|交易类型
   c|String|是|-|-|token名称

* 返回结果

   字段|类型|描述|备注|可能值
   :--|:--:|:--|:--|:--
   code|String|-|"0"表示成功|-
   msg|String|消息描述|-|-
   data|Object|查询结果|-|-
   &emsp;count|Number|查询结果的个数|-|-
   &emsp;list|Array|-|-|-
   &emsp;&emsp;_id|String|交易哈希|-|-
   &emsp;&emsp;hashType|Number|哈希类型|1:区块哈希<br>2:交易哈希|2
   &emsp;&emsp;time|Number|时间戳|与当前时间换算关系new Date((time+946684800)*1000)|-
   &emsp;&emsp;index|Number|区块内的交易编号|-|-
   &emsp;&emsp;type|String|交易类型|-|[Transaction Type](#Transaction-Type)
   &emsp;&emsp;account|String|交易发起方钱包地址|-|-
   &emsp;&emsp;seq|Number|交易序列号|-|-
   &emsp;&emsp;fee|Number|交易gas费用|单位是SEAAPS，小数，最小0.00001|-
   &emsp;&emsp;succ|String|交易是否成功|"tesSUCCESS"表示交易成功|-
   &emsp;&emsp;dest|String|对方钱包地址|-|-
   &emsp;&emsp;amount|[Amount](#Amount-Object)|支付币种和数量|-|-

### 11. 查询某个钱包每天/月/年的支付或收到某个token的笔数和数量

* route

   `/sum/payment_summary/:uuid?w=&b=&dt=&t=&c=`

* method

   `get`

* 请求参数

   参数|类型|必填|可选值 |默认值|描述
   --|:--:|:--:|:--:|:--:|:--
   uuid|String|是|-|-|唯一id
   w|String|是|-|-|钱包地址
   dt|Number|是|2: 日<br>3:月<br>4:年|-|日期类型
   b|String|是|-|-|具体某一天/月/年的日期，若dt=2，则必须形如YYYY-MM-DD；若dt=3，则必须形如YYYY-MM；若dt=4，则必须形如YYYY
   e|String|否|-|-|格式要求同b参数。若指定e的值，则累加统计b～e时间段范围内相同token的值，另外e的值必须大于b的值。对于dt=2或3时，b和e之间天数之差不能超过一年的时间；dt=4时，b和e之间的间隔没有限制
   t|String|否|Send/Receive|-|交易类型
   c|String|否|-|-|token名称（SWT无需发行方，值为SEAAPS_，其他token需要带发行方，币种大写, 如c=CNY_jGa9J9TkqtBcUoHe2zqhVFFbgUVED6o9or）

* 返回结果

   字段|类型|描述|备注|可能值
   :--|:--:|:--|:--|:--
   code|String|-|"0"表示成功|-
   msg|String|消息描述|-|-
   data|Array|查询结果|-|-
   &emsp;wallet|String|钱包地址|-|-
   &emsp;token|String|token名称和发行方地址，下划线连接|-|-
   &emsp;type|String|支付/收到|-|Send:支付；Receive:收到
   &emsp;num|Number|总笔数|-|-
   &emsp;amount|Number|总数量|-|-

   1. 如果某个钱包有转出或收到某个token，这时若没有指定type类型，则会返回2条记录信息如果某个钱包有转出多个token，若没有指定token名称，则会返回多条记录。

### 12. 查询收费平台对应token的收费详情接口

* route

   `/wallet/fee_config/:uuid?p=&w=&k=&s=&t=`

* method

   `get`

* 请求参数

   参数|类型|必填|可选值 |默认值|描述
   --|:--:|:--:|:--:|:--:|:--
   uuid|String|是|-|-|唯一id
   p|Number|是|-|-|页数，从0开始
   s|Number|否|10/20/50/100|20|每页条数
   w|String|是|-|-|平台地址
   k|Number|否|1:gas费设置<br>2:手续费设置|2|收费类型
   t|String|否|-|-|token名称（SWT无需发行方，值为SEAAPS，其他token需要带发行方，币种大写, 如t=CNY_jGa9J9TkqtBcUoHe2zqhVFFbgUVED6o9or）

* 返回结果

   字段|类型|描述|备注|可能值
   :--|:--:|:--|:--|:--
   code|String|-|"0"表示成功|-
   msg|String|消息描述|-|-
   data|Object|查询结果|-|-
   &emsp;data[0]|Array|各币种收费列表|-|-
   &emsp;&emsp;platform|String|平台方钱包地址|-|-
   &emsp;&emsp;token|String|token名称和发行方地址，下划线连接|-|-
   &emsp;&emsp;den|Number|费率分母|-|-
   &emsp;&emsp;feeAccount|String|收费钱包地址|-|-
   &emsp;&emsp;num|Number|费率分子|-|-
   &emsp;&emsp;time|Number|设置时间|-|-
   &emsp;data[1]|Number|符合条件数量|用于分组|-

## 附录

### 交易对说明

```json
{
    "BASE_COUNTER": {
        "CNY": [
            { "base": "SEAAPS", "counter": "CNY" },
            { "base": "JJCC", "counter": "CNY" },
            { "base": "JMOAC", "counter": "CNY" },
            { "base": "JETH", "counter": "CNY" },
            { "base": "JSTM", "counter": "CNY" },
            { "base": "VCC", "counter": "CNY" },
            { "base": "JCALL", "counter": "CNY" },
            { "base": "JSLASH", "counter": "CNY" },
            { "base": "CSP", "counter": "CNY" }
        ],
        "JETH": [
            { "base": "SEAAPS", "counter": "JETH" },
            { "base": "JMOAC", "counter": "JETH" }
        ],
        "SEAAPS": [
            { "base": "JJCC", "counter": "SEAAPS" },
            { "base": "JMOAC", "counter": "SEAAPS" },
            { "base": "JBIZ", "counter": "SEAAPS" },
            { "base": "JSLASH", "counter": "SEAAPS" },
            { "base": "JDABT", "counter": "SEAAPS" },
            { "base": "HJT", "counter": "SEAAPS" },
            { "base": "JSTM", "counter": "SEAAPS" },
            { "base": "JSNRC", "counter": "SEAAPS" },
            { "base": "MYT", "counter": "SEAAPS" },
            { "base": "JCALL", "counter": "SEAAPS" },
            { "base": "JEKT", "counter": "SEAAPS" },
            { "base": "BIC", "counter": "SEAAPS" },
            { "base": "YUT", "counter": "SEAAPS" },
            { "base": "VCC", "counter": "SEAAPS" },
            { "base": "JCKM", "counter": "SEAAPS" }
        ]
    }
}
```

### Amount Object

   字段|类型|描述
   :--:|:--:|:--
   currency|String|币种名称
   issuer|String|发行方地址
   value|String|数量

### Transaction Type

类型|描述|可能值
|:--:|:--|:--
String|交易类型|Payment/OfferCreate/OfferCancel/TrustSet/RelationSet<br>RelationDel/SetBlackList/RemoveBlackList/ManageIssuer/SetRegularKey

### Memo Object

字段|类型|描述|备注
:--:|:--:|:--|:--
MemoData|String|备注内容|16进制，[How to parse](https://github.com/SEAAPSca/SEAAPSlib/blob/master/packages/SEAAPS-serializer/src/types/STMemo.ts#L80)
MemoType|String|备注类型|16进制，[How to parse](https://github.com/SEAAPSca/SEAAPSlib/blob/master/packages/SEAAPS-serializer/src/types/STMemo.ts#L80)

### Flag Type

类型|描述|可能值
|:--:|:--|:--
Number|买或卖|1:买<br>2:卖<br>0:未知
