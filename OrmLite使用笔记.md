## OrmLite使用笔记[^1]



#### 1. 第一步添加依赖

`	compile 'com.j256.ormlite:ormlite-android:5.0'`

 或`http://ormlite.com/releases/` 下载jar包

#### 2. 建立数据库实体类 

```java
@DatabaseTable(tableName = "tab_violation")  //定义表名
public class Violation {

    //generatedId 定义 主键 自增长，
    //columnName 定义该字段在数据库中的列名
    //使用generateId=true，则id由数据库自己维护，自动增长效果
    //设置注解 id=true 时id可以由我们自己赋值
    @DatabaseField(columnName = "id",generatedId = true)
    private int id;

    @DatabaseField(columnName = "viol_count")
    private int violCount;

    @DatabaseField(columnName = "car_id")
    private String carId;

    @DatabaseField(columnName = "minut_sun")
    private int minuteSun;

    @DatabaseField(columnName = "money_sun")
    private int moneySun;
	
    // 定义一对多关系, 违章包含许多违章详情
    // 必须用ForeignCollection 不能用ArrayList
    // Collection : 集合, List实现了Collection 接口
   // @ForeignCollectionField(eager = false)
    //private ForeignCollection <Result> violEvent;

    public Violation(){} //必须要添加无参构造方法

    public Violation(int violCount, String carId, int minute, int money) {
        this.violCount = violCount;
        this.carId = carId;
        this.minuteSun = minute;
        this.moneySun = money;
    }

	//代码省略,大量生成get和set
}

@DatabaseTable(tableName = "tab_violinfo")
public class ViolInfo {
    
    @DatabaseField(columnName = "id",generatedId = true)
    private int id;
    
    // foreign = true 定义外键
    // foreignAutoRefresh = true 自动刷新,为了省事最好写上,不然通过外键获取到的只有id
    // 外键如果自定义id, 那么默认为 viol_id 
    @DatabaseField(foreign = true,foreignAutoRefresh = true)
    private Violation viol;
    
    @DatabaseField(columnName = "viol_time")
    private String violTime;
    
    @DatabaseField(columnName = "viol_loc")
    private String violLoc;
    
    @DatabaseField(columnName = "viol_status")
    private boolean violStatus;
    
    @DatabaseField(columnName = "viol_Info")
    private String violInfo;
    
    @DatabaseField(columnName = "viol_time")
    private String violTime;
    
    @DatabaseField(columnName = "viol_minute")
    private String violMinute;
    
    @DatabaseField(columnName = "viol_money")
    private String violMoney;
   
}
```



####3.将实体类与数据库关联

```java
public class DataBaseHelper extends OrmLiteSqliteOpenHelper {

    private static final String TABLE_NAME = "text.db";
    private static DataBaseHelper helper ;

    // Dao<T,ID> 泛型接口, T 代表 Dao实体类, ID 代表主键类型
    //private Dao<Violation,Integer> violationDao;

    // 用来存放Dao的 Map
    private Map<String, Dao> daos = new HashMap<String, Dao>();

    // 私有化构造方法, 不能new对象, 是实现单例模式的一部分
    private DataBaseHelper (Context context){
        // 参数分别为 上下文, 表名 , 游标实例(为null就好) , 数据库版本
        super(context,TABLE_NAME,null,1);
    }



    @Override
    public void onCreate(SQLiteDatabase sqLiteDatabase, ConnectionSource connectionSource) {
        try {
            // TableUtils.createTable() 是库自带的建表工具
            TableUtils.createTable(connectionSource, Violation.class);
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }


     //更新表, 即删除后再重新创建
    @Override
    public void onUpgrade(SQLiteDatabase sqLiteDatabase, ConnectionSource connectionSource, int i, int i1) {
        try {
            // 删除表
            TableUtils.dropTable(connectionSource, Violation.class, true);
        }catch (SQLException e) {
            e.printStackTrace();
        }
        onCreate(sqLiteDatabase,connectionSource);
    }

    // 单例模式获取 DataBaseHelper 实例
    public static DataBaseHelper getHelper(Context context){
       context = context.getApplicationContext();
        if (helper == null){
            synchronized (DataBaseHelper.class) {
                if (helper == null)
                    //这里创建对象
                    helper = new DataBaseHelper(context);
            }
        }
        return helper;
    }

    // 获得 Dao 实例
    public synchronized Dao getDao(Class clas)throws SQLException{
        Dao dao = null;
        String className = clas.getSimpleName(); //获得简单类名
        //如果 class 存在 map 中就从map中读取
        //containsKey() 判断是否存在map中
        if (daos.containsKey(clas)){
            dao = daos.get(className);
        }
        // 如果不存在就调用父类的 getDao()方法获得 dao
        // 并将它储存到 map 中, 便于下次调用
        if (dao == null){
            dao = super.getDao(clas);  //由父类方法创建并返回
            daos.put(className,dao);
        }
        return dao;
    }
    
     @Override
    public void close() {
        super.close();
        for (String key :
                daos.keySet()) {
            Dao dao = daos.get(key);
            dao = null;
        }
    }
}
```





