# 数据字典

本文档是数据分析助手的核心知识库之一，用于定义所有可用数据源的结构和业务含义。
数据可能来自于数据库或者本地文件、文件夹。
---
## 公司和行业术语

### FAS
- **解释**: 博世汽车售后德国总部的数据系统，主要内容是全球的车型、保有量、OE号以及适配数据、产品主数据
- **可能有的其他名字**: KP1

### FAS_KEY
- **解释**: FAS中车型的唯一标识，由2-3位字母和7位数字构成。字母代表车型品牌的代码。数字代表车型的编号，如果编号短于7位会在前面补零。例如`MB0023456`。

### WAVE
- **解释**: 用于创建FAS_KEY的系统，用户可以在WAVE系统里查询FAS_KEY对应的车型信息、保有量、OE号等数据。FAS系统的车型数据是从WAVE系统里同步过来的。

### WAVE_KEY
- **解释**: WAVE系统中车型的唯一标识，由FAS_KEY中的品牌代码+'_'+FAS_KEY中的车型编号去掉开头的0构成。WAVE_KEY和FAS_KEY是一一对应的关系，比如FAS_KEY `MB0023456`对应的WAVE_KEY是`MB_23456`。

### spiderB
- **解释**: 博世汽车售后中国事业部用的适配关系管理系统，主要内容是中国的车型、保有量、OE号以及适配数据

### 料号
- **解释**：博世的10位产品编号，由数字或者数字+大写字母构成
- **可能有的其他名字**: 10位号，PN, SNR,博世号
- **料号的属性描述**:
- `ProductKey`: 6位数字的零部件类型ID，首位有可能是0。相同ProductKey的料号通常是同一类型的零部件。

### TMM
- **解释**: 全称是Technical Material Master，是博世内部一个记录了所有料号的属性（例如ProductKey、OE号、状态）的系统。FAS系统和spiderB系统里料号相关的数据都是从TMM系统里同步过来的。
- **可能有的其他名字**: ViTTA

### 力洋
- **解释**: 博世汽车售后中国的数据供应商，提供了车型的基本信息、OE号等数据

### PC/LCV/HCV
- **解释**: PC代表乘用车, LCV代表轻型商用车, HCV代表重型商用车。这是汽车类型的分类标准，通常在FAS系统的class字段里可以找到这个信息。

---

## 数据源连接

### SMS数据库
- **类型**: MS SQL Server 
- **主机**: `szhvsql42`
- **数据库名**: `DB_MA_SMS_MBL_CNEA_Data_Analysis_SQL`
- **提示**: 使用Trusted_Connection的方式登录，无需用户名


### 新加坡数据库
- **类型**: MS SQL Server 
- **主机**: `sgpvsql51.apac.bosch.com`
- **数据库名**: `DB_APSAnalyticsWorkbench_SQL`
- **提示**: 使用Trusted_Connection的方式登录，无需用户名
---

