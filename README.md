## SQLite数据库存储


#### 创建数据库


在mac上查看数据库的工具：SQLPro  for SQLite

数据库文件位于：/data/data/package name/databases/目录下

MyDatabaseHelper.java

```
package com.zhoujian.sqlitedemo;

import android.content.Context;
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteOpenHelper;
import android.widget.Toast;

public class MyDatabaseHelper extends SQLiteOpenHelper {


    private Context mContext;

    //integer：表示整型
    //text:表示文本型
    //real:表示浮点型
    //blob:表示二进制类型

    public static final String CREATE_BOOK = "create table Book ("
            + "id integer primary key autoincrement, "
            + "author text, "
            + "price real, "
            + "pages integer, "
            + "name text)";

    public static final String CREATE_CATEGORY = "create table Category ("
            + "id integer primary key autoincrement, "
            + "category_name text, "
            + "category_code integer)";



    public MyDatabaseHelper(Context context, String name,
                            SQLiteDatabase.CursorFactory factory, int version) {
        super(context, name, factory, version);
        mContext = context;
    }

    @Override
    public void onCreate(SQLiteDatabase db) {
        db.execSQL(CREATE_BOOK);
        db.execSQL(CREATE_CATEGORY);
        Toast.makeText(mContext, "创建数据库成功", Toast.LENGTH_SHORT).show();
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {

        //当数据库升级的时候，如果表存在，会先删除表，然后重新创建
        db.execSQL("drop table if exists Book");
        db.execSQL("drop table if exists Category");
        onCreate(db);
    }

}

```

#### 增删改查

MainActivity.java

```
package com.zhoujian.sqlitedemo;

import android.content.ContentValues;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.util.Log;
import android.view.View;
import android.widget.Button;
import android.widget.Toast;

public class MainActivity extends AppCompatActivity
{

    private MyDatabaseHelper dbHelper;
    private SQLiteDatabase mDb;
    private Button mCreateDatabase;
    private Button mAddData;
    private Button mQueryButton;
    private Button mDeleteButton;
    private Button mUpdateData;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        dbHelper = new MyDatabaseHelper(this, "Book.db", null, 1);
        initViews();
        clickEvents();
    }

    private void initViews()
    {
        mCreateDatabase = (Button) findViewById(R.id.create_database);
        mAddData = (Button) findViewById(R.id.add_data);
        mQueryButton = (Button) findViewById(R.id.query_data);
        mDeleteButton = (Button) findViewById(R.id.delete_data);
        mUpdateData = (Button) findViewById(R.id.update_data);
    }
    private void clickEvents()
    {
        mCreateDatabase.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {

                //getReadableDatabase():如果磁盘已满，将以只读的方式打开数据库
                //getWritableDatabase():如果磁盘已满，将出现异常

                mDb = dbHelper.getWritableDatabase();
            }
        });

        mAddData.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if(mDb!=null){
                    ContentValues values = new ContentValues();
                    // 插入第一条数据
                    values.put("name", "Android开发艺术探索");
                    values.put("author", "任玉刚");
                    values.put("pages", 494);
                    values.put("price", 65.5);
                    mDb.insert("Book", null, values);
                    values.clear();
                    // 插入第二条数据
                    values.put("name", "第一行代码");
                    values.put("author", "郭霖");
                    values.put("pages", 510);
                    values.put("price", 75);
                    mDb.insert("Book", null, values);
                    values.clear();
                    // 插入第三条数据
                    values.put("name", "疯狂Android讲义");
                    values.put("author", "李刚");
                    values.put("pages", 580);
                    values.put("price", 85);
                    mDb.insert("Book", null, values);


                    Toast.makeText(MainActivity.this, "添加数据成功", Toast.LENGTH_SHORT).show();
                }else {
                    Toast.makeText(MainActivity.this, "请先创建数据库", Toast.LENGTH_SHORT).show();
                }
            }
        });

        mUpdateData.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if(mDb!=null){

                    //把Android开发艺术探索这本书的价格改为70
                    ContentValues values = new ContentValues();
                    values.put("price", 70);
                    mDb.update("Book", values, "name = ?", new String[] { "Android开发艺术探索" });
                    Toast.makeText(MainActivity.this, "更新数据成功", Toast.LENGTH_SHORT).show();
                }else {
                    Toast.makeText(MainActivity.this, "请先创建数据库", Toast.LENGTH_SHORT).show();
                }
            }
        });

        mDeleteButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if(mDb!=null){
                    //删除页数大于500的书籍
                    mDb.delete("Book", "pages > ?", new String[] { "500" });
                    Toast.makeText(MainActivity.this, "删除数据成功", Toast.LENGTH_SHORT).show();
                }else {
                    Toast.makeText(MainActivity.this, "请先创建数据库", Toast.LENGTH_SHORT).show();
                }
            }
        });

        mQueryButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {

                if(mDb!=null){

                    // 查询Book表中所有的数据
                    Cursor cursor = mDb.query("Book", null, null, null, null, null, null);
                    if (cursor.moveToFirst()) {
                        do {
                            // 遍历Cursor对象，取出数据并打印
                            String name = cursor.getString(cursor.getColumnIndex("name"));
                            String author = cursor.getString(cursor.getColumnIndex("author"));
                            int pages = cursor.getInt(cursor.getColumnIndex("pages"));
                            double price = cursor.getDouble(cursor.getColumnIndex("price"));
                            Log.d("MainActivity", "书籍的名字： " + name);
                            Log.d("MainActivity", "书籍的作者： " + author);
                            Log.d("MainActivity", "书籍的页数： " + pages);
                            Log.d("MainActivity", "书籍的价格：" + price);
                        } while (cursor.moveToNext());
                    }
                    cursor.close();
                    Toast.makeText(MainActivity.this, "查询数据成功", Toast.LENGTH_SHORT).show();
                }else {
                    Toast.makeText(MainActivity.this, "请先创建数据库", Toast.LENGTH_SHORT).show();
                }
            }
        });
    }
}

```

[Demo下载：https://github.com/zeke123/SqliteDemo](https://github.com/zeke123/SqliteDemo)







