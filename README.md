详细实现过程
1. NoteList界面中笔记条目增加时间戳显示

添加目的：为 Android 项目的 NotesList 界面添加时间戳功能，目的是展示每条笔记的修改时间。通过这一功能，用户能够清楚地看到每条笔记的最后更新时间，提升用户体验。

1.1 修改查询字段，增加时间戳列

在 NotesList.java 中，修改 PROJECTION 数组，添加时间戳字段 COLUMN_NAME_MODIFICATION_DATE：

```
private static final String[] PROJECTION = new String[] {
    NotePad.Notes._ID, // 0
    NotePad.Notes.COLUMN_NAME_TITLE, // 1
    NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE // 2 - 新增时间戳字段
};

1.2 修改布局文件，增加时间戳 TextView

在 res/layout/noteslist_item.xml 中，增加一个 TextView 用于显示时间戳。该 TextView 将显示在笔记标题下方，并采用较小的字体显示，以避免干扰标题的显示。

<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:padding="5dp">

    <!-- 原来的标题 TextView -->
    <TextView
        android:id="@android:id/text1"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:gravity="center_vertical"
        android:singleLine="true"
    />

    <!-- 新增的时间戳 TextView -->
    <TextView
        android:id="@+id/note_timestamp"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textSize="12sp"
        android:textColor="#808080"
        android:gravity="center_vertical"
        android:paddingTop="2dp"
    />
</LinearLayout>
```
1.3 更新适配器，绑定时间戳字段到 TextView

在 NotesList.java 文件的 onCreate() 方法中，修改 SimpleCursorAdapter，将时间戳字段映射到布局中的 TextView。

```
String[] dataColumns = {
    NotePad.Notes.COLUMN_NAME_TITLE, 
    NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE  // 新增时间戳字段
};

int[] viewIDs = {
    android.R.id.text1, 
    R.id.note_timestamp  // 绑定到布局中的时间戳 TextView
};

SimpleCursorAdapter adapter = new SimpleCursorAdapter(
    this,                     // Context
    R.layout.noteslist_item,  // 指定列表项的布局文件
    cursor,                   // 数据来源
    dataColumns,
    viewIDs
);
setListAdapter(adapter);
```

1.4 格式化时间戳为可读的日期格式
 
为了让时间戳更易于理解，需要将其转换为用户友好的日期格式。我们使用 SimpleDateFormat 来格式化时间戳，并设置时区为中国北京时间。
```
adapter.setViewBinder(new SimpleCursorAdapter.ViewBinder() {
    @Override
    public boolean setViewValue(View view, Cursor cursor, int columnIndex) {
        if (view.getId() == R.id.note_timestamp) {
            // 获取时间戳
            long timestamp = cursor.getLong(columnIndex);

            // 创建日期格式化对象，并设置为中国北京时间（Asia/Shanghai 时区）
            SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm");
            dateFormat.setTimeZone(TimeZone.getTimeZone("Asia/Shanghai"));

            // 将时间戳转换为北京时间
            String date = dateFormat.format(new Date(timestamp));
            ((TextView) view).setText(date);
            return true;  // 我们处理了这个绑定
        }
        return false;  // 其他字段交由默认绑定处理
    }
});
```

1.5 更新数据库插入和更新逻辑

在笔记插入和更新时，需要确保时间戳字段被正确设置。在 NotePadProvider.java 中的 insert 和 update 方法中加入逻辑，以确保每次插入或更新时，COLUMN_NAME_MODIFICATION_DATE 字段会被更新为当前时间。

```
// 在 NotePadProvider.java 中的 insert 方法中
if (!values.containsKey(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE)) {
    values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, System.currentTimeMillis());
}

// 在 update 方法中，将修改日期更新为当前时间
values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, System.currentTimeMillis());
```

1.6 结果：

通过以上步骤，成功地在 NotesList 界面中添加了时间戳显示功能。每个笔记项下方会显示其最后修改时间，并且时间格式为“年-月-日 时:分”，显示时区为中国北京时间（Asia/Shanghai）。用户可以清晰地看到每条笔记的修改时间。

