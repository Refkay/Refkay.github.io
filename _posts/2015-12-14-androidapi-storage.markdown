---
layout:     post
title:      "安卓持久化存储"
date:       2015-12-14 16:00:00
author:     "Lemon Yang"
tags:
    - android
    - api
---


>译自android.com

安卓为你是实现持久化数据提供了几种选择。具体的方案选择取决于你的特定需要，比如存储的数据是应用私有的还是对其他应用程序也是可见的，存储你的数据需要多大的空间。

你存储数据的选择如下：

	Shared Preferences 以键值对的形式保持私有的原始数据
	Internal Storage 在设备的内存内保存私有的数据
	External Storage 在外部存储器保存公开的数据
	SQLite Databases 在私有化的表中保存具备结构的数据
	Network Connection 将数据保存在你的个人的网络服务器

安卓通过content provider为你提供了一种向其他应用暴露你的隐私数据的方案。content provider是一个可以选择实现的帮助你暴露你的应用数据的读写权限的组件，是一个
>subject to whatever restrictions you want to impose.

## Using Shared Preferences

**SharedPreferences**这个类提供了一个允许你以保存和索取键值对类型的私密数据的框架。你可以使用SharedPreferences保存任何私密数据，比如布尔类型、浮点型、整型、长整型和字符串数据。这些数据会一直被持久化保存在整个用户期间（即使你的应用进程已经被杀死）

**为了获得一个SharedPreferences对象，你可以使用下面任何一种方法：**

* getSharedPreferences()使用这个方法，你可以通过指定文件的名字获得一个特定的配置文件（这种情况下，应用内会存在多个不同名字的配置文件）

* getPreferences()使用这个方法如果你只需要为你的activity获得唯一一个配置文件的对象。

你可以通过以下的步骤将数据写进文件：

	1.调用edit()这个方法获取一个SharedPreferences.Editor.对象
	2.通过诸如putBoolean()andputString()的方法添加数据
	3.使用commit()方法提交数据

下面是一个具体的例子



    publicclassCalcextendsActivity{
    publicstaticfinalStringPREFS_NAME="MyPrefsFile";
    @Override
    protectedvoidonCreate(Bundlestate){
    super.onCreate(state);
    ...
    // Restore preferences
    SharedPreferencessettings=getSharedPreferences(PREFS_NAME,0);
    booleansilent=settings.getBoolean("silentMode",false);
    setSilent(silent);
    }
    @Override
    protectedvoidonStop(){
    super.onStop();
    // We need an Editor object to make preference changes.
    // All objects are from android.context.Context
    SharedPreferencessettings=getSharedPreferences(PREFS_NAME,0);
    SharedPreferences.Editoreditor=settings.edit();
    editor.putBoolean("silentMode",mSilentMode);
    // Commit the edits!
    editor.commit();
    }
    }


## Using the Internal Storage

你可以直接直接将数据保存在设备的内部存储器。默认的情况下，保存在内部存储设备的文件是隐秘的，即只有你自己的应用可以使用，其他应用，包括用户没有权限读写这些文件。当你的应用被卸载的时候，这些文件也会被移除。

**你可以通过以下步骤创建一个文件并将其保存到你的内部存储设备：**

* 调用openFileOutput()这个方法，并将文件的名字以及操作的模式作为参数传入，这个方法会返回一个FileOutputStream

* 利用write()将数据写入文件

* 利用close()这个方法关闭输入流

例如：

    StringFILENAME="hello_file";
    Stringstring="hello world!";
    FileOutputStreamfos=openFileOutput(FILENAME,Context.MODE_PRIVATE);
    fos.write(string.getBytes());
    fos.close();

**MODE_PRIVATE**将会创建一个文件（或者取代原来的同名文件），并且文件只有应用本书可以操作。其他有效的操作模式包括：**MODE_APPEND,MODE_WORLD_READABLE, andMODE_WORLD_WRITEABLE**

**你可以通过以下步骤读取内部存储的文件数据：**

* 调用openFileInput()方法并将文件名字作为参数传入。这个方法会返回一个FileInputStream.

* 利用read()按字节读取文件中的内容

