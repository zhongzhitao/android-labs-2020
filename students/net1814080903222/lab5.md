# 实验五：Android存储编程

## 一、实验目标

1. 了解Android的存储手段；
2. 掌握Android的文件存储；
3. 掌握Android的数据库存储。

## 二、实验要求

1. 应用数据存储可采用数据库存储；
2. 将应用产生的数据存储到数据库中；
3. 将应用运行结果截图。

## 三、实验步骤

1. 在AndroidManifest里获取必要的权限；

    ``` xml
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    ```

2. 为了将排除的文件夹列表放置在数据库中，采用官方指导推荐的Room库来操作内置的SQLite数据库，需要描述所要存取数据的表结构ExcludeItem.java，然后描述操作表的类ExcludeItemDao.java，然后定义总的数据库（在这个题目中仅有一个表）的数据结构ExcludeItemDatabase.java；
3. 使用Room注解创建保存排除列表的表格式。

    ``` java
    @Entity
    public class ExcludeItem {
        @PrimaryKey
        @NotNull
        @ColumnInfo(name = "item_path")
        private String itemPath;

        public ExcludeItem(@NotNull String itemPath) {
            this.itemPath = itemPath;
        }

        @NotNull
        public String getItemPath() {
            return itemPath;
        }

        public void setItemPath(@NotNull String itemPath) {
            this.itemPath = itemPath;
        }
    }
    ```

4. 创建排除列表的数据库交互类ExcludeItemDao

    ``` java
    @Dao
    public interface ExcludeItemDao {
        @Query("SELECT * FROM ExcludeItem")
        LiveData<List<ExcludeItem>> getAll();

        @Insert
        void insertAll(ExcludeItem... items);

        @Delete
        void delete(ExcludeItem item);
    }
    ```

5. 创建数据库的统一接入点ExcludeItemDatabase

    ``` java
    @Database(entities = {ExcludeItem.class}, version = 1)
    public abstract class ExcludeItemDatabase extends RoomDatabase {
        public abstract ExcludeItemDao excludeItemDao();
    }
    ```

6. Database的交互对象应该保证单例设计模式，所以创建一个单例的数据库实例类StorageHelper

    ``` java
    public class StorageHelper {
        private static ExcludeItemDatabase database;
        private static StorageHelper helper;

        public StorageHelper(Context context) {
            database = Room.databaseBuilder(context,
                    ExcludeItemDatabase.class, "kGallery").build();
            helper = this;
        }

        static public StorageHelper getInstance() {
            return helper;
        }

        public ExcludeItemDatabase getDatabase() {
            return database;
        }
    }
    ```

7. 修改原先使用“死数据”的代码片段改用“活数据”。

    ``` java
    public class SettingViewModel extends ViewModel {

        private final ExcludeItemDatabase db;

        public SettingViewModel() {
            db = StorageHelper.getInstance().getDatabase();
        }

        LiveData<List<ExcludeItem>> getExcludeList() {
            return db.excludeItemDao().getAll();
        }
    }
    ```

## 四、实验结果

![database](https://raw.githubusercontent.com/zhongzhitao/android-labs-2020/master/students/net1814080903222/lab5.png)

## 五、实验心得

第五次实验有些难度，但是学到许多，特别是Room库的使用，在过程中发现Room会记录表的变动，能够让开发者定义迁移不同表版本的代码，比起直接写操作SQLite的代码，兼容性，和跨版本升级的能力更强。
