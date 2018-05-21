<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<head>
	<link rel="stylesheet" type="text/css" media="all" href="/assets/css/markdown8.css" />
</head>

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

#### Transaction物件

```csharp
protected void btnTransaction_Click(object sender EventArgs e)
{
	using (SqlConnetcion con = new SqlConnection(connectionString))
	{
		con.Open();
		
		SqlTransaction tran = con.BeginTransaction();
		
		try
		{
			SqlCommand cmd = new SqlCommand();
			
			//connection及transaction都需指定給command	
			cmd.Connection = con;
			cmd.Transaction = transaction;
			
			cmd.CommandType = CommandType.StoredProcedure;
			cmd.CommandText = "stp_Test1";
			cmd.ExecuteNonQuery();
			cmd.CommandType = CommandType.StoredProcedure;
			cmd.CommandText = "stp_Test2";
			cmd.ExecuteNonQuery();
			
			tran.Commit();
		}
		catch
		{
			tran.Rollback(); //一個單元失敗，就全數復原
		}
	}
}
```

#### 常見錯誤

##### Fill: SelectCommand.Connection 屬性尚未初始化。
```csharp
using (SqlConnection conn = new SqlConnection(connectionString))
{
	try
	{
		conn.Open();
		
		SqlCommand cmd = new SqlCommand();
		cmd.CommandType = CommandType.Text;
		
		// 指定連線後，解決 Fill: SelectCommand.Connection 屬性尚未初始化。
		cmd.Connection = conn;
		cmd.CommandText = "Select (PD_PDTCODE + '   ' + PD_PDTNAME) As fullName From PLI_PDT_M Where PK_CLASS < 21";
	}
}
```



### 資料庫正規化 Normal Form [請參閱 資料庫正規化](http://job.achi.idv.tw/2013/04/21/database-normalization/)

最近聽到前輩提到正規化，驚! 我才發現我想不起來這名詞是在做什麼的

