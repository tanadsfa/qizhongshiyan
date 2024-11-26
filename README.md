详细实现过程
1. NoteList界面中笔记条目增加时间戳显示

添加目的：为 Android 项目的 NotesList 界面添加时间戳功能，目的是展示每条笔记的修改时间。通过这一功能，用户能够清楚地看到每条笔记的最后更新时间，提升用户体验。

1.1 修改查询字段，增加时间戳列

在 NotesList.java 中，修改 PROJECTION 数组，添加时间戳字段 COLUMN_NAME_MODIFICATION_DATE：


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

1.3 更新适配器，绑定时间戳字段到 TextView

在 NotesList.java 文件的 onCreate() 方法中，修改 SimpleCursorAdapter，将时间戳字段映射到布局中的 TextView。


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

1.4 格式化时间戳为可读的日期格式
 
为了让时间戳更易于理解，需要将其转换为用户友好的日期格式。我们使用 SimpleDateFormat 来格式化时间戳，并设置时区为中国北京时间。

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

1.5 更新数据库插入和更新逻辑

在笔记插入和更新时，需要确保时间戳字段被正确设置。在 NotePadProvider.java 中的 insert 和 update 方法中加入逻辑，以确保每次插入或更新时，COLUMN_NAME_MODIFICATION_DATE 字段会被更新为当前时间。

// 在 NotePadProvider.java 中的 insert 方法中
if (!values.containsKey(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE)) {
    values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, System.currentTimeMillis());
}

// 在 update 方法中，将修改日期更新为当前时间
values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, System.currentTimeMillis());

1.6 结果：

通过以上步骤，成功地在 NotesList 界面中添加了时间戳显示功能。每个笔记项下方会显示其最后修改时间，并且时间格式为“年-月-日 时:分”，显示时区为中国北京时间（Asia/Shanghai）。用户可以清晰地看到每条笔记的修改时间。

2. 查询功能

实验目的：为了实现查询功能，我们使用了 SearchView 组件来允许用户输入关键字，进而查询数据库中的笔记。查询的条件是笔记的标题和内容包含用户输入的关键字。通过使用 SimpleCursorAdapter 适配器和 Cursor，可以实现动态查询并更新界面。

2.1 添加 SearchView 组件

直接在代码中动态添加 SearchView

在 NotesList.java 的 onCreate 方法中动态创建 SearchView，并将其添加到 ListView 的头部。

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


2.2 实现 searchNotes 方法

在 NotesList.java 中添加 searchNotes 方法，用于根据用户输入的关键字查询笔记标题和内容。使用 ContentResolver.query() 方法查询数据库，并动态更新列表。

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

2.3 确保 PROJECTION 常量包含查询所需的字段

PROJECTION 常量中应包含笔记的标题 (COLUMN_NAME_TITLE) 和内容 (COLUMN_NAME_NOTE) 字段，以便能够查询和显示笔记的标题和内容。

private static final String[] PROJECTION = new String[] {
    NotePad.Notes._ID,
    NotePad.Notes.COLUMN_NAME_TITLE,
    NotePad.Notes.COLUMN_NAME_NOTE,
    NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE // 如果需要显示时间戳
};

2.4 管理 Cursor 的生命周期

确保在 onDestroy 方法中关闭旧的 Cursor，以避免内存泄漏。

@Override
protected void onDestroy() {
    super.onDestroy();
    Cursor cursor = ((SimpleCursorAdapter) getListAdapter()).getCursor();
    if (cursor != null && !cursor.isClosed()) {
        cursor.close();
    }
}
2.5结果

通过实现上述功能，成功地在 NotesList 界面上添加了笔记查询功能。用户可以在 SearchView 中输入关键字，实时查询笔记的标题或内容，并动态更新列表显示。当用户输入时，系统会根据查询条件在数据库中查找匹配的笔记，并在界面上显示结果。

3. 笔记按内容多少分类


4. 最近打开功能


5. 使用SharedPreferences更换背景





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




