# NotePad笔记本应用
---
## 原应用
图1：NotePad主界面<br>
![NotePadMain](https://github.com/wzs8253669/picture/raw/master/NotePadMain.png)<br>
图2：新建笔记<br>
![NewNoteEdit](https://github.com/wzs8253669/picture/raw/master/NewNoteEdit.png)<br>
图3：新建笔记退回主页面<br>
![NewNoteList](https://github.com/wzs8253669/picture/raw/master/NewNoteList.png)<br>
图4：进入笔记，编辑标题菜单<br>
![EditTitle](https://github.com/wzs8253669/picture/raw/master/EditTitle.png)<br>
图5：编辑标题<br>
![EditTitleDialog](https://github.com/wzs8253669/picture/raw/master/EditTileDialog.png)<br>
图6：笔记列表<br>
![moreNotes](https://github.com/wzs8253669/picture/raw/master/moreNotes.png)<br>
图7：长点“第二条笔记”，菜单<br>
![menu](https://github.com/wzs8253669/picture/raw/master/menu.png)<br>

## 拓展功能
- NotesList中显示条目增加时间显示
- 笔记查询
- UI美化
- 背景更换
- 导出笔记
- 笔记排序

## 拓展功能解析

- NotesList中增加时间显示

```
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@android:id/text1"
    android:layout_width="match_parent"
    android:layout_height="?android:attr/listPreferredItemHeight"
    android:textAppearance="?android:attr/textAppearanceLarge"
    android:gravity="center_vertical"
    android:paddingLeft="5dip"
    android:singleLine="true"
/>
```
要在标题下方加时间显示，就要在标题的TextView下再加一个时间的TextView。
```
<?xml version="1.0" encoding="utf-8"?>
<!--添加一个垂直的线性布局-->
<LinearLayout  xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    <!--原标题TextView-->
    <TextView xmlns:android="http://schemas.android.com/apk/res/android"
        android:id="@android:id/text1"
        android:layout_width="match_parent"
        android:layout_height="?android:attr/listPreferredItemHeight"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:gravity="center_vertical"
        android:paddingLeft="5dip"
        android:textColor="@color/colorBlack"
        android:singleLine="true"
        />
    <!--添加显示时间的TextView-->
    <TextView
        android:id="@+id/text1_time"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textAppearance="?android:attr/textAppearanceSmall"
        android:paddingLeft="5dip"
        android:textColor="@color/colorBlack"/>
</LinearLayout>
```
查看程序如何定义数据库结构<br>
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
通过**Cursor**从数据库中读取出：<br>
```
Cursor cursor = managedQuery(
            getIntent().getData(),            // Use the default content URI for the provider.
            PROJECTION,                       // Return the note ID and title for each note.
            null,                             // No where clause, return all records.
            null,                             // No where clause, therefore no where column values.
            NotePad.Notes.DEFAULT_SORT_ORDER  // Use the default sort order.
        );
```
通过**SimpleCursorAdapter**装填：<br>
```
String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE } ;
int[] viewIDs = { android.R.id.text1 };
SimpleCursorAdapter adapter
    = new SimpleCursorAdapter(
            this, // The Context for the ListView
            R.layout.noteslist_item, // Points to the XML for a list item
            cursor,   // The cursor to get items from
            dataColumns,
            viewIDs
    );
// Sets the ListView's adapter to be the cursor adapter that was just created.
setListAdapter(adapter);
```
修改时间作为显示的时间。<br>
```
 private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            //扩展 显示时间 颜色
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,
            NotePad.Notes.COLUMN_NAME_BACK_COLOR, 
    };
```
在dataColumns，viewIDs中补充时间部分：<br>
```
String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,  NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE } ;
int[] viewIDs = { android.R.id.text1 , R.id.text1_time };
```
dateTime为转化后的时间格式，将其用ContentValues的put方法存入数据库。<br>
```
Long now = Long.valueOf(System.currentTimeMillis());
Date date = new Date(now);
SimpleDateFormat format = new SimpleDateFormat("yyyy.MM.dd HH:mm:ss");
String dateTime = format.format(date);
```
运行效果：<br>
添加笔记：<br>
![time1](https://github.com/wzs8253669/picture/raw/master/time1.jpg)<br>
修改第一条笔记：<br>
![time2](https://github.com/wzs8253669/picture/raw/master/time2.jpg)<br>

- 笔记查询

要添加笔记查询功能，就要在应用中增加一个搜索的入口。<br>
```
<item
    android:id="@+id/menu_search"
    android:title="@string/menu_search"
    android:icon="@android:drawable/ic_search_category_default"
    android:showAsAction="always">
</item>
```
在NotesList中找到onOptionsItemSelected方法，在switch中添加搜索的case语句:<br>
```
 //添加搜素
    case R.id.menu_search:
    Intent intent = new Intent();
    intent.setClass(NotesList.this,NoteSearch.class);
    NotesList.this.startActivity(intent);
    return true;
```
菜单：<br>
![searchmenu](https://github.com/wzs8253669/picture/raw/master/searchmenu.png)<br>
把**SearchView跟ListView相结合**，**动态地显示搜索结果**。<br>
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical" android:layout_width="match_parent"
    android:layout_height="match_parent">
    <SearchView
        android:id="@+id/search_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:iconifiedByDefault="false"
        android:queryHint="输入搜索内容..."
        android:layout_alignParentTop="true">
    </SearchView>
    <ListView
        android:id="@android:id/list"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
    </ListView>
</LinearLayout>
```
要动态地显示搜索结果，就要对SearchView文本变化设置监听，NoteSearch除了要继承ListView外还要实现SearchView.OnQueryTextListener接口：<br>
```
public class NoteSearch extends ListActivity  implements SearchView.OnQueryTextListener {
    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            //扩展 显示时间 颜色
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // 2
            NotePad.Notes.COLUMN_NAME_BACK_COLOR
    };
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.note_search_list);
        Intent intent = getIntent();
        if (intent.getData() == null) {
            intent.setData(NotePad.Notes.CONTENT_URI);
        }
        SearchView searchview = (SearchView)findViewById(R.id.search_view);
        //为查询文本框注册监听器
        searchview.setOnQueryTextListener(NoteSearch.this);  
    }
    @Override
    public boolean onQueryTextSubmit(String query) {
        return false;
    }
    @Override
    public boolean onQueryTextChange(String newText) {
        String selection = NotePad.Notes.COLUMN_NAME_TITLE + " Like ? ";
        String[] selectionArgs = { "%"+newText+"%" };
        Cursor cursor = managedQuery(
                getIntent().getData(),            // Use the default content URI for the provider.
                PROJECTION,                       // Return the note ID and title for each note. and modifcation date
                selection,                        // 条件左边
                selectionArgs,                    // 条件右边
                NotePad.Notes.DEFAULT_SORT_ORDER  // Use the default sort order.
        );
        String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,  NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE };
        int[] viewIDs = { android.R.id.text1 , R.id.text1_time };
        MyCursorAdapter adapter = new MyCursorAdapter(
                this,
                R.layout.noteslist_item,
                cursor,
                dataColumns,
                viewIDs
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
动态搜索的实现最主要的部分在onQueryTextChange方法中，使用这个方法要先为SearchView注册监听：<br>
```
SearchView searchview = (SearchView)findViewById(R.id.search_view);
searchview.setOnQueryTextListener(NoteSearch.this);  
```
搜索要做到模糊匹配，使用数据库查询语句中的LIKE和%结合来实现：<br>
```
String[] selectionArgs = { "%"+newText+"%" };
```
最后在AndroidManifest.xml注册NoteSearch：<br>
```
<!--添加搜索activity-->
    <activity
        android:name="NoteSearch"
        android:label="@string/title_notes_search">
    </activity>
```
搜索界面：<br>
![searchui](https://github.com/wzs8253669/picture/raw/master/searchui.png)<br>
搜索结果：<br>
1：<br>
![search](https://github.com/wzs8253669/picture/raw/master/search1.jpg)<br>
2：<br>
![search](https://github.com/wzs8253669/picture/raw/master/search2.jpg)<br>
3：<br>
![search](https://github.com/wzs8253669/picture/raw/master/search3.jpg)<br>

- UI美化

把NotesList的主题换成白色，在AndroidManifest.xml中NotesList的Activity中添加：<br>
```
android:theme="@android:style/Theme.Holo.Light"
```
改变后如下图：<br>
![main](https://github.com/wzs8253669/picture/raw/master/main.png)<br>
要保存颜色的数据就要在数据库中添加一个颜色字段，在这之前在NotePad契约类中添加：<br>
```
public static final String COLUMN_NAME_BACK_COLOR = "color";
```
创建数据库表地方添加颜色的字段：<br>
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
```
把颜色定义为INTEGER的主要原因是，在系统中预定于好五种颜色，根据颜色对应不int值选择要显示的颜色，契约类中的定义：<br>
```
public static final int DEFAULT_COLOR = 0; //白
public static final int YELLOW_COLOR = 1; //黄
public static final int BLUE_COLOR = 2; //蓝
public static final int GREEN_COLOR = 3; //绿
public static final int RED_COLOR = 4; //红
```

- 背景更换

编辑笔记的Activity为NoteEditor。在PROJECTION中添加颜色项：<br>
```
  private static final String[] PROJECTION =
        new String[] {
            NotePad.Notes._ID,
            NotePad.Notes.COLUMN_NAME_TITLE,
            NotePad.Notes.COLUMN_NAME_NOTE,
            NotePad.Notes.COLUMN_NAME_BACK_COLOR
    };
```
先在菜单文件中添加一个更改背景的选项：<br>
```
<item android:id="@+id/menu_color"
        android:title="@string/menu_color"
        android:icon="@drawable/ic_menu_color"
        android:showAsAction="always"/>
```
在NoteEditor中找到onOptionsItemSelected()方法，在菜单的switch中添加：<br>
```
//换背景颜色选项
    case R.id.menu_color:
        changeColor();
        break;
```
在NoteEditor中添加函数changeColor()：<br>
```
//跳转改变颜色的activity，将uri信息传到新的activity
    private final void changeColor() {
        Intent intent = new Intent(null,mUri);
        intent.setClass(NoteEditor.this,NoteColor.class);
        NoteEditor.this.startActivity(intent);
    }
```
对选择颜色界面进行布局，新建布局note_color.xml，垂直线性布局放置5个ImageButton，并创建NoteColor的Acitvity，用来选择颜色。<br>
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
        android:background="@color/colorWhite"
        android:onClick="white"/>
    <ImageButton
        android:id="@+id/color_yellow"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/colorYellow"
        android:onClick="yellow"/>
    <ImageButton
        android:id="@+id/color_blue"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/colorBlue"
        android:onClick="blue"/>
    <ImageButton
        android:id="@+id/color_green"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/colorGreen"
        android:onClick="green"/>
    <ImageButton
        android:id="@+id/color_red"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/colorRed"
        android:onClick="red"/>
</LinearLayout>
```

```
<!--换背景色-->
<activity android:name="NoteColor"
    android:theme="@android:style/Theme.Holo.Light.Dialog"
    android:label="ChangeColor"
    android:windowSoftInputMode="stateVisible"/>
```

选择菜单上的衣服图标：<br>
![choicecolor](https://github.com/wzs8253669/picture/raw/master/choicecolor.png)<br>
选择蓝色：<br>
![background](https://github.com/wzs8253669/picture/raw/master/background.png)<br>

- 导出笔记

在菜单文件中添加一个导出笔记的选项，editor_options_menu.xml：<br>
```
<item android:id="@+id/menu_output"
    android:title="@string/menu_output" />
```
在NoteEditor中找到onOptionsItemSelected()方法，在菜单的switch中添加：<br>
```
//导出笔记选项
   case R.id.menu_output:
        outputNote();
        break;
```
在NoteEditor中添加函数outputNote()：<br>
```
//跳转导出笔记的activity，将uri信息传到新的activity
    private final void outputNote() {
        Intent intent = new Intent(null,mUri);
        intent.setClass(NoteEditor.this,OutputText.class);
        NoteEditor.this.startActivity(intent);
    }
```
对选择导出文件界面进行布局，新建布局output_text.xml，垂直线性布局放置EditText和Button，并创建OutputText的Acitvity，用来选择颜色。：<br>

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:paddingLeft="6dip"
    android:paddingRight="6dip"
    android:paddingBottom="3dip">
    <EditText android:id="@+id/output_name"
        android:maxLines="1"
        android:layout_marginTop="2dp"
        android:layout_marginBottom="15dp"
        android:layout_width="wrap_content"
        android:ems="25"
        android:layout_height="wrap_content"
        android:autoText="true"
        android:capitalize="sentences"
        android:scrollHorizontally="true" />
    <Button android:id="@+id/output_ok"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="right"
        android:text="@string/output_ok"
        android:onClick="OutputOk" />
</LinearLayout>
```
```
public class OutputText extends Activity {
   //要使用的数据库中笔记的信息
    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_NOTE, // 2
            NotePad.Notes.COLUMN_NAME_CREATE_DATE, // 3
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // 4
    };
    //读取出的值放入这些变量
    private String TITLE;
    private String NOTE;
    private String CREATE_DATE;
    private String MODIFICATION_DATE;
    //读取该笔记信息
    private Cursor mCursor;
    //导出文件的名字
    private EditText mName;
    //NoteEditor传入的uri，用于从数据库查出该笔记
    private Uri mUri;
    //关于返回与保存按钮的一个特殊标记，返回的话不执行导出，点击按钮才导出
    private boolean flag = false;
    private static final int COLUMN_INDEX_TITLE = 1;
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.output_text);
        mUri = getIntent().getData();
        mCursor = managedQuery(
                mUri,        // The URI for the note that is to be retrieved.
                PROJECTION,  // The columns to retrieve
                null,        // No selection criteria are used, so no where columns are needed.
                null,        // No where columns are used, so no where values are needed.
                null         // No sort order is needed.
        );
        mName = (EditText) findViewById(R.id.output_name);
    }
    @Override
    protected void onResume(){
        super.onResume();
        if (mCursor != null) {
            // The Cursor was just retrieved, so its index is set to one record *before* the first
            // record retrieved. This moves it to the first record.
            mCursor.moveToFirst();
            //编辑框默认的文件名为标题，可自行更改
            mName.setText(mCursor.getString(COLUMN_INDEX_TITLE));
        }
    }
    @Override
    protected void onPause() {
        super.onPause();
        if (mCursor != null) {
        //从mCursor读取对应值
            TITLE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_TITLE));
            NOTE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_NOTE));
            CREATE_DATE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_CREATE_DATE));
            MODIFICATION_DATE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE));
            //flag在点击导出按钮时会设置为true，执行写文件
            if (flag == true) {
                write();
            }
            flag = false;
        }
    }
    public void OutputOk(View v){
        flag = true;
        finish();
    }
    private void write()
    {
        try
        {
            // 如果手机插入了SD卡，而且应用程序具有访问SD的权限
            if (Environment.getExternalStorageState().equals(
                    Environment.MEDIA_MOUNTED)) {
                // 获取SD卡的目录
                File sdCardDir = Environment.getExternalStorageDirectory();
                //创建文件目录
                File targetFile = new File(sdCardDir.getCanonicalPath() + "/" + mName.getText() + ".txt");
                //写文件
                PrintWriter ps = new PrintWriter(new OutputStreamWriter(new FileOutputStream(targetFile), "UTF-8"));
                ps.println(TITLE);
                ps.println(NOTE);
                ps.println("创建时间：" + CREATE_DATE);
                ps.println("最后一次修改时间：" + MODIFICATION_DATE);
                ps.close();
                Toast.makeText(this, "保存成功,保存位置：" + sdCardDir.getCanonicalPath() + "/" + mName.getText() + ".txt", Toast.LENGTH_LONG).show();
            }
        }
        catch (Exception e)
        {
            e.printStackTrace();
        }
    }
}
```
将这个Acitvity主题定义为对话框样式，并且加入权限：<br>
```
<!--添加导出activity-->
        <activity android:name="OutputText"
            android:label="@string/output_name"
            android:theme="@android:style/Theme.Holo.Dialog"
            android:windowSoftInputMode="stateVisible">
        </activity>
```
```
 <!-- 在SD卡中创建与删除文件权限 -->
    <uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS"/>
    <!-- 向SD卡写入数据权限 -->
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
```

点进“楼五四”笔记：<br>
![note1](https://github.com/wzs8253669/picture/raw/master/note1.png)<br>
点击隐藏的菜单，选择导出笔记：<br>
 ![outputmenu](https://github.com/wzs8253669/picture/raw/master/outputmenu.png)<br>
点击确认导出：<br>
![name1](https://github.com/wzs8253669/picture/raw/master/name1.png)<br>
保存成功，弹出提示：<br>
![outputok1](https://github.com/wzs8253669/picture/raw/master/outputok1.png)<br>

- 笔记排序

在菜单文件list_options_menu.xml中添加：<br>
```
<item
    android:id="@+id/menu_sort"
    android:title="@string/menu_sort"
    android:icon="@android:drawable/ic_menu_sort_by_size"
    android:showAsAction="always" >
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
```
在NotesList菜单switch下添加case：<br>
```
//创建时间排序
    case R.id.menu_sort1:
        cursor = managedQuery(
                getIntent().getData(),            
                PROJECTION,                      
                null,                          
                null,                          
                NotePad.Notes._ID 
                );
        adapter = new MyCursorAdapter(
                this,
                R.layout.noteslist_item,
                cursor,
                dataColumns,
                viewIDs
        );
        setListAdapter(adapter);
        return true;
 //修改时间排序
    case R.id.menu_sort2:
        cursor = managedQuery(
                getIntent().getData(),          
                PROJECTION,                      
                null,                            
                null,                       
                NotePad.Notes.DEFAULT_SORT_ORDER 
        );
        adapter = new MyCursorAdapter(
                this,
                R.layout.noteslist_item,
                cursor,
                dataColumns,
                viewIDs
        );
        setListAdapter(adapter);
        return true;
    //颜色排序
    case R.id.menu_sort3:
        cursor = managedQuery(
                getIntent().getData(),
                PROJECTION,      
                null,       
                null,       
                NotePad.Notes.COLUMN_NAME_BACK_COLOR
                );
        adapter = new MyCursorAdapter(
                this,
                R.layout.noteslist_item,
                cursor,
                dataColumns,
                viewIDs
                );
        setListAdapter(adapter);
        return true;
```

- 扩展后的目录结构

![dirstructure1](https://github.com/wzs8253669/picture/raw/master/dirstructure1.png)<br>
![dirstructure2](https://github.com/wzs8253669/picture/raw/master/dirstructure2.png)<br>