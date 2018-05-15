<meta http-equiv="Content-Type" content="text/html; charset=utf-8">

### 2018.05.12 - First Time to Use Github Page

今天心血來潮，來研究一下Github Page，結果駑鈍如我，花了一個下午的時間，才學會自訂theme，也才知道.md是「markdown」，是基於讓html更易讀及使用的文件標記。
用markdown作筆記真的方便多了，希望我能繼續堅持下去，讓page不會長草。

***

## 資源連結整理

### UI 好用網站懶人包

1.今天在研究這個jekyll theme，發現這個好用的網站[DEVICON](http://konpa.github.io/devicon/)-免費的程式語言、設計、開發工具圖示集. 

***

## programming筆記

### 何謂「強型別」? 何謂「弱型別」?

◆ 強型別：指定變數資料型別 (程式所定義的資料型別 = 變數在Runtime的型別)

      - 讓IntelliSence可以支援變數，在輸入程式碼時可以看到變數的屬性及其他成員
    
      - 可利用編譯器進行型別檢查。在Complie time即可發現錯誤，減少在Runtime發生錯誤的機會
    
      - 執行程式碼速度較快
    
    
◆ 弱型別：以object當變數型別、無法明確表達runtime變數型別者皆屬之

### ADO.NET (請參閱 [VITO學習筆記:使用ADO.NET Conneted類別](http://vito-note.blogspot.tw/2012/09/adonet-connected.html)、 [何謂ADO.NET-藍色小舖](http://www.blueshop.com.tw/board/FUM201412241618269T8/BRD20151103140432TEC.html))

#### DBConnection：

* 分為Connneted Data Source及Disconnected Data Model

* 不同的資料儲存區，會有不同的資料提供者 

  * SQL Server原生資料來源-System.Data.SqlClient    
  * OLE DB資料來源-System.Data.OleDb      
  * Oracle資料來源-System.Data.OracleClient      
  * ODBC資料來源-System.DataOdbc
  
* 使用同一個連線字串建立連線時，系統會至ThreadPool取得連線物件，減少另外建立連線資源消耗

* 傳送Command至資料儲存區前，連線需先開啟

##### 建立SqlConnection :

```csharp
SqlConnetction con = new SqlConnection();
con.ConnectionString = @"Data Source=資料庫IP; Initial Catalog=資料庫名稱; User ID=登入ID; Password=登入密碼;";
```

###### 實例 :


```csharp
protected SqlConnection CreateConnection()
{
  SqlConnetction con = new SqlConnection();
  con.ConnectionString = @"Data Source=資料庫IP; Initial Catalog=資料庫名稱; User ID=登入ID; Password=登入密碼;";
  
  return con;
}

protected void btnCreateConnection_Click(object sender, EventArgs e)
{
  //方法1 - 手動關閉連線
  SqlConnection con = CreateConnection();
  
  try
  {
    con.Open();
  }
  catch (Exception ex)
  {
    throw ex;
  }
  finally
  {
    con.Close();
  }
  
  //方法2 - 使用using
  using (SqlConnetion con = CreateConnection())
  {
    try
    {
      con.Open();
    }
    catch (Exception ex)
    {
      throw ex;
    }
  }
}
```

#### DbCommand :

* 需指定CommandType及CommandText (CommandType預設為Text)

* 亦可透過DbParameter物件傳遞參數

##### 建立DbCommand :

```csharp
SqlCommand cmd = con.CreateCommand();
cmd.CommandType = CommandType.Text;
cmd.CommandText = "Select * From tableName";
```

##### 執行DbCommand :

* ExecuteNonQuery(): 執行不回傳結果集，Ex Insert、Update、Delete
* ExecuteScalar(): 執行並回傳第一列第一行之資料
* ExecuteReader(): 執行並回傳DataReader，以讀取資料集之資料

```csharp
using (SqlConnection con = new SqlConnection("Data Source=資料庫IP; Initial Catalog=資料庫名稱; User ID=登入ID; Password=登入密碼"))
{
	con.Open();
	
	//ExecuteNonQuery()
	SqlCommand cmd1 = con.CreateCommand();
	cmd1.CommandType = CommandType.Text;
	cmd1.CommandText = "Insert PLI_PDT_M(PDT_NAME, VERSION) Values ('心安利變型', '1')";
	cmd1.ExecuteNonQuery();
	
	//ExecuteScalar()
	SqlCommand cmd2 = con.CreateCommand();
	cmd2.CommandType = CommandType.StoredProcedure;
	cmd2.CommandText = "stp_InsertProduct";
	int id = Convert.ToInt32(cmd2.ExecuteScalar());
	
	//ExecuteReader()
	SqlCommand cmd3 = con.CreateCommand();
	cmd3.CommandType = CommandType.Text;
	cmd3.CommandText = "Select * From SCUSERM";
	DbDataReader reader = cmd3.ExecuteReader();
}
```

##### 使用DbDataReader

* 須由ExecuteReader回傳，無法自行建立DbDataReader物件

* 順向唯讀

```csharp
SqlCommand cmd = con.CreateCommand();
cmd.CommandType = CommandType.Text;
cmd.CommandText = "Select * From PLI_PDT_M";

cmd.Connection = con;
con.Open();
reader = cmd.ExecuteReader();

//順向唯讀
while (reader.Read())
{
	if (reader["PDT_CODE"].ToString() == "DBA")
	{
		Console.WriteLine("Find");
		break;
	}
}

//DataReader轉成DataTable
DataTable dt = new DataTable();
dt.Load(reader);
```

#### DbDataAdapter

* 若想取得DbCommand執行結果，叫用ExecuteReader()即可取得，但回傳的資料為一個「連線狀態」的物件。即讀取資料時，必須與資料來源保持連線狀態

* 利用DataAdapter可進一步操作資料庫，它是由一個SqlConnection及數個SqlCommand組成的物件，採離線存取的方式；透過Fill()自動開啟連線並取得資料，在撈完資料後關閉連線，藉由此將查詢結果存入DataSet或DataTable等離線狀態物件，或者利用此將異動後的資料更新回資料庫。

##### Fill()

將資料由資料庫移到指定的物件

```csharp
protected void btnFill_Click(object sender, EventArgs e)
{
	using (SqlConnection con = New SqlConnection("Data Source=資料庫IP; Initial Catalog=資料庫名稱; User ID=登入ID; Password=登入密碼"))
	{
		con.Open();
		
		SqlCommand cmd = con.CreateCommand();
		cmd.CommandType = CommandType.Text;
		cmd.CommandText = "Select * From PLI_PDT_M";
		
		SqlDataAdapter adapter = new SqlDataAdapter(cmd);
		DataSet ds = new DataSet("Produts");
		adpter.Fill(ds, "Products");
	}
	
}
```

##### Update()

使用Update()前，需先設定好DataAdapter物件的四個指令(SelectCommand、UpdateCommand、InsertCommadn、DeleteCommand)，系統方知如何回存資料庫。
但若針對單一資料表，可利用SqlCommandBuilder物件，它會根據SelectCommand指令物件內容，建立其餘三個指令物件。

```csharp
protected void btnUpdate_Click(object sender EventArgs e)
{
	using (SqlConnection con = new SqlConnection(connectString))
	{
		con.Open();
		
		SqlCommand cmd = con.CreateCommand();
		cmd.CommandType = CommandType.Text;
		cmd.CommandText = "Select * From PLI_PDT_M";
		
		SqlDataAdpter adapter = new SqlDataAdapter(cmd);
		
		//建立 SqlCommandBuilder
		SqlCommandBuilder cmdBuilder = new SqlCommandBuilder(adapter);
		
		DataSet ds = new DataSet("Products");
		adapter.Fill(ds, "Products");
		
		ds.Tables[0].Rows[0]["PDT_NAME"] = ds.Tables[0].Rows[0]["PDT_NAME"].ToString() + "_test";
		adapter.Update(ds, "Products");
		
		//觀察SqlCommandBuilder所建立的指令物件
		Console.WriteLine(cmdBuilder.GetInsertCommand().CommandText);
		Console.WriteLine(cmdBuilder.GetUpdateCommand().CommandText);
		Console.WriteLine(cmdBuilder.GetDeleteCommand().CommandText);		
	}
}
```
##### UpdateBatchSize批次更新

執行Update()，是將已變更之資料列，row by row更新至資料庫

我們可以設定UpdateBatchSize屬性，將單次傳送筆數增加，以增加效能

* UpdateBatchSize = 1 預設值

* UpdateBatchSize = 0 無限制

* UpdateBatchSize = n n筆

###### RowUpdateEvent

取得 SQL 陳述式 (Statement) 的執行所變更、插入或刪除的資料列數目

```csharp
protected void btnBatch_Click(object sender, EventArgs e)
{
	using (SqlConnection con = new SqlConnection(connectString))
	{
		con.Open();
		
		SqlCommand cmd = con.CreateCommand();
		cmd.CommandType = CommandType.Text;
		cmd.CommandText = "Select * From PLI_PDT_M";
		
		SqlDataAdapter adapter = new SqlDataAdapter(cmd);
		SqlCommandBuilder cmdBuilder = new SqlCommandBuilder(adapter);
		DataSet ds = new DataSet();
		adapter.Fill(ds);
		
		//更新三筆資料
		ds.Table[0].Rows[0]["PDT_NAME"] = ds.Table[0].Rows[0]["PDT_NAME"].ToString() + "_1";	
		ds.Table[0].Rows[0]["PDT_NAME"] = ds.Table[0].Rows[0]["PDT_NAME"].ToString() + "_2";
		ds.Table[0].Rows[0]["PDT_NAME"] = ds.Table[0].Rows[0]["PDT_NAME"].ToString() + "_3";
		
		//觀察更新事件
		adapter.RowUpdate += SqlRowUpdated;
		
		//設定批次更新之單次傳送筆數
		adapter.UpdateBatchSize = 2;
		adapter.Update(ds);
	}
}

//第一次 RecordsAffected = 2
//第二次 RecordsAffected = 1
protected void SqlRowUpdated(object sender, SqlRowUpdateEventArgs e)
{
	DataRow dr = e.Row;
	Console.WriteLine(e.RecordsAffected.ToString());
}
```



