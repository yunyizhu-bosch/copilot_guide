# 代码模板库

本文档提供标准化的、可复用的代码示例，以确保代码风格统一和开发效率。使用时可根据具体的场景替换变量的名称以及赋值

---

### 1. 数据读取

#### 从MS SQL Server 数据库读取

```python
import pyodbc

def get_data_from_mssql(server_name:str, db_name:str, query: str):
    """
    从 MSSQL 数据库执行查询并返回 DataFrame。
    """
    try:
        # 从环境变量获取连接信息，更安全
        conn_str = "Driver={SQL Server}"+";Server={};Database={};Trusted_Connection=yes;".format(server_name,db_name)
        conn = pyodbc.connect(conn_str)
        
        df = pd.read_sql(query, conn)
        print("数据读取成功。")
        return df
    except Exception as e:
        print(f"数据读取失败: {e}")
        return pd.DataFrame()

# 使用示例:
# query = "SELECT * FROM orders LIMIT 10;"
# orders_df = get_data_from_postgres(query)
```

---

### 2. 数据处理