2. 查询功能

实验目的：为了实现查询功能，我们使用了 SearchView 组件来允许用户输入关键字，进而查询数据库中的笔记。查询的条件是笔记的标题和内容包含用户输入的关键字。通过使用 SimpleCursorAdapter 适配器和 Cursor，可以实现动态查询并更新界面。

2.1 添加 SearchView 组件

直接在代码中动态添加 SearchView

在 NotesList.java 的 onCreate 方法中动态创建 SearchView，并将其添加到 ListView 的头部。

```
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);

    // 获取 ListView
    ListView listView = getListView();

    // 创建 SearchView
    SearchView searchView = new SearchView(this);
    searchView.setLayoutParams(new ListView.LayoutParams(
            ListView.LayoutParams.MATCH_PARENT,
            ListView.LayoutParams.WRAP_CONTENT
    ));
    searchView.setQueryHint("搜索笔记标题或内容");

    // 将 SearchView 添加到 ListView 的 Header
    listView.addHeaderView(searchView);

    // 设置查询监听器
    searchView.setOnQueryTextListener(new SearchView.OnQueryTextListener() {
        @Override
        public boolean onQueryTextSubmit(String query) {
            searchNotes(query);  // 执行查询
            return true;
        }

        @Override
        public boolean onQueryTextChange(String newText) {
            searchNotes(newText);  // 动态查询
            return true;
        }
    });

    // 原有的查询和适配器设置代码
    ...
}
```

2.2 实现 searchNotes 方法

在 NotesList.java 中添加 searchNotes 方法，用于根据用户输入的关键字查询笔记标题和内容。使用 ContentResolver.query() 方法查询数据库，并动态更新列表。

```
private void searchNotes(String query) {
    // 定义查询条件，搜索标题或内容中包含关键字的笔记
    String selection = NotePad.Notes.COLUMN_NAME_TITLE + " LIKE ? OR " +
                       NotePad.Notes.COLUMN_NAME_NOTE + " LIKE ?";
    String[] selectionArgs = new String[]{"%" + query + "%", "%" + query + "%"};

    // 执行查询
    Cursor cursor = getContentResolver().query(
            NotePad.Notes.CONTENT_URI,       // 查询 URI
            PROJECTION,                      // 返回的列
            selection,                       // 查询条件
            selectionArgs,                   // 查询参数
            NotePad.Notes.DEFAULT_SORT_ORDER // 排序
    );

    // 更新适配器的数据
    ((SimpleCursorAdapter) getListAdapter()).changeCursor(cursor);
}
```

2.3 确保 PROJECTION 常量包含查询所需的字段

PROJECTION 常量中应包含笔记的标题 (COLUMN_NAME_TITLE) 和内容 (COLUMN_NAME_NOTE) 字段，以便能够查询和显示笔记的标题和内容。

```
private static final String[] PROJECTION = new String[] {
    NotePad.Notes._ID,
    NotePad.Notes.COLUMN_NAME_TITLE,
    NotePad.Notes.COLUMN_NAME_NOTE,
    NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE // 如果需要显示时间戳
};
```

2.4 管理 Cursor 的生命周期

确保在 onDestroy 方法中关闭旧的 Cursor，以避免内存泄漏。

```
@Override
protected void onDestroy() {
    super.onDestroy();
    Cursor cursor = ((SimpleCursorAdapter) getListAdapter()).getCursor();
    if (cursor != null && !cursor.isClosed()) {
        cursor.close();
    }
}
```
2.5结果

通过实现上述功能，成功地在 NotesList 界面上添加了笔记查询功能。用户可以在 SearchView 中输入关键字，实时查询笔记的标题或内容，并动态更新列表显示。当用户输入时，系统会根据查询条件在数据库中查找匹配的笔记，并在界面上显示结果。

3. 笔记按内容多少分类

添加目的：允许用户通过点击一个按钮切换笔记列表的排序方式，支持按笔记内容的大小或修改时间进行排序。用户可以动态地查看不同排序方式下的笔记内容，并通过一个排序按钮切换排序方式。

