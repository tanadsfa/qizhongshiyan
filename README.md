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

详细实现过程
1. NoteList界面中笔记条目增加时间戳显示
修改的文件：
res/layout/noteslist_item.xml
NotesList.java
修改步骤：
1.1在 res/layout/noteslist_item.xml 中为时间戳添加一个 TextView：

<TextView
    android:id="@+id/note_timestamp"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:textColor="#666666"
    android:textSize="12sp"
    android:gravity="end" />

1.2在 NotesList.java 的 loadNotes() 方法中，增加对时间戳字段的处理。
1.3设置 PROJECTION 包括时间戳字段。
1.4使用 SimpleCursorAdapter 的 ViewBinder 格式化时间戳。


2. 查询功能
修改的文件：
activity_notes_list.xml
NotesList.java
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
activity_notes_list.xml
NotesList.java

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
NotePadProvider.java
NotesList.java

修改步骤
4.1修改 loadNotes 方法：在查询条件中增加 LIMIT 3，当用户选择“最近打开”选项时，按修改时间倒序排列，并限制为 3 条笔记。
4.2修改 onOptionsItemSelected 方法：添加一个菜单选项“最近打开”，调用 loadNotes 方法，并设置排序条件为修改时间。
4.3修改菜单资源文件：
	在菜单文件中添加“最近打开”选项。
4.4验证功能：点击“最近打开”时，确保显示最近 3 条打开的笔记，笔记按修改时间倒序排列，保持其他功能（如新增、编辑笔记和默认排序）正常运行。

5. 使用SharedPreferences更换背景
修改的文件：
note_editor.xml
NoteEditor.java

修改步骤：
5.1在 note_editor.xml 中为背景设置默认属性。
5.2在 NoteEditor.java 中实现背景颜色存储与加载：使用 SharedPreferences 保存用户选择的颜色，在 onCreate 中加载颜色并应用。
5.3添加颜色选择对话框：使用 AlertDialog 显示颜色列表，保存用户选择的颜色到 SharedPreferences。


