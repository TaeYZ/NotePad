# 安卓移动期中实验：NotePad笔记本应用

## 一、试验内容
本实验将基于NotePad应用做功能扩展

## 二、试验代码与运行截图

1.NoteList中显示条目增加时间戳显示

代码：

notelist_item.xml

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout android:layout_height="match_parent"
    android:layout_width="match_parent"
    android:orientation = "vertical"
    xmlns:android="http://schemas.android.com/apk/res/android">
    <TextView xmlns:android="http://schemas.android.com/apk/res/android"
        android:id="@android:id/text1"
        android:layout_width="match_parent"
        android:layout_height="?android:attr/listPreferredItemHeight"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:gravity="center_vertical"
        android:paddingLeft="5dip"
        android:singleLine="true"

        ></TextView>
    <TextView
        android:id="@+id/text2"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:textColor="#212121"
        android:textSize="20sp"
        />
</LinearLayout>
```

NotePadProvider.java数据库设计：

```
@Override
    public void onCreate(SQLiteDatabase db) {
        db.execSQL("CREATE TABLE " + NotePad.Notes.TABLE_NAME + " ("
            + NotePad.Notes._ID + " INTEGER PRIMARY KEY,"
            + NotePad.Notes.COLUMN_NAME_TITLE + " TEXT,"
            + NotePad.Notes.COLUMN_NAME_NOTE + " TEXT,"
            + NotePad.Notes.COLUMN_NAME_CREATE_DATE + " INTEGER,"
            + NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE + " INTEGER"
            + ");");
    }
```

修改NotePadProvider.java中insert方法和NoteEditor.java中的updateNote方法

```
Long now = Long.valueOf(System.currentTimeMillis());
Date date = new Date(now);
SimpleDateFormat format = new SimpleDateFormat("yyyy.MM.dd HH:mm:ss");
String dateTime = format.format(date);
```

截图：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190517153308327.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1YWlxaTgzNzg=,size_16,color_FFFFFF,t_70)

2.添加笔记查询功能（根据标题查询）

代码：

list_options_menu.xml

```
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <!--  This is our one standard application action (creating a new note). -->
    <item android:id="@+id/menu_add"
          android:icon="@drawable/ic_menu_compose"
          android:title="@string/menu_add"
          android:alphabeticShortcut='a'
          app:showAsAction="always" />
    <!--  If there is currently data in the clipboard, this adds a PASTE menu item to the menu
          <!- so that the user can paste in the data.. &ndash;&gt;-->
    <item
        android:id="@+id/menu_search"
        android:icon="@drawable/search"
        android:title="@string/menu_search"
        android:alphabeticShortcut="b"
        />
    <item
        android:id="@+id/menu_sort"
        android:title="@string/menu_sort"
        android:icon="@android:drawable/ic_menu_sort_by_size"
        app:showAsAction="always" >
        <menu>
            <item
                android:id="@+id/menu_sort1"
                android:title="@string/menu_sort1"/>
            <item
                android:id="@+id/menu_sort2"
                android:title="@string/menu_sort2"/>
            <item
                android:id="@+id/menu_sort3"
                android:title="@string/menu_sort3"/>
        </menu>
    </item>
    <item android:id="@+id/menu_paste"
        android:icon="@drawable/ic_menu_compose"
        android:title="@string/menu_paste"
        android:alphabeticShortcut='p' />
</menu>
```

NoteSearch.java

```
public class NoteSearch extends ListActivity implements SearchView.OnQueryTextListener{
    private static final String[] PROJECTION = new String[]{
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,//在这里加入了修改时间的显示
    };
    SearchView mSearchView;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.note_search);
        Intent intent = getIntent();// Gets the intent that started this Activity.