3.1 布局文件修改

为了支持排序功能，我们需要在布局文件中添加一个按钮用于切换排序方式，以及一个 SearchView 搜索框用于过滤笔记。

```
<!-- activity_notes_list.xml -->
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <!-- 搜索框 -->
    <SearchView
        android:id="@+id/search_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:queryHint="搜索笔记" />

    <!-- 排序按钮 -->
    <Button
        android:id="@+id/button_sort_by_size"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="按内容大小排序" />

    <!-- 笔记列表 -->
    <ListView
        android:id="@android:id/list"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

</LinearLayout>
```

3.2 按钮功能实现

按钮 button_sort_by_size 负责切换排序方式。点击该按钮时，调用 toggleSortOrder() 方法来切换排序方式，并更新按钮文本。

```
sortBySizeButton = findViewById(R.id.button_sort_by_size);
sortBySizeButton.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        toggleSortOrder(); // 切换排序方式
    }
});
```

3.3 排序切换逻辑

toggleSortOrder() 方法根据当前排序状态切换排序方式，更新按钮文本，并调用 loadNotes() 方法重新加载数据：

```
private void toggleSortOrder() {
    if (isSortedByContentSize) {
        loadNotes(NotePad.Notes.DEFAULT_SORT_ORDER); // 默认按时间排序
        sortBySizeButton.setText("按内容大小排序"); // 更新按钮文本
    } else {
        loadNotes("LENGTH(" + NotePad.Notes.COLUMN_NAME_NOTE + ") ASC"); // 按内容大小排序
        sortBySizeButton.setText("按时间排序"); // 更新按钮文本
    }
    isSortedByContentSize = !isSortedByContentSize; // 切换排序状态
}
```

3.4 数据加载与排序

loadNotes() 方法负责加载笔记数据，根据传入的 sortOrder 参数（按时间或按内容大小）查询笔记数据，并将查询结果显示在列表中：

```
private void loadNotes(String sortOrder) {

    // 获取启动此 Activity 的 Intent
    Intent intent = getIntent();

    // 如果 Intent 没有携带数据，则将其数据设置为默认 URI（访问笔记列表）
    if (intent.getData() == null) {
        intent.setData(NotePad.Notes.CONTENT_URI);
    }

    // 执行查询，并使用传入的 sortOrder 参数作为排序字段
    Cursor cursor = managedQuery(
            intent.getData(),         // 使用默认内容提供器 URI
            PROJECTION,               // 返回每条笔记的 ID 和标题
            null,                     // 没有 where 子句，返回所有记录
            null,                     // 没有 where 子句，因此没有列值
            sortOrder                 // 使用传入的排序字段（按时间或内容大小排序）
    );

    // 数据列映射
    String[] dataColumns = {
            NotePad.Notes.COLUMN_NAME_TITLE,             // 显示笔记标题
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE  // 显示笔记修改时间
    };

    // 视图 ID
    int[] viewIDs = {
            android.R.id.text1,                          // 显示笔记标题
            R.id.note_timestamp                          // 显示笔记时间戳
    };

    // 创建 SimpleCursorAdapter 来绑定数据和视图
    SimpleCursorAdapter adapter = new SimpleCursorAdapter(
            this,                     // Context
            R.layout.noteslist_item,  // 指定列表项的布局文件
            cursor,                   // 数据来源
            dataColumns,
            viewIDs
    );

    // 设置 ViewBinder 来格式化时间
    adapter.setViewBinder(new SimpleCursorAdapter.ViewBinder() {
        @Override
        public boolean setViewValue(View view, Cursor cursor, int columnIndex) {
            if (view.getId() == R.id.note_timestamp) {
                // 获取时间戳
                long timestamp = cursor.getLong(columnIndex);

                // 创建日期格式化对象，并设置为中国北京时间（Asia/Shanghai 时区）
                SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm");
                dateFormat.setTimeZone(TimeZone.getTimeZone("Asia/Shanghai"));

                // 将时间戳转换为北京时间
                String date = dateFormat.format(new Date(timestamp));
                ((TextView) view).setText(date);
                return true;  // 我们处理了这个绑定
            }
            return false;  // 其他字段交由默认绑定处理
        }
    });

    setListAdapter(adapter);  // 将适配器绑定到列表视图
}
```

