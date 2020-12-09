# 深入理解--Android Loader

开发 [漫品](http://aimanpin.com/) 客户端 本地图书导入页面 的过程中，需要获取到手机目录中所有的txt文件进行展示用于提供给的用户进行
如果使用Java读取目录,目前想到的是递归的方式进行文件获取，但获取过程其实是比较缓慢的,基于获取到的文件再向下进行传递。如果手机文件较多，内容较多的话，这并不是一个好的选中，
也许查找时间会非常的长。
```Java
private void searchBook(ObservableEmitter<File> e, File parentFile) {
    if (null != parentFile && parentFile.listFiles() != null &&parentFile.listFiles().length > 0) {
        File[] childFiles = parentFile.listFiles();
        for (int i = 0; i < childFiles.length; i++) {
            if (childFiles[i].isFile() && childFiles[i].getName().substring(childFiles[i].getName().lastIndexOf(".") + 1).equalsIgnoreCase("txt") && childFiles[i].length() > 100 * 1024) {   //100kb
                e.onNext(childFiles[i]);
                continue;
            }
            if (childFiles[i].isDirectory() && childFiles[i].listFiles().length > 0) {
                searchBook(e, childFiles[i]);
            }
        }
    }
}

```

## 一. Loader是什么，有什么用。
Loader 顾名思义 就是 加载器。

借助 Loader API，您可以从内容提供程序或其他数据源中加载数据，以便在 FragmentActivity 或 Fragment 中显示。如果您不理解为何需要 Loader API 来执行这个看似无关紧要的操作，请首先考虑没有加载器时可能会遇到的一些问题：

 - 如果直接在 Activity 或片段中获取数据，由于通过界面线程执行查询的速度可能较慢，响应能力的不足将影响您的用户。
 - 如果从另一个线程获取数据（方法可能是使用 AsyncTask），则您需负责通过各种 Activity或片段生命周期事件（例如 onDestroy() 和配置变更）来管理线程和界面线程。

加载器不仅能解决这些问题，同时还具备其他优势。例如：

 - 加载器在单独的线程上运行，以免界面出现卡顿或无响应问题。
 - 加载器可在事件发生时提供回调方法，从而简化线程管理。
 - 加载器会保留和缓存配置变更后的结果，以免出现重复查询问题。
 - 加载器可实现观察器，从而监控基础数据源的变化。例如，CursorLoader 会自动注册 ContentObserver，以在数据变更时触发重新加载。

上面是官方的介绍，其实总结下就是以下两点：

- 1）在单独的线程中读取数据，不会阻塞UI线程

- 2）监视数据的更新

## 二. Loader API 总结

在应用中使用加载器时，可能会涉及到多个类和接口。下表对其进行了总结：

---

- LoaderManager

一种与 FragmentActivity 或 Fragment 相关联的抽象类，用于管理一个或多个 Loader 实例。每个 Activity 或片段只有一个 LoaderManager，但 LoaderManager 可管理多个加载器。

如要获取 LoaderManager，请从 Activity 或片段调用 getSupportLoaderManager()。

如要从加载器开始加载数据，请调用 initLoader() 或 restartLoader()。系统会自动确定是否已存在拥有相同整型 ID 的加载器，并将创建新加载器或重复使用现有的加载器。

---

- LoaderManager.LoaderCallbacks

此接口包含加载器事件发生时所调用的回调方法。接口定义三种回调方法：

- onCreateLoader(int, Bundle) - 系统需要创建新加载器时调用。您的代码应创建 Loader 对象并将其返回系统。

- onLoadFinished(Loader<D>, D) - 加载器在完成数据加载时调用。一般来说，您的代码应向用户显示数据。

- onLoaderReset(Loader<D>) - 重置之前创建的加载器时调用（当您调用 destroyLoader(int) 时），或由于系统销毁 Activity 或片段而使其数据不可用时调用。您的代码应删除其对加载器数据的任何引用。

此接口一般由您的 Activity 或片段实现，并在您调用 initLoader() 或 restartLoader() 时进行注册。

---

- Loader

Loader 类执行数据的加载。此类属于抽象类，并且是所有加载器的基类。您可以直接创建 Loader 的子类，或使用以下某个内置子类来简化实现：

- AsyncTaskLoader - 抽象加载器，可通过提供 AsyncTask 在单独的线程上执行加载操作。

- CursorLoader - AsyncTaskLoader 的具体子类，用于异步加载 ContentProvider 的数据。该类会查询 ContentResolver 并返回 Cursor。

---

## 三. 如何使用Loader

使用loader的几个必备条件如下:

1. 一个Activity 或者 一个Fragment。LoaderManager获取需要传递Owner，这里必须是Activity 或者fragment

2. 获取一个LoaderManager的实例。
```Java
LoaderManager.getInstance(activity).initLoader(LoaderCreator.ALL_BOOK_FILE, null, new MediaLoaderCallbacks(activity, resultCallback));
```

```java
/**
 * Gets a LoaderManager associated with the given owner, such as a {@link androidx.fragment.app.FragmentActivity} or
 * {@link androidx.fragment.app.Fragment}.
 *
 * @param owner The owner that should be used to create the returned LoaderManager
 * @param <T> A class that maintains its own {@link android.arch.lifecycle.Lifecycle} and
 *           {@link android.arch.lifecycle.ViewModelStore}. For instance,
 *           {@link androidx.fragment.app.FragmentActivity} or {@link androidx.fragment.app.Fragment}.
 * @return A valid LoaderManager
 */
@NonNull
public static <T extends LifecycleOwner & ViewModelStoreOwner> LoaderManager getInstance(
        @NonNull T owner) {
    return new LoaderManagerImpl(owner, owner.getViewModelStore());
}

```

3. 一个CursorLoader，从一个ContentProvider里加载数据。

4. 一个LoaderManager.LoaderCallbacks的实现。在这你创建新的loader，和管理已经存在的loaders。

LoaderManager.LoaderCallbacks<D>接口是LoaderManager用来向客户返回数据的方式。

每个Loader都有自己的回调对象供与LoaderManager进行交互。

该回调对象在实现LoaderManager中地位很高，告诉LoaderManager如何实例化Loader(onCreateLoader)，以及当载入行为结束或者重启（onLoadFinished或者onLoadReset）之后执行什么操作。

大多数情况，你需要把该接口实现为组件的一部分，比如说让你的Activity或者Fragment实现LoadManager.LoaderCallbacks<D>接口。


```Java

public interface LoaderCallbacks<D> {
    @MainThread
    @NonNull
    Loader<D> onCreateLoader(int id, @Nullable Bundle args);


    @MainThread
    void onLoadFinished(@NonNull Loader<D> loader, D data);

    @MainThread
    void onLoaderReset(@NonNull Loader<D> loader);
}


```

一旦实现该接口，客户端将回调对象（本例中为“this”）作为LoaderManager的initLoader函数的第三个参数传输。
总的来说，实现回调接口非常直接明了。每个回调方法都有各自明确的与LoaderManager进行交互的目的：

  - 1. onCreateLoader是一个工厂方法，用来返回一个新的Loader。LoaderManager将会在它第一次创建Loader的时候调用该方法。

  - 2. onLoadFinished方法将在Loader创建完毕的时候自动调用。典型用法是，当载入数据完毕，客户端（译者注：调用它的Activity之类的）需要更新应用UI。客户端假设每次有新数据的时候，新数据都会返回到这个方法中。记住，检测数据源是Loader的工作，Loader也会执行实际的同步载入操作。一旦Loader载入数据完成，LoaderManager将会接受到这些载入数据，并且将将结果传给回调对象的onLoadFinished方法，这样客户端（比如Activity或者Fragment）就能使用该数据了。

  - 3. 最后，当Loader们的数据被重置的时候将会调用onLoadReset。该方法让你可以从就的数据中移除不再有用的数据。


5. （可选）一种数据源，例如一个Conterprovider（当使用CursorLoader）。

6. （可选）一种显示loader加载数据的方式，例如SimpleCursorAdapter。


## 四. 获取媒体库中所有的书籍文件（手机中所有的.txt文件）
源码地址: https://github.com/AnyMarvel/ManPinAPP

路径 app/src/main/java/com/mp/android/apps/monke/monkeybook/utils/media

```java
package com.mp.android.apps.monke.monkeybook.utils.media;

import android.content.Context;
import android.database.Cursor;
import android.os.Bundle;

import androidx.fragment.app.FragmentActivity;
import androidx.loader.app.LoaderManager;
import androidx.loader.content.Loader;

import java.io.File;
import java.lang.ref.WeakReference;
import java.util.List;

/**
 * 获取媒体库的数据。
 */

public class MediaStoreHelper {

    /**
     * 获取媒体库中所有的书籍文件
     * <p>
     * 暂时只支持 TXT
     *
     * @param activity
     * @param resultCallback
     */
    public static void getAllBookFile(FragmentActivity activity, MediaResultCallback resultCallback) {
        // 将文件的获取处理交给 LoaderManager。
        LoaderManager.getInstance(activity)
                .initLoader(LoaderCreator.ALL_BOOK_FILE, null, new MediaLoaderCallbacks(activity, resultCallback));
    }

    public interface MediaResultCallback {
        void onResultCallback(List<File> files);
    }

    /**
     * Loader 回调处理
     */
    static class MediaLoaderCallbacks implements LoaderManager.LoaderCallbacks<Cursor> {
        protected WeakReference<Context> mContext;
        protected MediaResultCallback mResultCallback;

        public MediaLoaderCallbacks(Context context, MediaResultCallback resultCallback) {
            mContext = new WeakReference<>(context);
            mResultCallback = resultCallback;
        }

        @Override
        public Loader<Cursor> onCreateLoader(int id, Bundle args) {
            return LoaderCreator.create(mContext.get(), id, args);
        }

        @Override
        public void onLoadFinished(Loader<Cursor> loader, Cursor data) {
            LocalFileLoader localFileLoader = (LocalFileLoader) loader;
            localFileLoader.parseData(data, mResultCallback);
        }

        @Override
        public void onLoaderReset(Loader<Cursor> loader) {
        }
    }
}

```

LoaderCreator.java

```java
package com.mp.android.apps.monke.monkeybook.utils.media;

import android.content.Context;
import android.os.Bundle;

import androidx.loader.content.CursorLoader;


public class LoaderCreator {
    public static final int ALL_BOOK_FILE = 1;

    public static CursorLoader create(Context context, int id, Bundle bundle) {
        LocalFileLoader loader = null;
        switch (id){
            case ALL_BOOK_FILE:
                loader = new LocalFileLoader(context);
                break;
            default:
                loader = null;
                break;
        }
        if (loader != null) {
            return loader;
        }

        throw new IllegalArgumentException("The id of Loader is invalid!");
    }
}

```

LocalFileLoader.java

```java
package com.mp.android.apps.monke.monkeybook.utils.media;

import android.content.Context;
import android.database.Cursor;
import android.net.Uri;
import android.provider.MediaStore;
import android.text.TextUtils;

import androidx.annotation.NonNull;
import androidx.loader.content.CursorLoader;

import java.io.File;
import java.sql.Blob;
import java.util.ArrayList;
import java.util.List;

public class LocalFileLoader extends CursorLoader {
    private static final String TAG = "LocalFileLoader";

    private static final Uri FILE_URI = Uri.parse("content://media/external/file");
    private static final String SELECTION = MediaStore.Files.FileColumns.DATA + " like ?";
    private static final String SEARCH_TYPE = "%.txt";
    private static final String SORT_ORDER = MediaStore.Files.FileColumns.DISPLAY_NAME + " DESC";
    private static final String[] FILE_PROJECTION = {
            MediaStore.Files.FileColumns.DATA,
            MediaStore.Files.FileColumns.DISPLAY_NAME
    };

    public LocalFileLoader(Context context) {
        super(context);
        initLoader();
    }

    /**
     * 为 Cursor 设置默认参数
     */
    private void initLoader() {
        setUri(FILE_URI);
        setProjection(FILE_PROJECTION);
        setSelection(SELECTION);
        setSelectionArgs(new String[]{SEARCH_TYPE});
        setSortOrder(SORT_ORDER);
    }

    public void parseData(Cursor cursor, final MediaStoreHelper.MediaResultCallback resultCallback) {
        List<File> files = new ArrayList<>();
        // 判断是否存在数据
        if (cursor == null) {
            // TODO:当媒体库没有数据的时候，需要做相应的处理
            // 暂时直接返回空数据
            resultCallback.onResultCallback(files);
            return;
        }
        // 重复使用Loader时，需要重置cursor的position；
        cursor.moveToPosition(-1);
        while (cursor.moveToNext()) {
            String path;

            path = cursor.getString(cursor.getColumnIndexOrThrow(MediaStore.Files.FileColumns.DATA));
            // 路径无效
            if (TextUtils.isEmpty(path)) {
                continue;
            } else {
                File file = new File(path);
                if (file.isDirectory() || !file.exists()){
                    continue;
                }
                else {
                    files.add(file);
                }
            }
        }
        if (resultCallback != null) {
            resultCallback.onResultCallback(files);
        }
    }

}

```

以上是漫品 客户端加载本地文件的方式，欢迎有更好方式的童鞋留言。

[https://github.com/AnyMarvel/ManPinAPP](https://github.com/AnyMarvel/ManPinAPP)

欢迎各位大大 start

漫品 客户端小说模块：

- 基于mvp架构进行代码布局,降低代码耦合度。
- 采用 sql 数据库对数据进行存储。
- 支持小说更新提示。

阅读页支持：

- 支持翻页动画:仿真翻页、覆盖翻页、上下滚动翻页等翻页效果。
- 支持页面定制:亮度调节、背景调节、字体大小调节
- 支持全屏模式(含有虚拟按键的手机)、音量键翻页
- 支持页面进度显示、页面切换、上下章切换。
- 支持在线章节阅读、本地书籍查找。
- 支持本地书籍加载到页面(支持本地书籍分章、加载速度快、耗费内存少)