* 利用close()关闭输出流

>Tip:If you want to save a static file in your application at compile time, save the file in your projectres/raw/directory. You can open it withopenRawResource(), passing theR.raw.resource ID. This method returns anInputStreamthat you can use to read the file (but you cannot write to the original file).

## Saving cache files

如果你只是想保存临时文件而并不是将数据持久化保存，你可以使用**getCacheDir()**的方法打开一个目录，这是一个保存你的应用的临时的缓存文件的目录。

当你的设备的内部存储容量不足的时候，安卓系统会自动的删除这些缓存文件以便于回收空间。尽管如此，你不应该依赖于系统的自动回收功能，你应该自己控制这些缓存文件并维持一个合理的内存占用显示，比如1mb。当你的应用被卸载的时候，这些文件就会被移除。

Other useful methods

	getFilesDir() 获取内部文件保存的绝对路径
	getDir() 在内部存储系统中创建（存在的时候就直接打开）一个目录
	deleteFile() 删除保存在内部文件系统的文件
	fileList() 返回当前应用保存的文件

### Using the External Storage

每一个安卓设备都支持外部存储器，因此你可以用来保存你的文件。这可以是一个可移除的存储器（比如sd卡），也可以是内置的（不可移除）存储器。存储在外部存储器的文件是全局可读的并且用户可以修改这些文件当他们通过usb大容量数据与电脑连接的时候。

>Caution:External storage can become unavailable if the user mounts the external storage on a computer or removes the media, and there's no security enforced upon files you save to the external storage. All applications can read and write files placed on the external storage and the user can remove them.

为了读写外部存储设备的文件，你的应用需要获得这些权限：**READ_EXTERNAL_STORAGE、WRITE_EXTERNAL_STORAGE**


在你使用外部存储设备的时候，你应该调用方法检测外部存储设备是否可用。外部存储设备有可能正被挂载到电脑、缺失、只读或者其他的状态，例如：

    /* Checks if external storage is available for read and write */
    publicbooleanisExternalStorageWritable(){
    Stringstate=Environment.getExternalStorageState();
    if(Environment.MEDIA_MOUNTED.equals(state)){
    return true;
    }
    return false;
    }
 

    /* Checks if external storage is available to at least read */
    publicbooleanisExternalStorageReadable(){
    Stringstate=Environment.getExternalStorageState();
    if(Environment.MEDIA_MOUNTED.equals(state)||
    Environment.MEDIA_MOUNTED_READ_ONLY.equals(state)){
    return true;
    }
    return false;
    }

一般情况下，用户能够通过你的应用打开的文件应该被存储在一个公共的路径下并且其他应用可以访问，并且用户可以轻易的在设备上拷贝这些文件。当你这样做的时候，你应该使用任意一个共享的目录，比如：**Music/,Pictures/, andRingtones/.**

为了获得一个合适的公共目录的file对象，你可以调用**getExternalStoragePublicDirectory()**这个方法并将路径的类型作为参数传入，比如**DIRECTORY_MUSIC,DIRECTORY_PICTURES,DIRECTORY_RINGTONES**或者其他。通过将文件保存在直接的媒体类型的目录内，系统的多媒体扫描器可以将你的文件进行合适的分类。比如：

    publicFilegetAlbumStorageDir(StringalbumName){
    // Get the directory for the user's public pictures directory.
    File file=newFile(Environment.getExternalStoragePublicDirectory(
    Environment.DIRECTORY_PICTURES),albumName);
    if(!file.mkdirs()){
    Log.e(LOG_TAG,"Directory not created");
	}
	return file;
	}

### Saving files that are app-private

如果你正在处理一些文件你并不打算向其他应用程序公开，你应该通过调用方法**getExternalFilesDir()**.使用外部存储器中的隐私目录。这个方法也可以指定一个类型参数作为子目录。如果你并不需要一个指定的路径，你可以传一个空值获取你应用的隐私目录的根目录。

从安卓4.4开始，读写本应用的隐私目录并不需要提供**READ_EXTERNAL_STORAGEorWRITE_EXTERNAL_STORAGE**这两个权限，所以你可以通过添加**maxSdkVersion**这个属性声明只有在低版本的系统中才需要加入权限：
>android:maxSdkVersion="18"/>