## 数据表定义
### 表1: `FAS application data`
- **描述**: 记录了FAS里车型在不同国家或地区和博世料号的适配关系以及相关的保有量数据。这个表的数据是从FAS系统里导出的，仅包含部分产品线的数据。包含了所有APAC的国家和地区。
- **数据格式**：sql server表格
- **地址**： 新加坡数据库 `DB_APSAnalyticsWorkbench_SQL` 中的 `kb.SCANex_REPORT_2_SC` 视图
- **字段**:
- `BRAND` (str, 联合主键): FAS中车型品牌代码，由2-3位字母。
- `NR` (str,联合主键): FAS中车型编号，由数字构成。完整的FAS_KEY由BRAND和NR组成，如果NR短于7位需要自行在前面补零以构成完整的FAS_KEY。比如BRAND是MB，NR是23456，那么完整的FAS_KEY应该是MB0023456。
- `LAND` (str,联合主键): 国家或地区标识，`PRC`表示中国，`RC`表示台湾,`J`代表日本，其他国家或地区也有相应的代码。
- `LAND_NAME` (str): 国家或地区名称。
- `PRODUCT`(str,联合主键):产品线名称，例如`Wiper (front)`,`Brake Pad front`,`Cabin-Filter`,`Oil-Filter`,`Battery`,`Air Filter`,`Brake Pad rear`,`Brake Disc front`,`Spark-Plug`,`Brake Disc rear`,`O2 Sensor`,`Ignition Coil`,`Fuel-Filter`,`Wiper (rear)`,`Fuel Pump`,
- `ESCHL` (str,联合主键): 也就是ProductKey,表示零部件类型的ID, 由6位数字构成，首位有可能是0。如果这个车没有关联料号，则为空。
- `MATERIAL` (str,联合主键): 博世料号，如果这个FAS_KEY在这个国家没有关联这个产品线的料号，则为空。如果这个车关联了多个博世料号，则每个料号一行。
- `RG_POPULATION` (str): 使用前需要把数据类型转化成float。该FAS_KEY在该国家或地区的保有量。不同国家和地区的保有量时效性不一样，对于中国而言，RG_POPULATION是截止到前年年底的保有量。
- `RELEVANT_POPULATION` (str): 使用前需要把数据类型转化成float。对于某条产品线，该FAS_KEY在该国家或地区原车配备了该产品线的保有量。如果该FAS_KEY在该国家、该产品线没有连产品或者只连了一个产品，则RELEVANT_POPULATION为RG_POPULATION；如果连了多个产品，则把RG_POPULATION分配到每个产品，使得同一个FAS_KEY、同一个国家、同一个PRODUCT下的RELEVANT_POPULATION的和等于该FAS_KEY在该国家的RG_POPULATION。
- `COVERED_POPULATION` (str): 使用前需要把数据类型转化成float。对于某条产品线，该FAS_KEY在该国家或地区可以适配博世产品的保有量。如果一个这一行关联了博世料号，则COVERED_POPULATION等于RELEVANT_POPULATION；如果一个FAS_KEY没有关联任何博世料号，则COVERED_POPULATION为0。  
### 表2：`spiderB vehicle master data`
- **描述**: 记录了spiderB系统里车型的基本信息，包括车型名称、车型代码、生产年份等。
- **数据格式**：parquet
- **地址**： `\\bosch.com\DfsRb\DfsCN\LOC\Sgh\AA\Department\AA_MBL_CN\05_Data\02_Market Data\14_SpiderB\PRCsnapshot\0_latest\ods_spiderb_sys_main_vehicle_basic.parquet`
- **字段**: 
- `bosch_id`(str,主键): 博世车型ID,spiderB系统中车型的唯一标识。 
- `source_id`(str): 力洋车型ID，source_id和bosch_id一一对应。 
- `manufactor`(str): 生产商。
- `brand`(str): 品牌。
- `brand_en`(str): 品牌英文名称。
- `vehicle_system`(str): 车系名称。
- `vehicle_type`(str): 车型名称。
- `sales_name`(str): 销售名称， 车系->车型->销售名称是层层细化的关系。
- `model_year`(str): 年款。
- `chassis_number`(str): 底盘号。
- `listing_year`(str): 上市年份。
- `listing_month`(str): 上市月份。
- `production_year`(str): 生产年份。
- `shutdown_year`(str): 停产年份。
- `engine_model`(str): 发动机型号。
- `displacement`(str): 排量。
- `intake_form`(str): 进气形式。
- `fuel_type`(str): 燃料类型。
- `transmission_des`(str): 变速器描述。
- `gear_number`(str): 档位数量。
- `driving_form`(str): 驱动形式。
- `vehicle_class`(str): 车辆类别。
- `capital_type`(str): '合资','进口','国产'。
- `is_deleted`(str): 这条记录是否被删除，'1'表示已删除，0表示未删除。只有0的记录才是有效的。
- `create_time`(str): 创建时间。

