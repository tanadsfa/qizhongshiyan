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

2. 修改布局文件，增加时间戳 TextView

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

3. 更新适配器，绑定时间戳字段到 TextView

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

4. 格式化时间戳为可读的日期格式
5. 
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

5. 更新数据库插入和更新逻辑

在笔记插入和更新时，需要确保时间戳字段被正确设置。在 NotePadProvider.java 中的 insert 和 update 方法中加入逻辑，以确保每次插入或更新时，COLUMN_NAME_MODIFICATION_DATE 字段会被更新为当前时间。

// 在 NotePadProvider.java 中的 insert 方法中
if (!values.containsKey(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE)) {
    values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, System.currentTimeMillis());
}

// 在 update 方法中，将修改日期更新为当前时间
values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, System.currentTimeMillis());

2. 查询功能

修改的文件：

activity_notes_list.xml、NotesList.java

修改步骤：

2.1在 activity_notes_list.xml 中添加 SearchView：

<SearchView
    android:id="@+id/search_view"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:queryHint="搜索笔记" />

2.2在 NotesList.java 中，设置查询逻辑：在 onCreate 中为 SearchView 添加监听器，实现 searchNotes() 方法，根据查询条件动态更新 Cursor。

3. 笔记按内容多少分类

修改的文件：

activity_notes_list.xml、NotesList.java

修改步骤：

3.1在布局中增加一个排序按钮：

<Button
    android:id="@+id/button_sort_by_size"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="按内容大小排序" />

3.2在 NotesList.java 中实现排序逻辑：添加一个 toggleSortOrder() 方法，用于切换排序方式，在 loadNotes() 中根据 sortOrder 参数动态加载排序数据。

4. 最近打开功能

修改的文件：

NotePadProvider.java 、NotesList.java

修改步骤：

4.1修改 loadNotes 方法：在查询条件中增加 LIMIT 3，当用户选择“最近打开”选项时，按修改时间倒序排列，并限制为 3 条笔记。
4.2修改 onOptionsItemSelected 方法：添加一个菜单选项“最近打开”，调用 loadNotes 方法，并设置排序条件为修改时间。
4.3修改菜单资源文件：
	在菜单文件中添加“最近打开”选项。
4.4验证功能：点击“最近打开”时，确保显示最近 3 条打开的笔记，笔记按修改时间倒序排列，保持其他功能（如新增、编辑笔记和默认排序）正常运行。

5. 使用SharedPreferences更换背景

修改的文件：

note_editor.xml、NoteEditor.java

修改步骤：

5.1在 note_editor.xml 中为背景设置默认属性。
5.2在 NoteEditor.java 中实现背景颜色存储与加载：使用 SharedPreferences 保存用户选择的颜色，在 onCreate 中加载颜色并应用。
5.3添加颜色选择对话框：使用 AlertDialog 显示颜色列表，保存用户选择的颜色到 SharedPreferences。



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