####4. 查询

```java
private void readingDataBase() {
    
    	// 获得数据库帮助类实例  (单例模式)
        DataBaseHelper db = DataBaseHelper.getHelper(this);
        try {
            // 获得中间操作层
            violationDao = db.getDao(Violation.class);
            for (int i = 0; i < 5; i++) {
                Violation violation = new Violation(i,"ssss"+1,(i+5),(i+6));
                violationDao.create(violation); //添加数据库
            }
		
            resultDao = db.getDao(Result.class);
            // 获得第一条数据 ueryForFirst()
            Violation v1 = (Violation) violationDao.queryBuilder().queryForFirst();
            Log.d(TAG, "onCreate: "+v1.getMinute());
            for (int i = 0; i < 6; i++) {
                Result result = new Result(...........) //省略
                result.setViolation(v1); //添加外键
                resultDao.create(result); //添加到数据库
            }
            
            // 一对多关系 : 通过 <1>查询<n> 的方法有两种
            // 其一通过少的对象直接查询 <n>
            // List<Result> resultList = (List<Result>) 
            // resultDao.queryBuilder().where().eq("viol_id",v1);
            // 其二直接查询 <1>
            // 简单但必须在<1>中定义 外键容器 : @ForeignCollectionField  
            // ForeignCollection<Result> resultList =  v1.getViolEvent();
            ForeignCollection<Result> resultList =  v1.getViolEvent();
            for (Result r : esultList) {
                Log.d(TAG, "onCreate: "+r.getmoney());
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
```





####5.注意要点

* 定义实体类时必须定义一个空参构造函数
* 外键容器 必须要使用 ForeignCollection 容器 而不能用 List 等容器







####6. 更多



##### 注解参数表


| 字段名         | 参数名             | 说明                                                         |
| -------------- | ------------------ | ------------------------------------------------------------ |
| @DatabaseTable | tableName          | 指定表明,没有将使用类名作为表明                              |
| @DatabaseField | cloumnName         | 指定字段名,不指定则变量名作为字段名                          |
|                | canBeNull          | 是否可以为null                                               |
|                | dataType           | 指定字段的类型                                               |
|                | defaultValue       | 指定默认值                                                   |
|                | width              | 指定长度                                                     |
|                | id                 | 指定字段为id                                                 |
|                | generatedId        | 指定字段为自增长的id                                         |
|                | foreign            | 指定这个字段的对象是一个外键,外键值是这个对象的id            |
|                | useGetSet          | 指定ormlite访问变量使用set,get方法默认使用的是反射机制直接访问变量 |
|                | throwIfNull        | 如果空值抛出异常                                             |
|                | persisted          | 指定是否持久化此变量,默认true                                |
|                | unique             | 字段值唯一                                                   |
|                | uniqueCombo        | 整列的值唯一                                                 |
|                | index              | 索引                                                         |
|                | uniqueIndex        | 唯一索引                                                     |
|                | foreignAutoRefresh | 外键值,自动刷新                                              |
|                | uniqueIndex        | 外键值,自动刷新                                              |
|                | foreignAutoCreate  | 外键不存在时是否自动添加到外间表中                           |
|                | foreignColumnName  | 外键字段指定的外键表中的哪个字段                             |

#####查询方法表


假设有Person实体，对应数据库t_person表。通过该表来讲下述各种查询方法。

| Id   | LastName | FirstName | Address        | City     |
| ---- | -------- | --------- | -------------- | -------- |
| 1    | Adams    | John      | Oxford Street  | London   |
| 2    | Bush     | George    | Fifth Avenue   | New York |
| 3    | Carter   | Thomas    | Changan Street | Beijing  |
| 4    | Gates    | Bill      | Xuanwumen 10   | Beijing  |


**WEHRE子句**

在SQL语句中，经常会用到where语句，where 进行条件筛选。

dao.queryBuilder.()where()方法返回一个where对象，where中提供了很多方法来进行条件筛选,下边逐个讲where中的方法。