3.5 列表项布局

noteslist_item.xml 文件负责定义每个笔记列表项的布局，其中包含了笔记标题和时间戳（显示笔记的最后修改时间）：

```
<!-- noteslist_item.xml -->
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical">

    <TextView
        android:id="@android:id/text1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="18sp"
        android:textColor="#000000" />

    <!-- 时间戳 -->
    <TextView
        android:id="@+id/note_timestamp"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="12sp"
        android:textColor="#888888" />
</LinearLayout>
```

3.6 总结：

通过此功能，用户能够轻松切换排序方式并查看按内容大小或修改时间排列的笔记列表。该功能通过 SimpleCursorAdapter 和 ViewBinder 的结合，确保了数据与视图的有效绑定，并通过 Cursor 进行动态查询和显示。

4. 最近打开功能

允许用户查看最近访问过的笔记，并通过主菜单进行访问。用户点击笔记时，会更新该笔记的最近访问时间；点击“最近打开”菜单时，显示最近访问过的 3 个笔记。

4.1 修改 NotePad 类
无需改动 NotePad 类，因为它已经包含了 modified 字段（COLUMN_NAME_MODIFICATION_DATE），该字段用于记录最近打开的时间。

4.2 修改 NotesList 类

4.2.1 更新笔记的 modified 字段

在 onListItemClick() 方法中，每次用户打开笔记时，更新该笔记的 modified 字段为当前的时间戳。

```
@Override
protected void onListItemClick(ListView l, View v, int position, long id) {
    // 构建笔记的 URI
    Uri uri = ContentUris.withAppendedId(getIntent().getData(), id);

    // 更新最近打开时间戳
    ContentValues values = new ContentValues();
    values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, System.currentTimeMillis());
    getContentResolver().update(uri, values, null, null);

    // 打开笔记编辑界面
    startActivity(new Intent(Intent.ACTION_EDIT, uri));
}
```

4.2.2 在主菜单中添加“最近打开”选项

在 onOptionsItemSelected() 方法中处理“最近打开”选项，当点击时调用 loadNotes() 方法加载并显示最近打开的笔记，排序按 modified 字段倒序。

```
@Override
public boolean onOptionsItemSelected(MenuItem item) {
    switch (item.getItemId()) {
        case R.id.menu_add:
            // 新建笔记
            startActivity(new Intent(Intent.ACTION_INSERT, getIntent().getData()));
            return true;

        case R.id.menu_recently_opened:
            // 加载最近打开的笔记列表（限制为 3 个）
            loadNotes(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE);
            Toast.makeText(this, "显示最近打开的 3 条笔记", Toast.LENGTH_SHORT).show();
            return true;

        default:
            return super.onOptionsItemSelected(item);
    }
}
```

4.2.3 修改 loadNotes() 方法

loadNotes() 方法需要根据传入的排序字段加载笔记，并对“最近打开的笔记”进行数量限制。通过在查询中添加 LIMIT 3，确保返回最多 3 条笔记。

