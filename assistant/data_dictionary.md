# 数据字典

本文档是数据分析助手的核心知识库之一，用于定义所有可用数据源的结构和业务含义。
数据可能来自于数据库或者本地文件、文件夹。
---
## 公司和行业术语

### FAS
- **解释**: 博世汽车售后德国总部的数据系统，主要内容是全球的车型、保有量、OE号以及适配数据
- **可能有的其他名字**: KP1

### FAS_KEY
- **解释**: FAS中车型的唯一标识，由2-3位字母和7位数字构成。字母代表车型品牌的代码。数字代表车型的编号，如果编号短于7位会在前面补零。例如`MB0023456`。

### WAVE
- **解释**: 用于创建FAS_KEY的系统，用户可以在WAVE系统里查询FAS_KEY对应的车型信息、保有量、OE号等数据。FAS系统的车型数据是从WAVE系统里同步过来的。

### WAVE_KEY
- **解释**: WAVE系统中车型的唯一标识，由FAS_KEY中的品牌代码+'_'+FAS_KEY中的车型编号去掉开头的0构成。WAVE_KEY和FAS_KEY是一一对应的关系，比如FAS_KEY `MB0023456`对应的WAVE_KEY是`MB_23456`。

### spiderB
- **解释**: 博世汽车售后中国事业部用的适配关系管理系统，主要内容是全球的车型、保有量、OE号以及适配数据

### 料号
- **解释**：博世的10位产品编号，由数字或者数字+大写字母构成
- **可能有的其他名字**: 10位号，PN, SNR,博世号

### 力洋
- **解释**: 博世汽车售后中国的数据供应商，提供了车型的基本信息、OE号等数据

### PC/LCV/HCV
- **解释**: PC代表乘用车, LCV代表轻型商用车, HCV代表重型商用车。这是汽车类型的分类标准，通常在FAS系统的class字段里可以找到这个信息。

---
## 数据库以及链接方式

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
### 表1: `CN FAS coverage tracking file`
- **描述**: 记录了FAS里中国和台湾的车型连了哪些料号
- **数据格式**：Excel
- **地址**： `\\bosch.com\Dfsrb\DfsSG\DIV\AA\98_Shared\05_MBL-APS\Coverage Assistant & Tracking\CN\February\Tracking File`。该目录底下有多个文件，每个文件对应一条产品线，可以通过文件名区分。例如 `Battery_Tracking_File.xlsx`对应了电池产品线的覆盖数据。如果不确定要使用的文件具体是什么名字，可以通过os.listdir()来列出目录下的所有文件，找到对应的文件。
- **字段**:
    - `FAS_KEY` (str, 联合主键): FAS中车型的唯一标识。
    - `LAND` (str,联合主键): 国家或地区标识，PRC表示中国，RC表示台湾。
    - `MATERIAL` (str,联合主键): 博世料号，如果这个车没有关联料号，则为空。如果这个车关联了多个博世料号，则每个料号一行。
    - `RELEVANT_POPULATION` (DECIMAL): FAS_KEY在相应地区的保有量。
    - `Product Key` (str): 表示零部件类型的ID, 由6位数字构成，首位有可能是0。如果这个车没有关联料号，则为空。

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
- `bosch_id`(str,外键): `spiderB vehicle master data`表中的bosch_id，spiderB系统中车型的唯一标识。
- `fas_key`(str,外键): FAS系统中车型的唯一标识。
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


## 常见输出模板
### 模板1:spiderB适配数据模板
- **描述**: 适配数据的标准格式，包含了车型、料号、适配关系等信息。用户在分析过程中可能需要将某些数据转换为这个格式，以便上传到spiderB系统。
- **数据格式**：Excel
- **sheet名**：`application.xlsx`
- **字段**:
- `is_delete` (int): 表示新增记录还是删除记录，1表示删除记录，0表示新增或者更新记录。
- `BoschID` (str, 联合主键): 博世车型ID。
- `ProductNumber` (str， 联合主键): 10位号。
- `ProductKey` (str, 联合主键): 表示零部件类型的ID, 由6位数字构成，首位有可能是0。
- `OE Number` (str): 车型和料号是根据哪个原厂号码匹配的，默认为空。
- `Remark Internal` (str): 内部备注，默认为空。
- `Remark External` (str): 外部备注，默认为空。
- `Country` (str): 国家，默认为PRC。
- `Specialcase` (str): FAS的系统能识别的备注，默认为空。
- `Remark Internal - Local Language` (str): 没有用的字段，默认为空。
- `Remark External - Local Language` (str): 没有用的字段，默认为空。