方法 ：**eq(columnName,value)**    **等于**（=）equals

使用示范：mDao.queryBuilder().where().eq("id", 2).query();

对应SQL：SELECT * FROM `t_person` WHERE `id` = 2

| Id   | LastName | FirstName | Address | City |
| ---- | -------- | --------- | ------- | ---- |
|  2   | Bush | George | Fifth Avenue | New York |

方法 ：**lt(columnName,value)**    **小于**（<） less than

使用示范：mDao.queryBuilder().where().lt("id", 2).query();

对应SQL：SELECT * FROM `t_person` WHERE `id` < 2
| Id   | LastName | FirstName | Address | City |
| ---- | -------- | --------- | ------- | ---- |
| 1    | Adams | John | Oxford Street | London |


方法 ：**gt(columnName,value)**    **大于**（>） greater than

使用示范：mDao.queryBuilder().where().gt("id", 2).query();

对应SQL：SELECT * FROM `t_person` WHERE `id` > 2
| Id   | LastName | FirstName | Address | City |
| ---- | -------- | --------- | ------- | ---- |
| 3    | Carter | Thomas | Changan Street | Beijin |


方法 ：**ge(columnName,value)**    **大于等于**（>=）greater-than or equals-to

使用示范：mDao.queryBuilder().where().ge("id", 2).query();

对应SQL：SELECT * FROM `t_person` WHERE `id` >= 2
| Id   | LastName | FirstName | Address        | City     |
| ---- | -------- | --------- | -------------- | -------- |
| 2    | Bush   | George | Fifth Avenue   | New York |
| 3    | Carter | Thomas | Changan Street | Beijing  |

方法 ：**le(columnName,value)**    **小于等于**（<=）less than or equals-to

使用示范：mDao.queryBuilder().where().le("id", 2).query();

对应SQL：SELECT * FROM `t_person` WHERE `id` <= 2
| Id   | LastName | FirstName | Address        | City     |
| ---- | -------- | --------- | -------------- | -------- |
| 1    | Adams | John   | Oxford Street | London   |
| 2    | Bush  | George | Fifth Avenue  | New York |

方法 ：**ne(columnName,value)**    **不等于**（<>）not-equal-to

使用示范：mDao.queryBuilder().where().ne("id", 2).query();

对应SQL：SELECT * FROM `t_person` WHERE `id` <> 2

结果：
| Id   | LastName | FirstName | Address        | City     |
| ---- | -------- | --------- | -------------- | -------- |
| 1    | Adams  | John   | Oxford Street  | London  |
| 3    | Carter | Thomas | Changan Street | Beijing |

方法 ：**in(columnName,object…)**   **在...中**  在指定列中匹配object数组所对应的值，返回匹配到的结果行集合

in还有几个重载方法，需要的话可以去看文档或源码

使用示范：mDao.queryBuilder().where().in("id", 1，2).query();

对应SQL：SELECT * FROM `t_person` WHERE `id` IN (1，2 )

结果：
| Id   | LastName | FirstName | Address        | City     |
| ---- | -------- | --------- | -------------- | -------- |
| 1    | Adams | John   | Oxford Street | London   |
| 2    | Bush  | George | Fifth Avenue  | New York |

方法 ：**notIn(columnName,object…)**   **不在...中**  在指定列中匹配object数组所对应的值，返回没有匹配到的结果行集合

notIn还有几个重载方法，需要的话可以去看文档或源码

使用示范：mDao.queryBuilder().where().notIn("id",1,2).query();

对应SQL：SELECT * FROM `t_person` WHERE `id` NOT IN (1 ,2 )

结果：
| Id   | LastName | FirstName | Address        | City     |
| ---- | -------- | --------- | -------------- | -------- |
| 3    | Carter | Thomas | Changan Street | Beijin |

方法 ：**like(columnName,pattern)**    使用%通配符来匹配，指定行数据，返回匹配到的结果

使用示范：mDao.queryBuilder().where().like("LastName", "A%").query(); 匹配A开头的LastName