// If there is no data associated with the Intent, sets the data to the default URI, which
// accesses a list of notes.
        if (intent.getData()==null) {
            intent.setData(NotePad.Notes.CONTENT_URI);
        }
        ListView list = (ListView)findViewById(android.R.id.list);
        list.setOnCreateContextMenuListener(this);

        mSearchView = (SearchView)findViewById(R.id.search_view);//注册监听器
        mSearchView.setIconifiedByDefault(false); //显示搜索的天幕，默认只有一个放大镜图标
        mSearchView.setSubmitButtonEnabled(true); //显示搜索按钮
        mSearchView.setOnQueryTextListener(this);
    }
    @Override
    public boolean onQueryTextSubmit(String s) {
        return false;
    }
    @Override
    public boolean onQueryTextChange(String s) { //Test改变的时候执行的内容
        //Text发生改变时执行的内容
        String selection = NotePad.Notes.COLUMN_NAME_TITLE + " Like ? ";//查询条件
        String[] selectionArgs = { "%"+s+"%" };//查询条件参数，配合selection参数使用,%通配多个字符

        //查询数据库中的内容,当我们使用 SQLiteDatabase.query()方法时，就会得到Cursor对象， Cursor所指向的就是每一条数据。
        //managedQuery(Uri, String[], String, String[], String)等同于Context.getContentResolver().query()
        Cursor cursor = managedQuery(
                getIntent().getData(),            // Use the default content URI for the provider.用于ContentProvider查询的URI，从这个URI获取数据
                PROJECTION,                       // Return the note ID and title for each note. and modifcation date.用于标识uri中有哪些columns需要包含在返回的Cursor对象中
                selection,                        // 作为查询的过滤参数，也就是过滤出符合selection的数据，类似于SQL的Where语句之后的条件选择
                selectionArgs,                    // 查询条件参数，配合selection参数使用
                NotePad.Notes.DEFAULT_SORT_ORDER  // Use the default sort order.查询结果的排序方式，按照某个columns来排序，例：String sortOrder = NotePad.Notes.COLUMN_NAME_TITLE
        );

        //一个简单的适配器，将游标中的数据映射到布局文件中的TextView控件或者ImageView控件中
        String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,  NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE };
        int[] viewIDs = { android.R.id.text1 , R.id.text2 };
        SimpleCursorAdapter adapter = new SimpleCursorAdapter(
                this,                   //context:上下文
                R.layout.noteslist_item,         //layout:布局文件，至少有int[]的所有视图
                cursor,                          //cursor：游标
                dataColumns,                     //from：绑定到视图的数据
                viewIDs                          //to:用来展示from数组中数据的视图
                //flags：用来确定适配器行为的标志，Android3.0之后淘汰
        );
        setListAdapter(adapter);
        return true;
    }
    @Override
    protected void onListItemClick(ListView l, View v, int position, long id) {
        // Constructs a new URI from the incoming URI and the row ID
        Uri uri = ContentUris.withAppendedId(getIntent().getData(), id);
        // Gets the action from the incoming Intent
        String action = getIntent().getAction();
        // Handles requests for note data
        if (Intent.ACTION_PICK.equals(action) || Intent.ACTION_GET_CONTENT.equals(action)) {
            // Sets the result to return to the component that called this Activity. The
            // result contains the new URI
            setResult(RESULT_OK, new Intent().setData(uri));
        } else {
            // Sends out an Intent to start an Activity that can handle ACTION_EDIT. The
            // Intent's data is the note ID URI. The effect is to call NoteEdit.
            startActivity(new Intent(Intent.ACTION_EDIT, uri));
        }
    }
}
```

在AndroidManifest.xml注册NoteSearch

```
 <activity
            android:name=".NoteSearch"
            android:label="@string/title_activity_note_search"
            android:theme="@style/AppTheme.NoActionBar">
            <intent-filter>
                <action android:name="android.intent.action.NoteSearch" />
                <action android:name="android.intent.action.SEARCH" />
                <action android:name="android.intent.action.SEARCH_LONG_PRESS" />

                <category android:name="android.intent.category.DEFAULT" />

                <data android:mimeType="vnd.android.cursor.dir/vnd.google.note" />
            </intent-filter>
        </activity>
```

截图：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190517153319749.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1YWlxaTgzNzg=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190517153328569.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1YWlxaTgzNzg=,size_16,color_FFFFFF,t_70)

3.拓展功能：UI美化（将记事本列表背景设置为白色，将记事本内容背景颜色显示在记事本列表上，并将英文g功能选项改成中文）

代码：

String.xml

 ```
 <?xml version="1.0" encoding="utf-8"?>

<resources>
    <string name="app_name">NotePad</string>
    <string name="live_folder_name">NotePad</string>

    <string name="title_edit_title">笔记标题:</string>
    <string name="title_create">新笔记</string>
    <string name="title_edit">编辑: %1$s</string>
    <string name="title_notes_list">NotePad</string>

    <string name="menu_add">添加记事本</string>
    <string name="menu_save">保存</string>
    <string name="menu_delete">删除</string>
    <string name="menu_open">打开</string>
    <string name="menu_revert">撤销修改</string>
    <string name="menu_copy">复制</string>
    <string name="menu_paste">粘贴</string>
    <string name="menu_search">搜索</string>
    <string name="menu_sort">排序</string>
    <string name="menu_sort1">按创建时间排序</string>
    <string name="menu_sort2">按修改时间排序</string>
    <string name="menu_sort3">按颜色排序</string>

    <string name="button_ok">OK</string>
    <string name="text_title">Title:</string>

    <string name="resolve_edit">编辑笔记</string>
    <string name="resolve_title">编辑标题</string>

    <string name="error_title">错误</string>
    <string name="error_message">加载笔记错误</string>
    <string name="nothing_to_save">没有内容需要保存</string>
    <string name="title_activity_note_search">笔记搜索</string>
    <string name="title_activity_output_text">输出笔记</string>