### 表3： `FAS vehicle master data`
- **描述**: 记录了FAS系统里车型的基本信息，包括车型名称、车型代码、生产年份、发动机信息等。
- **数据格式**：sql query
- **地址**：新加坡数据库 `DB_APSAnalyticsWorkbench_SQL` 中的 `aavm.V_YMTK00006` 视图，需要做加工，sql查询语句如下：
```sql
select KFZMAR as Brand_code, Brand,KFZMAR+KFZLFDNR as fas_key, 
KFZMAR+'_'+SUBSTRING(KFZLFDNR, PATINDEX('%[^0]%', KFZLFDNR + '.'), LEN(KFZLFDNR)) AS wave_key, 
HERSTLAND as production_land, FA as class, OWNER as owner
from aavm.V_YMTK00006 t1 left join 
mbl_sandbox.DIM_FAS_Brands t2
on t1.KFZMAR = t2.Brand_code
```
- **字段**:
- `Brand_code`(str): 品牌代码，FAS系统中品牌的唯一标识，由2-3位字母构成。
- `Brand`(str): 品牌名称。和Brand_code是一一对应的关系。
- `fas_key`(str,主键): FAS中车型的唯一标识，由品牌代码和车型编号构成。
- `wave_key`(str): WAVE系统中车型的唯一标识，由品牌代码和车型编号构成。和fas_key是一一对应的关系。
- `production_land`(str): 生产国别。由代表国家或地区的代码构成，例如PRC表示中国，RC表示台湾，J代表日本。
- `class`(str): 汽车类型的代码，比如P0和PT代表乘用车，N0代表HCV，NT代表LCV。
- `owner`(str): 车型创建者所在的地区取值为'EU', 'APA', 'CN', 'NAF', 'ZA', 'AS', 'IN'。其中"CN"代表中国，"EU"代表欧洲，"APA"代表亚太（不包括中国），"NAF"代表北美，"ZA"代表非洲，"AS"代表拉美，"IN"代表印度。

### 表4：`BoschID to FAS_KEY mapping`
- **描述**: 记录了spiderB系统中的BoschID和FAS系统中的FAS_KEY的对应关系。这个表是通过对比spiderB系统和FAS系统中车型的基本信息（品牌、车系、车型、发动机型号等）生成的。一个FAS_KEY可能对应多个BoschID（例如同一车型的不同年款或者不同市场版本在spiderB系统里可能有不同的BoschID），但一个BoschID只能对应一个FAS_KEY。
- **数据格式**：parquet
- **地址**： `\\bosch.com\DfsRb\DfsCN\LOC\Sgh\AA\Department\AA_MBL_CN\05_Data\02_Market Data\14_SpiderB\PRCsnapshot\0_latest\ods_spiderb_sys_car_mapping.parquet`
- **字段**:
- `bosch_id`(str,外键，联合主键): `spiderB vehicle master data`表中的bosch_id，spiderB系统中车型的唯一标识。
- `fas_key`(str,外键，联合主键): FAS系统中车型的唯一标识。
- `is_deleted`(str): 这条记录是否被删除，'1'表示已删除，'0'表示未删除。只有'0'的记录才是有效的。

### 表5:`FAS VIO data`
- **描述**: 记录了FAS系统中车型在全球每个国家或地区的保有量数据。
- **数据格式**：sql query
- **地址**：新加坡数据库 `DB_APSAnalyticsWorkbench_SQL` 中的 `aavm.V_YMTK00135` 视图，需要做加工，sql查询语句如下：
```sql
select KFZMAR+KFZLFDNR as fas_key, LAND as land, BESTAND as vio
from aavm.V_YMTK00135
```
- **字段**:
- `fas_key`(str,外键，联合主键): FAS系统中车型的唯一标识。
- `land`(str，联合主键): 国家或地区标识，常见的有PRC表示中国，RC表示台湾，J代表日本。
- `vio`(int): FAS_KEY在相应国家或地区的保有量。