​                  mDao.queryBuilder().where().like("LastName", “%s").query(); 匹配s结尾的LastName

​                  mDao.queryBuilder().where().like("LastName", “%art%").query(); 匹配中间为art的LastName

对应SQL：SELECT * FROM `t_person` WHERE `LastName` LIKE 'A%'

结果：
| Id   | LastName | FirstName | Address        | City     |
| ---- | -------- | --------- | -------------- | -------- |
| 1    | Adams | John | Oxford Street | London |

方法 ：**between(columnName,low,high)**  **X到Y之间 (闭区间)**  获取指定范围内的结果

使用示范：mDao.queryBuilder().where().between("id", 1, 2).query();   获取id是1到2之间的结果

对应SQL：SELECT * FROM `t_person` WHERE `id` BETWEEN 1 AND 2

结果：
| Id   | LastName | FirstName | Address        | City     |
| ---- | -------- | --------- | -------------- | -------- |
| 1    | Adams | John   | Oxford Street | London   |
| 2    | Bush  | George | Fifth Avenue  | New York |

方法**and()**，**or()**用来组合上述where子语句。进行与，或操作。

方法 ：and()    where子句与操作

使用示范：mDao.queryBuilder().where().lt("id", 3).and().gt("id", 1).query();

对应SQL：SELECT * FROM `t_person` WHERE (`id` < 3 AND `id` > 1 )

结果：
| Id   | LastName | FirstName | Address        | City     |
| ---- | -------- | --------- | -------------- | -------- |
| 2    | Bush | George | Fifth Avenue | New York |
方法 ：or()    where子句或操作

使用示范：mDao.queryBuilder().where().eq("id", 1).or().eq("id", 2).query();

对应SQL：SELECT * FROM `t_person` WHERE (`id` = 1 OR `id` = 2 )

结果：
| Id   | LastName | FirstName | Address        | City     |
| ---- | -------- | --------- | -------------- | -------- |
| 1    | Adams | John   | Oxford Street | London   |
| 2    | Bush  | George | Fifth Avenue  | New York |

ORDER BY

根据指定列名排序，降序，升序

使用示范：mDao.queryBuilder().orderBy("id", false).query(); //参数false表示降序，true表示升序。

对应SQL：SELECT * FROM `t_person` ORDER BY `id` DESC（降序）

结果：

| Id   | LastName | FirstName | Address        | City     |
| ---- | -------- | --------- | -------------- | -------- |
| 4    | Gates    | John      | Oxford Street  | Beijing  |
| 3    | Carter   | Thomas    | Changan Street | Beijing  |
| 2    | Bush     | George    | Fifth Avenue   | New York |
| 1    | Adams    | John      | Oxford Street  | London   |

**DISTINCT**

过滤指定列不重复数据行，重复的只返回一次。

使用示范：mDao.queryBuilder().selectColumns("City").distinct().query();

对应SQL：SELECT DISTINCT `City` FROM `t_person`

结果：

| **City** |
| -------- |
| London   |
| New York |
| Beijing  |

其中Beijing过滤掉了一个。

**GROUP BY**

按照指定列分组

使用示范：mDao.queryBuilder().groupBy("city").query();

对应SQL：SELECT * FROM `t_person` GROUP BY `city`
| Id   | LastName | FirstName | Address        | City     |
| ---- | -------- | --------- | -------------- | -------- |
| 3    | Gates | Thomas | Changan Street | Beijing  |
| 2    | Bush  | George | Fifth Avenue   | New York |
| 1    | Adams | John   | Oxford Street  | London   |

**offset Limit**

offset跳过指定的行数

limit限制获取指定行数

使用示范：mDao.queryBuilder().offset(2).limit(2).query();  可以用来分页

对应SQL：SELECT * FROM `t_person` LIMIT 2 OFFSET 2

结果：
| Id   | LastName | FirstName | Address        | City     |
| ---- | -------- | --------- | -------------- | -------- |
| 3    | Carter | Thomas | Changan Street | Beijing |
| 4    | Gates  | Bill   | Xuanwumen 10   | Beijing |

**Having**

等同于sql中的Having，针对分组数据，进行聚合函数（SUM, COUNT, MAX, AVG）运算。

使用示范：mPersonList = mDao.queryBuilder().groupBy("City").having("SUM(id)>4").query()

对应SQL：SELECT * FROM `t_person` GROUP BY `City` HAVING SUM(id)>4

结果

| 4    | Gates | Bill | Xuanwumen 10 | Beijing |
| ---- | ----- | ---- | ------------ | ------- |
|      |       |      |              |         |

 

**countOf**

返回查询结果的总数

使用示范：mDao.queryBuilder().countOf()

对应SQL：SELECT COUNT(*) FROM `t_person`

结果 4

**iterator**

返回一个结果集的迭代器。

使用示范：Iterator<Person> iterator = mDao.queryBuilder().iterator();

**queryForFirst**

返回所有行的第一行。

使用示范：mDao.queryBuilder().queryForFirst();




[^1]: orm:对象关系映射