</resources>
 ```
 
 截图：
 
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190517153336392.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1YWlxaTgzNzg=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190517153344596.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1YWlxaTgzNzg=,size_16,color_FFFFFF,t_70)

4.拓展功能：更改记事本的背景

代码：

NotePad.java

 ```
public static final String COLUMN_NAME_BACK_COLOR = "color";
public static final int DEFAULT_COLOR = 0; //白
public static final int YELLOW_COLOR = 1; //黄
public static final int BLUE_COLOR = 2; //蓝
public static final int GREEN_COLOR = 3; //绿
public static final int RED_COLOR = 4; //红
```

NotePadProvider.java：

```
@Override
    public void onCreate(SQLiteDatabase db) {
        db.execSQL("CREATE TABLE " + NotePad.Notes.TABLE_NAME + "   ("
        + NotePad.Notes._ID + " INTEGER PRIMARY KEY,"
        + NotePad.Notes.COLUMN_NAME_TITLE + " TEXT,"
        + NotePad.Notes.COLUMN_NAME_NOTE + " TEXT,"
        + NotePad.Notes.COLUMN_NAME_CREATE_DATE + " INTEGER,"
        + NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE + " INTEGER,"
        + NotePad.Notes.COLUMN_NAME_BACK_COLOR + " INTEGER" //颜色
        + ");");
       }
 static{}中
 sNotesProjectionMap.put(
        NotePad.Notes.COLUMN_NAME_BACK_COLOR,
        NotePad.Notes.COLUMN_NAME_BACK_COLOR);
 insert中
 if (values.containsKey(NotePad.Notes.COLUMN_NAME_BACK_COLOR) == false) {
        values.put(NotePad.Notes.COLUMN_NAME_BACK_COLOR, NotePad.Notes.DEFAULT_COLOR);
        }
 MyCurcorAdapter:
 package com.example.android.notepad;

import android.content.Context;
import android.database.Cursor;
import android.graphics.Color;
import android.view.View;
import android.widget.SimpleCursorAdapter;

public class MyCursorAdapter extends SimpleCursorAdapter {
    public MyCursorAdapter(Context context, int layout, Cursor c,
                           String[] from, int[] to) {
        super(context, layout, c, from, to);
    }
    @Override
    public void bindView(View view, Context context, Cursor cursor){
        super.bindView(view, context, cursor);
        //从数据库中读取的cursor中获取笔记列表对应的颜色数据，并设置笔记颜色
        int x = cursor.getInt(cursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_BACK_COLOR));
        /**
         * 白 255 255 255
         * 黄 247 216 133
         * 蓝 165 202 237
         * 绿 161 214 174
         * 红 244 149 133
         */
        switch (x){
            case NotePad.Notes.DEFAULT_COLOR:
                view.setBackgroundColor(Color.rgb(255, 255, 255));
                break;
            case NotePad.Notes.YELLOW_COLOR:
                view.setBackgroundColor(Color.rgb(247, 216, 133));
                break;
            case NotePad.Notes.BLUE_COLOR:
                view.setBackgroundColor(Color.rgb(165, 202, 237));
                break;
            case NotePad.Notes.GREEN_COLOR:
                view.setBackgroundColor(Color.rgb(161, 214, 174));
                break;
            case NotePad.Notes.RED_COLOR:
                view.setBackgroundColor(Color.rgb(244, 149, 133));
                break;
            default:
                view.setBackgroundColor(Color.rgb(255, 255, 255));
                break;
        }
    }
}
```

Nodelist.java：

```
private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            //扩展 显示时间 颜色
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // 2
            NotePad.Notes.COLUMN_NAME_BACK_COLOR,
    };
adapter = new MyCursorAdapter(
        this,
        R.layout.noteslist_item,
        cursor,
        dataColumns,
        viewIDs
    );
```
    
PROJECTION

```
private static final String[] PROJECTION =
        new String[] {
            NotePad.Notes._ID,
            NotePad.Notes.COLUMN_NAME_TITLE,
            NotePad.Notes.COLUMN_NAME_NOTE,
            NotePad.Notes.COLUMN_NAME_BACK_COLOR
    };