記得當初資料庫管理這堂課我如魚得水，才剛踏入職場沒多久，就全還給教授了((教授對不起



#### 第一正規化 1NF

* 每個欄位僅能存放單一值

* 每筆資料須有唯一主鍵

* 不得以多個欄位表達同一事實 (Ex 喜歡的食物，拆成: 喜歡的食物(1)、喜歡的食物(2)、喜歡的食物(3))

#### 第二正規化 2NF

若某資料表符合第二正規化**若且唯若**

* 符合第一正規化

* 所有非主鍵欄位必與主鍵相關

#### 第三正規化 3NF

在滿足第一及第二正規化的條件下，所有非鍵欄位僅相依於主鍵(非鍵欄位之間應獨立無關)

EX :   訂單編號  |  客戶名稱  |  單價  |  數量  |  小計

小計 = 單價 * 數量 

「小計」相依於非鍵欄位「單價」及「數量」，故不符合3NF，須將小計剔除




### C\#存取詞

* private : 私有型別，僅自身內部類別可存取

* public : 公用型別，自身及外部其他類別皆可存取

* protected : 保護型別，自身及繼承之子類別可存取




### Partial Class

當定義好介面，而有一人以上需要同時開發同個時做類別；或者某個實作類別的功能，有些需要撰寫較多的商業邏輯。此時就可以使用Partial Class，以達成多公及好維護的目的。

###### ITime.cs
```csharp
public interface IGetTime
{
	string GetNow();
	string GetDate();
}
```

###### GetTime.cs
```csharp
public partial class GetTime:IGetTime
{
	return DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss");
}
```

###### GetTimeForSome.cs
```csharp
public partial class GetTime
{
	return DateTime.Now.ToString("yyyy-MM-dd");
}
```

###### Form1.cs
```csharp
public partial class Form1: Form
{
	public Form1()
	{
		InitializeComponent();
	}
	
	private void GetNow_Click(object sender, EventArgs)
	{
		GetTime t = new GetTime();
		MessageBox.Show(t.GetNow());
	}
	
	private void GetDate_Click(object sender, EventArgs e)
	{
		GetTime g = new GetTime();
		MessageBox.Show(g.GetDate());
	}
}
```




### Static (請參閱 [一秒看破static](http://weisnote.blogspot.tw/2012/08/static.html))

* 靜態成員：

  * 靜態方法 屬於「類別」所有
  * 不需要實體(Instance)即可進行訪問
  * 靜態並非沒有實體，而是只有一個實體，在程式執行之初即建立
  * 若使用過多靜態成員，易造成不必要的記憶體浪費
  * 靜態類別內不得有非靜態成員
  
* 非靜態成員：

  * 非靜態方法 屬於「實體」所有
  * 需要new一個實體(Instance)才可進行訪問

**** 說明1

```csharp
public class NotStaticClass {

	public static void StaticMethod() {
		// TODO.
	}
	
	public void NotStaticMethod() {
		// TODO.
	}
}
```

```csharp
//非靜態方法 只有該類別實例可執行
NotStaticClass notStaticClass = new NotStaticClass();
notStaticClass.NotStaticMethod();

//靜態方法 
NotStaticClass.StaticMethod();
```

#### 說明2

```csharp
public static class StaticClass
{
	//靜態類別中僅能存在靜態成員
	public static string userName = "AAA";
	
	public static void Login()
	{
		// TODO.
	}
}

public class NotStaticClass
{
	public static string userName2 = "BBB";
	public void Logout()
	{
		// TODO.
	}
}

class Program
{
	static void Main(string[] args)
	{
		Console.WriteLine(StaticClass.userName);
		Console.WriteLine(StaticClass.Login());
		
		Console.WriteLine(NotStaticClass.userName2);
		
		NotStaticClass nsc = new NotStaticClass();
		Console.WriteLine(NotStaticClass.Logout());
	}
}
```



### HTML - X-UA-Compatible設置IE兼容模式 (請參閱 [黑暗執行緒-搞懂X-UA0Competible相容設定](http://blog.darkthread.net/post-2016-05-26-x-ua-compatible-setting.aspx))

#### IE包含三種文件模式

* Standard Mode(標準模式) : 盡力支援最新HTML5/CSS3/SVG等標準。但不同版本IE支持程度不同
* Quirks Mode(接縫模式) : 力求相容較早版本瀏覽器行為。
* Almost-Standards Mode(準標準模式) : 支援最新標準，但保有先前版本的圖形渲染行為

#### HTML控制文件模式

<!DOCTYPE>及<meta http-equiv="X-UA-Compatible">皆可控制，而IE套用原則為:

* 若網頁同時有<!DOCTYPE>及<meta http-equiv="X-UA-Compatible">，以meta為準
* 若瀏覽器支援<meta http-equiv="X-UA-Compatible">，但不支援meta指定之文件模式，將採用最高文件標準(IE=Edge)
* 舊版瀏覽器如不支援<meta http-equiv="X-UA-Compatible">，由<!DOCTYPE>決定
* IE9(含)以前會以IE5(Quirks Mode)顯示沒有標示<!DOCTYPE>的網頁，建議網頁一律加<!DOCTYPE html>

<meta http-equiv="X-UA-Compatible">應放置於<header>區前段，其上僅允許存在其他meta Header或<title>
	
#### X-UA-Compatible Header寫法

* 限制IE使用IE9/IE8/IE7標準模式
  <meta http-equiv="X-UA-Compatible" content="IE=9">
  <meta http-equiv="X-UA-Compatible" content="IE=8">
  <meta http-equiv="X-UA-Compatible" content="IE=7">

* EmulateIE*
  EmulateIE* 效果視<!DOCTYPE>宣告而定，若依宣告使用標準模式，則IE會依EmulateIE* 決定使用IE9/8/7標準模式；若網頁缺少<!DOCTYPE>宣告   則進入Quirks Mode(IE5)
  <meta http-equiv="X-UA-Compatible" content="IE=EmulateIE9">
  <meta http-equiv="X-UA-Compatible" content="IE=EmulateIE8">
  <meta http-equiv="X-UA-Compatible" content="IE=EmulateIE7">
  
#### 實驗參考程式

[JSbin](http://jsbin.com/ozejuk/1/edit?html,output)