```
private void loadNotes(String sortOrder) {
    Intent intent = getIntent();

    // 如果 Intent 没有携带数据，则设置为默认 URI（访问笔记列表）
    if (intent.getData() == null) {
        intent.setData(NotePad.Notes.CONTENT_URI);
    }

    // 如果是最近打开的笔记，则限制数量为 3
    if (NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE.equals(sortOrder)) {
        sortOrder = sortOrder + " DESC LIMIT 3";  // 限制为最近的 3 个笔记
    }

    // 查询数据库，加载笔记
    Cursor cursor = managedQuery(
            intent.getData(),         // 默认内容提供器 URI
            PROJECTION,               // 返回每条笔记的 ID 和标题
            null,                     // 没有 where 子句，返回所有记录
            null,                     // 没有 where 子句，因此没有列值
            sortOrder                 // 使用传入的排序字段
    );

    // 映射数据到视图
    String[] dataColumns = {
            NotePad.Notes.COLUMN_NAME_TITLE,
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE
    };
    int[] viewIDs = {
            android.R.id.text1,
            R.id.note_timestamp
    };

    SimpleCursorAdapter adapter = new SimpleCursorAdapter(
            this,                     // Context
            R.layout.noteslist_item,  // 列表项布局文件
            cursor,                   // 数据来源
            dataColumns,
            viewIDs
    );

    // 自定义 ViewBinder 显示格式化后的时间戳
    adapter.setViewBinder(new SimpleCursorAdapter.ViewBinder() {
        @Override
        public boolean setViewValue(View view, Cursor cursor, int columnIndex) {
            if (view.getId() == R.id.note_timestamp) {
                long timestamp = cursor.getLong(columnIndex);

                SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm");
                dateFormat.setTimeZone(TimeZone.getTimeZone("Asia/Shanghai"));
                String date = dateFormat.format(new Date(timestamp));
                ((TextView) view).setText(date);
                return true;
            }
            return false;
        }
    });

    setListAdapter(adapter);
}
```

4.3.3 修改菜单 XML

在 res/menu/list_options_menu.xml 文件中，添加“最近打开”菜单项，供用户点击查看最近访问的笔记。

```
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:id="@+id/menu_add"
        android:title="新建笔记"
        android:showAsAction="ifRoom" />

    <item
        android:id="@+id/menu_recently_opened"
        android:title="最近打开"
        android:showAsAction="never" />
</menu>
```

4.3.4 修改布局文件

在 res/layout/noteslist_item.xml 中，确保笔记列表项布局中有显示时间戳的 TextView，用于显示每个笔记的最后修改时间。

```
<TextView
    android:id="@+id/note_timestamp"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:textSize="12sp"
    android:textColor="#808080"
    android:paddingTop="4dp" />
```

5 使用SharedPreferences更换背景


4.5.1 修改 note_editor.xml

在 note_editor.xml 中，确保背景颜色可以动态更改，将默认背景设置为透明，以便动态应用颜色。

```
<?xml version="1.0" encoding="utf-8"?>
<view xmlns:android="http://schemas.android.com/apk/res/android"
    class="com.example.android.notepad.NoteEditor$LinedEditText"
    android:id="@+id/note"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@android:color/transparent"
    android:padding="5dp"
    android:scrollbars="vertical"
    android:fadingEdge="vertical"
    android:gravity="top"
    android:textSize="22sp"
    android:capitalize="sentences" />
```
5.2 修改 NoteEditor 类

5.2.1 添加 SharedPreferences 支持

在 NoteEditor 类中添加 SharedPreferences 的初始化和动态背景颜色加载逻辑。

```
private SharedPreferences sharedPreferences;
private static final String PREF_NAME = "NotePadPreferences"; // 存储偏好设置名称
```

5.2.2 修改 onCreate 方法

在 onCreate 方法中，添加加载背景颜色的逻辑。

```
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);

    // 初始化 SharedPreferences
    sharedPreferences = getSharedPreferences(PREF_NAME, MODE_PRIVATE);

    final Intent intent = getIntent();
    final String action = intent.getAction();

    if (Intent.ACTION_EDIT.equals(action)) {
        mState = STATE_EDIT;
        mUri = intent.getData();
    } else if (Intent.ACTION_INSERT.equals(action) || Intent.ACTION_PASTE.equals(action)) {
        mState = STATE_INSERT;
        mUri = getContentResolver().insert(intent.getData(), null);
        if (mUri == null) {
            Log.e(TAG, "Failed to insert new note into " + getIntent().getData());
            finish();
            return;
        }
        setResult(RESULT_OK, (new Intent()).setAction(mUri.toString()));
    } else {
        Log.e(TAG, "Unknown action, exiting");
        finish();
        return;
    }

    mCursor = managedQuery(mUri, PROJECTION, null, null, null);

    if (Intent.ACTION_PASTE.equals(action)) {
        performPaste();
        mState = STATE_EDIT;
    }

    setContentView(R.layout.note_editor);
    mText = (EditText) findViewById(R.id.note);

    // **加载背景颜色**
    loadBackgroundColor();

    if (savedInstanceState != null) {
        mOriginalContent = savedInstanceState.getString(ORIGINAL_CONTENT);
    }
}
```