```
    
NoteEditor.java：

```
int x = mCursor.getInt(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_BACK_COLOR));
    /**
    * 白 255 255 255
    * 黄 247 216 133
    * 蓝 165 202 237
    * 绿 161 214 174
    * 红 244 149 133
    */
    switch (x){
        case NotePad.Notes.DEFAULT_COLOR:
            mText.setBackgroundColor(Color.rgb(255, 255, 255));
            break;
        case NotePad.Notes.YELLOW_COLOR:
            mText.setBackgroundColor(Color.rgb(247, 216, 133));
            break;
        case NotePad.Notes.BLUE_COLOR:
            mText.setBackgroundColor(Color.rgb(165, 202, 237));
            break;
        case NotePad.Notes.GREEN_COLOR:
            mText.setBackgroundColor(Color.rgb(161, 214, 174));
            break;
        case NotePad.Notes.RED_COLOR:
            mText.setBackgroundColor(Color.rgb(244, 149, 133));
            break;
        default:
            mText.setBackgroundColor(Color.rgb(255, 255, 255));
            break;
    }
```      
        
NoteEditor.java

```
case R.id.menu_color:
        changeColor();
        break;
  private final void changeColor() {
        Intent intent = new Intent(null,mUri);
        intent.setClass(NoteEditor.this,NoteColor.class);
        NoteEditor.this.startActivity(intent);
    }
```

NoteColor.java

```
public class NoteColor extends Activity {
    private Cursor mCursor;
    private Uri mUri;
    private int color;
    private static final int COLUMN_INDEX_TITLE = 1;
    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_BACK_COLOR,
    };
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.note_color);
        //从NoteEditor传入的uri
        mUri = getIntent().getData();
        mCursor = managedQuery(
                mUri,        // The URI for the note that is to be retrieved.
                PROJECTION,  // The columns to retrieve
                null,        // No selection criteria are used, so no where columns are needed.
                null,        // No where columns are used, so no where values are needed.
                null         // No sort order is needed.
        );
    }
    @Override
    protected void onResume(){
        //执行顺序在onCreate之后
        if (mCursor != null) {
            mCursor.moveToFirst();
            color = mCursor.getInt(COLUMN_INDEX_TITLE);
        }
        super.onResume();
    }
    @Override
    protected void onPause() {
        //执行顺序在finish()之后，将选择的颜色存入数据库
        super.onPause();
        ContentValues values = new ContentValues();
        values.put(NotePad.Notes.COLUMN_NAME_BACK_COLOR, color);
        getContentResolver().update(mUri, values, null, null);
    }
    public void white(View view){
        color = NotePad.Notes.DEFAULT_COLOR;
        finish();
    }
    public void yellow(View view){
        color = NotePad.Notes.YELLOW_COLOR;
        finish();
    }
    public void blue(View view){
        color = NotePad.Notes.BLUE_COLOR;
        finish();
    }
    public void green(View view){
        color = NotePad.Notes.GREEN_COLOR;
        finish();
    }
    public void red(View view){
        color = NotePad.Notes.RED_COLOR;
        finish();
    }

}
```

note_color.xml

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="horizontal" android:layout_width="match_parent"
    android:layout_height="match_parent">
    <ImageButton
        android:id="@+id/color_white"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="#FFFFFF"
        android:onClick="white"/>
    <ImageButton
        android:id="@+id/color_yellow"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="#EEEE00"
        android:onClick="yellow"/>
    <ImageButton
        android:id="@+id/color_green"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="#00EE00"
        android:onClick="green"/>
    <ImageButton
        android:id="@+id/color_blue"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="#436EEE"
        android:onClick="blue"/>
    <ImageButton
        android:id="@+id/color_red"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="#FF0000"
        android:onClick="red"/>
</LinearLayout>
 ```
 
AndroidManifest.xml

 ```
<activity
            android:name=".NoteColor"
            android:label="ChangeColor"
            android:theme="@android:style/Theme.Holo.Light.Dialog"
            android:windowSoftInputMode="stateVisible" />
 ```
 
 截图：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190517153400621.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1YWlxaTgzNzg=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190517153409992.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1YWlxaTgzNzg=,size_16,color_FFFFFF,t_70)

5.拓展功能：记事本导出

代码：

 ```
 ```
 
 截图：
 
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190517153418102.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1YWlxaTgzNzg=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190517153424901.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1YWlxaTgzNzg=,size_16,color_FFFFFF,t_70)