...

>Note:When the user uninstalls your application, this directory and all its contents are deleted. Also, the system media scanner does not read files in these directories, so they are not accessible from theMediaStorecontent provider. As such, youshould not use these directoriesfor media that ultimately belongs to the user, such as photos captured or edited with your app, or music the user has purchased with your app—those files should besaved in the public directories.

有时候，设备已经从内部存储器中分割一部分内存作为外部存储并且同时又提供了一个sd卡的拓展卡槽。当这样的设备搭载的系统低于安卓4.3（包括4.3）的时候，方法**getExternalFilesDir()**只提供了读写内部存储的权限并且你的应用并不可以对sd卡进行读写操作。然而，从安卓4.4开始，你可以通过调用方法**getExternalFilesDir()**获取内部和外部的读写权限，这时候方法会返回一个包含两个路径的file数组。第一个数组元素是最主要的外部存储设备的路径，你应该使用这一个路径除非它的存储空间已满或者不可用。如果你想要在系统为4.3及以下的设备上获取两个路径的权限。你可以使用支持库的静态方法**ContextCompat.getExternalFilesDirs()**。这个方法会返回一个file的数组，但是通常情况下，在4.3及以下的系统只会得到一个元素。

>CautionAlthough the directories provided bygetExternalFilesDir()andgetExternalFilesDirs()are not accessible by theMediaStorecontent provider, other apps with theREAD_EXTERNAL_STORAGEpermission can access all files on the external storage, including these. If you need to completely restrict access for your files, you should instead write your files to theinternal storage.

### Saving cache files

你可以调用方法**getExternalCacheDir()**获得内部存储器的一个用于存储缓存文件的file对象。如果你的应用被卸载的话，这写文件也会被移除。

与上面提及到的方法**ContextCompat.getExternalFilesDirs()**相类似，你可以获得第二存储器中的缓存目录通过调用方法**ContextCompat.getExternalCacheDirs()。**

>Tip:To preserve file space and maintain your app's performance, it's important that you carefully manage your cache files and remove those that aren't needed anymore throughout your app's lifecycle.

## Using Databases

安卓提供了对sqlite数据库的全部支持。并且通过你创建的数据库的名字可以在应用的每一个类中使用，但是不会超出你的应用。

推荐你使用继承SQLiteOpenHelper并重写onCreate()这个方法来创建一个新的数据库。在onCreate()方法中你可以执行一个sqlite命令在数据库中创建一张表，例如：

	publicclassDictionaryOpenHelperextendsSQLiteOpenHelper{
	privatestaticfinalintDATABASE_VERSION=2;
	privatestaticfinalStringDICTIONARY_TABLE_NAME="dictionary";
	privatestaticfinalStringDICTIONARY_TABLE_CREATE=
	"CREATE TABLE "+DICTIONARY_TABLE_NAME+" ("+
	KEY_WORD+" TEXT, "+
	KEY_DEFINITION+" TEXT);";
	DictionaryOpenHelper(Contextcontext){
	super(context,DATABASE_NAME,null,DATABASE_VERSION);
	}
	@Override
	publicvoidonCreate(SQLiteDatabasedb){
	db.execSQL(DICTIONARY_TABLE_CREATE);
	}
	}

你可以获得一个你已经实现的SQLiteOpenHelper的实例通过你已经定义好的构造器。你可以调用**getWritableDatabase(),andgetReadableDatabase()**这两个方法分别获取可读和可写的数据库对象。

你可以通过调用**SQLiteDatabase**的*query()*方法执行sqlite 的查询操作。query()方法乐意接受多个参数，包括要查询的表的名字、要查询的列名、分组列名等。对于那些复杂的查询操作，比如需要列的同义名的，你可以使用**SQLiteQueryBuilderSQLiteQueryBuilder**，它可以提供多种便捷的方法执行查询操作。

每一个sqlite的查询操作会返回一个指向所有查询到的行数据的指针。你可以利用这个指针实现查询结果的导航并且读取相关的行列信息。