### 表6:`spiderB vio data`
- **描述**: 记录了spiderB系统中车型在中国（PRC）。
- **数据格式**：parquet
- **地址**： `\\bosch.com\DfsRb\DfsCN\LOC\Sgh\AA\Department\AA_MBL_CN\05_Data\02_Market Data\14_SpiderB\PRCsnapshot\0_latest\ods_spiderb_sys_main_vehicle_population.parquet`
- **字段**:
- `bosch_id`(str,外键和主键): `spiderB vehicle master data`表中的bosch_id，spiderB系统中车型的唯一标识。
- `population`(str): 该BoschID在中国（PRC）的截止到前年年底的保有量，也就是上传到FAS的保有量。使用前需要把数据类型转化成float。
- `population_2018 `(str): 该BoschID在中国（PRC）截止到上个月的保有量，也就是最新的保有量。使用前需要把数据类型转化成float。
- `is_deleted`(str): 这条记录是否被删除，'1'表示已删除，'0'表示未删除。只有'0'的记录才是有效的。

### 表7:`spiderB application data`
- **描述**: 记录了spiderB系统里车型在中国(PRC)和博世料号的适配关系。这个表的数据是从spiderB系统里导出的，仅包含中国（PRC）的数据。仅包含部分产品线的数据
- **数据格式**：parquet
- **地址**： `\\bosch.com\DfsRb\DfsCN\LOC\Sgh\AA\Department\AA_MBL_CN\05_Data\02_Market Data\14_SpiderB\PRCsnapshot\0_latest\dw_application.parquet`
- **字段**:
- `bosch_id`(str,外键,联合主键): `spiderB vehicle master data`表中的bosch_id，spiderB系统中车型的唯一标识。
- `product_line`(str,外键，联合主键): spiderB中产品线的编号。
- `product_line_name`(str): 产品线名称，和`product_line`一一对应。例如`Wiper`,`Brake pad fr`,`Cabin filter`,`Oil filter`,`Battery`,`Air filter`,`Brake pad rr`,`Brake disk fr`,`Spark plug`,`Lambda sensor`,`Ignition coil`,`Fuel filter`
- `product_key`(str,外键，联合主键): 也就是ProductKey,表示零部件类型的ID, 由6位数字构成，首位有可能是0。如果这个车没有关联料号，则为空。
- `product_number`(str,外键，联合主键): 博世料号，如果这个BoschID在这个产品线没有关联任何料号，则为空。如果这个BoschID关联了多个博世料号，则每个料号一行。
- `oe_number`(str,联合主键): 原厂号码，如果这个BoschID在这个产品线没有关联任何原厂号码，则为空。如果这个BoschID关联了多个原厂号码，则每个号码一行。如果某一行的product_number和oe_number都不为空，表示这个车的这个产品线是根据这个原厂号码匹配的；如果某一行的product_number不为空但oe_number为空，表示这个车的这个产品线是根据其他信息（例如车型、发动机型号等）匹配的；如果某一行的product_number为空但oe_number不为空，表示这个车的没有基于这个oe号关联博世产品。
- `oe_status`(str): OE号状态。'OE new'表示这个车的这个OE号码是新增的；'OE Deleted'表示这个车的这个OE号码被删除了。

### 表8：`spiderB product key info`
- **描述**: 记录了spiderB系统里所有的ProductKey以及对应的产品线信息。这个表的数据是从spiderB系统里导出的。仅包含部分的博世产品线的数据。
- **数据格式**：parquet
- **地址**： `\\bosch.com\DfsRb\DfsCN\LOC\Sgh\AA\Department\AA_MBL_CN\05_Data\02_Market Data\14_SpiderB\PRCsnapshot\0_latest\ods_spiderb_sys_productkey.parquet`
- **字段**:
- `product_key`(str,主键): 也就是ProductKey,表示零部件类型的ID, 由6位数字构成，首位有可能是0。
- `en_name`(str): product_key的英文名称。
- `appdes`(str):product_key的中文名称.
- `product_line`(str): spiderB中产品线的编号
- `is_deleted`(str): 这条记录是否被删除，'1'表示已删除，'0'表示未删除。只有'0'的记录才是有效的。

