#### GreenDao3源码分析

GreenDao是Android中热门的数据库框架之一，以速度快、性能好著称。本篇我们来分析GreenDao3的源码。

#### 基本使用

1. 引入依赖

	- 项目根build.gradle

```
buildscript {
    repositories {
        jcenter()
        mavenCentral() 
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.1.4'
        classpath 'org.greenrobot:greendao-gradle-plugin:3.2.2' 
    }
}
```

	- 具体模块build.gradle

```
//省略其他配置...

//引入插件
apply plugin: 'org.greenrobot.greendao'

//greenDao配置
greendao {
    //数据库版本
    schemaVersion 1
    //生成的类存放目录
    daoPackage '包名.db'
    targetGenDir 'src/main/java'
}

dependencies {
	//引入greenDao依赖
	implementation 'org.greenrobot:greendao:3.2.2'
}

//...省略其他配置
```

2. 定义表映射实体Entity

```
@Entity(nameInDb = "person_info")
public class PersonInfoEntity {
    //主键
    @Id
    private String id;
    @Property(nameInDb = "person_name")
    
    //用户名
    @NotNull
    private String personName;
    
    //用户性别
    @Property(nameInDb = "sex")
    @NotNull
    private String sex;

    //用户年龄
    @Property(nameInDb = "age")
    @NotNull
    private Integer age;
}
```

3. Build项目，会生成3个类，分别是：

	- DaoMaster，所有Dao的主人，负责整个库的运行，内部有一个静态抽象类OpenHelper，继承于Android的SqliteOpenHelper。
	- DaoSession，会话层对象，主要用于生成每个Dao对象，以及Dao的注册、Dao的操作。
	- PersonInfoEntityDao，PersonInfoEntity实体对应的Dao类。

4.使用Dao对象操作数据表

```
//获取数据库SQLiteDatabase
DaoMaster.DevOpenHelper devOpenHelper = new DaoMaster.DevOpenHelper(this, "app_database");
SQLiteDatabase database = devOpenHelper.getWritableDatabase();
//创建DaoMaster
DaoMaster daoMaster = new DaoMaster(database);
//开启会话
DaoSession daoSession = daoMaster.newSession();
//获取具体的表的Dao对象
PersonInfoEntityDao dao = daoSession.getPersonInfoEntityDao();

PersonInfoEntity entity = new PersonInfoEntity();
//...省略entity的配置

//增
dao.insert(entity);

//删
dao.delete(entity);

//查（查询所有数据）
List<PersonInfoEntity> result = dao.loadAll();

//改
dao.update(entity);
```

#### 源码分析

1. 创建OpenHelper，无论怎么封装，只要是使用了Android的Sqlite，都需要创建Android提供的SQLiteOpenHelper，使用GreenDao，我们需要新建一个DaoMaster中的DevOpenHelper，或者间接继承DevOpenHelper。

```
//获取数据库SQLiteDatabase
DaoMaster.DevOpenHelper devOpenHelper = new DaoMaster.DevOpenHelper(this, "app_database");
```

那么我们来看下DaoMaster，我们创建的是DevOpenHelper，它继承于OpenHelper，在升级时会删除所有表再重新创建，OpenHelper又继承于DatabaseOpenHelper，我们继续跟踪DatabaseOpenHelper。

```
public class DaoMaster extends AbstractDaoMaster {
        public static abstract class OpenHelper extends DatabaseOpenHelper {
        public OpenHelper(Context context, String name) {
            super(context, name, SCHEMA_VERSION);
        }

        public OpenHelper(Context context, String name, CursorFactory factory) {
            super(context, name, factory, SCHEMA_VERSION);
        }

        @Override
        public void onCreate(Database db) {
            Log.i("greenDAO", "Creating tables for schema version " + SCHEMA_VERSION);
            createAllTables(db, false);
        }
    }

    /**
     * 开发时使用的DevOpenHelper继承上面的OpenHelper，当数据库升级时，先删除所有表，再重新创建
     */
    public static class DevOpenHelper extends OpenHelper {
        public DevOpenHelper(Context context, String name) {
            super(context, name);
        }

        public DevOpenHelper(Context context, String name, CursorFactory factory) {
            super(context, name, factory);
        }

        @Override
        public void onUpgrade(Database db, int oldVersion, int newVersion) {
            Log.i("greenDAO", "Upgrading schema from version " + oldVersion + " to " + newVersion + " by dropping all tables");
            dropAllTables(db, true);
            onCreate(db);
        }
    }
}
```

2. DatabaseOpenHelper继承于Android原生的SQLiteOpenHelper。DatabaseOpenHelper中有一个EncryptedHelper，它是加密型数据库的OpenHelper，也继承于SQLiteOpenHelper。

	- getWritableDb()，getReadableDb()，获取标准数据库的读写数据库对象。
	- getEncryptedWritableDb()，getEncryptedReadableDb()，获取加密型数据库的读写数据库对象。
	- getWritableDb()，getReadableDb()，这2个方法都调用了一个wrap()方法，创建了一个StandardDatabase类对象，形参为SQLiteDatabase参数，返回值为Database。
	- getEncryptedWritableDb()，getEncryptedReadableDb()，这2个方法都调用了EncryptedHelper的wrap()方法，形参为net.sqlcipher.database.SQLiteDatabase，返回值为Database。
	- 这2种wrap()方法都返回了一个Database对象。既然能返回，那么StandardDatabase和EncryptedDatabase肯定都是Database类的子类。

```
public abstract class DatabaseOpenHelper extends SQLiteOpenHelper {
    private final Context context;
    private final String name;
    private final int version;

    private EncryptedHelper encryptedHelper;
    private boolean loadSQLCipherNativeLibs = true;

    public DatabaseOpenHelper(Context context, String name, int version) {
        this(context, name, null, version);
    }

    public DatabaseOpenHelper(Context context, String name, CursorFactory factory, int version) {
        super(context, name, factory, version);
        this.context = context;
        this.name = name;
        this.version = version;
    }
    
    //获取标准数据库
    public Database getWritableDb() {
        return wrap(getWritableDatabase());
    }

    //获取标准数据库
    //getReadableDb()调用了wrap()包裹方法
    public Database getReadableDb() {
        return wrap(getReadableDatabase());
    }

    //重点：包裹方法
    protected Database wrap(SQLiteDatabase sqLiteDatabase) {
        return new StandardDatabase(sqLiteDatabase);
    }
    
    //...省略其他代码
    
    //获取加密型数据库
    public Database getEncryptedWritableDb(String password) {
        EncryptedHelper encryptedHelper = checkEncryptedHelper();
        return encryptedHelper.wrap(encryptedHelper.getWritableDatabase(password));
    }
    
    //获取加密型数据库
    public Database getEncryptedReadableDb(String password) {
        EncryptedHelper encryptedHelper = checkEncryptedHelper();
        return encryptedHelper.wrap(encryptedHelper.getReadableDatabase(password));
    }
    
    //...省略其他代码
    
    //加密型数据库的SQLiteOpenHelper
    private class EncryptedHelper extends net.sqlcipher.database.SQLiteOpenHelper {
        //...省略其他代码
    
        //重点：wrap()包裹方法
        protected Database wrap(net.sqlcipher.database.SQLiteDatabase sqLiteDatabase) {
            return new EncryptedDatabase(sqLiteDatabase);
        }
        
        //...省略其他代码
    }
}
```