5.2.3 添加 loadBackgroundColor 方法

从 SharedPreferences 中加载颜色，并将背景颜色应用到编辑框。

```
private void loadBackgroundColor() {
    // 从 SharedPreferences 加载颜色，如果没有设置则默认为白色
    String color = sharedPreferences.getString("note_background_color", "#FFFFFF");
    mText.setBackgroundColor(Color.parseColor(color));
}
```

5.3 添加颜色选择功能

5.3.1 修改菜单资源文件

在 res/menu/editor_options_menu.xml 中添加一个菜单项，用于触发颜色选择功能。

```
<item
    android:id="@+id/menu_set_background_color"
    android:title="选择背景颜色"
    android:showAsAction="never" />
```

5.3.2 修改 onOptionsItemSelected 方法

在 NoteEditor 类中，添加对新菜单项的处理：

```
@Override
public boolean onOptionsItemSelected(MenuItem item) {
    switch (item.getItemId()) {
        case R.id.menu_set_background_color:
            showColorPickerDialog();
            return true;
        default:
            return super.onOptionsItemSelected(item);
    }
}
```

5.3.3 添加 showColorPickerDialog 方法

实现一个简单的颜色选择对话框，并将用户选择的颜色存储到 SharedPreferences。

```
private void showColorPickerDialog() {
    // 颜色代码和中文描述
    final String[] colors = {"#FFFFFF", "#FFEBEE", "#E3F2FD", "#E8F5E9", "#FFFDE7", "#F3E5F5"};
    final String[] colorNames = {"白色", "浅红色", "浅蓝色", "浅绿色", "浅黄色", "浅紫色"};

    AlertDialog.Builder builder = new AlertDialog.Builder(this);
    builder.setTitle("选择背景颜色");

    builder.setItems(colorNames, new DialogInterface.OnClickListener() {
        @Override
        public void onClick(DialogInterface dialog, int which) {
            String selectedColor = colors[which]; // 根据选择的名称获取对应颜色代码
            saveBackgroundColor(selectedColor);
            loadBackgroundColor(); // 更新背景颜色
        }
    });
    builder.create().show();
}
```

5.3.4 添加 saveBackgroundColor 方法
将选择的颜色保存到 SharedPreferences。

```
private void saveBackgroundColor(String color) {
    SharedPreferences.Editor editor = sharedPreferences.edit();
    editor.putString("note_background_color", color);
    editor.apply();
}
```

1.NoteList界面中笔记条目增加时间戳显示:

![image](https://github.com/tanadsfa/qizhongshiyan/blob/main/image/02.png)

2.查询功能：

![image](https://github.com/tanadsfa/qizhongshiyan/blob/main/image/03.png)

![image](https://github.com/tanadsfa/qizhongshiyan/blob/main/image/04.png)

3.笔记按内容多少分类（少的在上面）：

默认为按时间排序：

![image](https://github.com/tanadsfa/qizhongshiyan/blob/main/image/05.png)

点击"按内容大小排序"

![image](https://github.com/tanadsfa/qizhongshiyan/blob/main/image/05.png)



4.最近打开（打开最近的三个笔记）：

![image](https://github.com/tanadsfa/qizhongshiyan/blob/main/image/06.png)

![image](https://github.com/tanadsfa/qizhongshiyan/blob/main/image/07.png)

5.使用SharedPreferences更换背景：

![image](https://github.com/tanadsfa/qizhongshiyan/blob/main/image/08.png)

![image](https://github.com/tanadsfa/qizhongshiyan/blob/main/image/09.png)

改变后（浅紫色）：

![image](https://github.com/tanadsfa/qizhongshiyan/blob/main/image/10.png)