### 表9：`spiderB product number info`
- **描述**: 记录了spiderB系统里所有的博世料号以及对应的产品信息。这个表的数据是从spiderB系统里导出的。仅包含部分的博世产品线的数据。
- **数据格式**：parquet
- **地址**： `\\bosch.com\DfsRb\DfsCN\LOC\Sgh\AA\Department\AA_MBL_CN\05_Data\02_Market Data\14_SpiderB\PRCsnapshot\0_latest\ods_spiderb_sys_product.parquet`
- **字段**:
- `product_number`(str,主键): 博世料号
- `product_key`(str,外键): 也就是ProductKey,表示零部件类型的ID, 由6位数字构成，首位有可能是0。注意这是产品在TMM系统里的product key。大部分情况下，`FAS application data`以及`spiderB application data`表格里的product key需要和TMM product key一样，但是也有可能不一样。比如33970160LX这个料号在TMM系统里的product key是186504，表示容翼pro雨刮片单支装，但是在application data里这个料号的product key是186501或者186502，表示容翼Pro单支装（主驾）和容翼Pro单支装（副驾）。
- `product_line`(str,外键): spiderB中产品线的编号，和`spiderB product key info`表中的product_line是一一对应的关系。
- `is_deleted`(str): 这条记录是否被删除，'1'表示已删除，'0'表示未删除。只有'0'的记录才是有效的。


### 表10：`spiderB product line info`
- **描述**: 记录了spiderB系统里所有的产品线信息。这个表的数据是从spiderB系统里导出的。仅包含部分的博世产品线的数据。
- **数据格式**：parquet 
- **地址**： `\\bosch.com\DfsRb\DfsCN\LOC\Sgh\AA\Department\AA_MBL_CN\05_Data\02_Market Data\14_SpiderB\PRCsnapshot\0_latest\ods_spiderb_sys_productline.parquet`
- **字段**:
- `product_line`(str,主键): spiderB中产品线的编号。
- `name`(str): 产品线名称，和`spiderB application data`表中的product_line_name是一一对应的关系。例如`Wiper`,`Brake pad fr`,`Cabin filter`,`Oil filter`,`Battery`,`Air filter`,`Brake pad rr`,`Brake disk fr`,`Spark plug`,`Lambda sensor`,`Ignition coil`,`Fuel filter`
- `is_deleted`(str): 这条记录是否被删除，'1'表示已删除，'0'表示未删除。只有'0'的记录才是有效的。



## 常见输出模板
### 模板1:spiderB适配数据模板
- **描述**: 适配数据的标准格式，包含了车型、料号、适配关系等信息。用户在分析过程中可能需要将某些数据转换为这个格式，以便上传到spiderB系统。
- **数据格式**：Excel
- **sheet名**：`application.xlsx`
- **字段**:
- `is_delete` (int): 表示新增记录还是删除记录，'1'表示删除记录，'0'表示新增或者更新记录。
- `BoschID` (str, 联合主键): 博世车型ID。
- `ProductNumber` (str， 联合主键): 10位号。
- `ProductKey` (str, 联合主键): 表示零部件类型的ID, 由6位数字构成，首位有可能是0。
- `OE Number` (str): 车型和料号是根据哪个原厂号码匹配的，默认为''。
- `Remark Internal` (str): 内部备注，默认为''。
- `Remark External` (str): 外部备注，默认为''。
- `Country` (str): 国家，默认为'PRC'。
- `Specialcase` (str): FAS的系统能识别的备注，默认为''。
- `Remark Internal - Local Language` (str): 没有用的字段，默认为''。
- `Remark External - Local Language` (str): 没有用的字段，默认为''。