---
title: findbugs clean
categories:
  - 开发者手册
字数统计: WordCount
阅读时长预计: Min2Read
总字数统计: TotalCount
date: 2020-05-13 21:57:43
tags: findbugs
---

收集的findbugs清理指导！

<!--more-->

1、Bug: Hard coded reference to an absolute pathname
BUG描述：This code constructs a File object using a hard coded to an absolute pathname（此代码包含文件对象为一个绝对路径名）
 
问题原因：硬编码指向绝对路径。
 
File preFile = new File(PREFERENCES_FILE_FULL_PATH);
 
而private static final String PREFERENCES_FILE_FULL_PATH =
 
        "/data/data/com.android.mms/shared_prefs/auto_downLoad.xml";
 
PREFERENCES_FILE_FULL_PATH声明为了final型，不可变的。如果后续文件路径有变更，引用不到了，但路径又不可更改，就会引入问题。
 
解决方法：去掉final。
 
2、Bug: Pattern: Dead store to local variable
 
BUG描述：This instruction assigns a value to a local variable, but the value is not read or used in any subsequent instruction. Often, this indicates an error, because the value computed is never used. （该指令为局部变量赋值，但在其后的没有对她做任何使用。通常，这表明一个错误，因为值从未使用过。）
 
问题原因：锁屏中提示Dead store to velocityX，分析代码
 
case MotionEvent.ACTION_POINTER_1_UP语句中定义了局部变量velocityX，并且只在if ((mDirectionFlag && velocityX > 0)||(!mDirectionFlag && velocityX < 0))
 
                    velocityX = -velocityX;中赋值后并未再使用。因此没有赋值的必要，并且分析代码不需要该变量，可以去除。
 
解决方法：去掉velocityX变量的定义及赋值。
 
3、BUG: Inconsistent synchronization
BUG描述：The fields of this class appear to be accessed inconsistently with respect to synchronization. （不合理的同步）
 
问题原因：根据描述ConfigLoader文件中mUnlockAppDataMap在46%的时间内都是处于被锁状态。分析代码mUnlockAppDataMap是在checkUnlockAppConfigChange这个函数中被锁的。而该方法public synchronized boolean checkUnlockAppConfigChange(Context context)没有地方调用。
 
解决方法：去掉synchronized关键字。
 
4、BUG: Incorrect lazy initialization of static field
 
BUG描述：This method contains an unsynchronized lazy initialization of a non-volatile static field. Because the compiler or processor may reorder instructions, threads are not guaranteed to see a completely initialized object, if the method can be called by multiple threads.（这种方法包含了一个不同步延迟初始化的非volatile静态字段。因为编译器或处理器可能会重新排列指令，如果该方法可以被多个线程调用，线程不能保证看到一个完全初始化的对象。）
 
问题原因：sInstance 是static型，clean()方法可能被多个线程调用，在sInstance判断为非空后，再清空置null时可能会有问题。
 
解决方法：给clean()加上关键字synchronized .public static synchronized void clean()
 
5、BUG: Redundant nullcheck of value known to be non-null
 
BUG描述：This method contains a redundant check of a known non-null value against the constant null.(方法中对不为空的值进行为空的判断。)
 
问题原因：分析findbugs报错的这段代码
 
if(mInputStream == null){
 
Log.i(TAG , " mInputStream is null ");
 
return;
 
}
 
mDBuilder = mDBuilderFactory.newDocumentBuilder();
 
if(mInputStream != null) {
 
mDocument = mDBuilder.parse(mInputStream);
 
}else {
 
Log.i(TAG , "mInputStream ==  null");
 
return;
 
}
 
mInputStream若为null，则直接返回。后面不需要再有if(mInputStream != null)的判断。
 
解决方法：在mInputStream判空后不再重复判空，将后面if判断中的mInputStream改为mDBuilder。
 
6、BUG: Should be a static inner class
 
BUG描述：This class is an inner class, but does not use its embedded reference to the object which created it.  This reference makes the instances of the class larger, and may keep the reference to the creator object alive longer than necessary.  If possible, the class should be made static.（若成员类中未访问外围类的非静态成员，为避免额外的空间和时间开销，建议改用静态成员类。）
 
问题原因：非静态成员类和静态成员类的区别在于，非静态成员类是对象的，静态成员类是类的。非静态成员类可以访问外围类的任何成员，但前提是必须存在外围类对象。JAVA需要额外维护非静态成员类和外围类对象的关系。分析代码private class IccText和private class MediaMetadata {没有访问到外围类的非静态成员，所以findbugs建议将其设为static型。
 
解决方法：将这2个内部类改为static型。
 
7、BUG: Switch statement found where one case falls through to the next case
BUG描述：This method contains a switch statement where one case branch will fall through to the next case. Usually you need to end this case with a break or return.（Switch语句中一个分支执行后又执行了下一个分支。通常case后面要跟break或者return语句来跳出。）
 
问题原因：case MotionEvent.ACTION_UP执行完之后没有break，会继续走case MotionEvent.ACTION_CANCEL分支。分析代码逻辑，手指抬起后，锁屏图标需要回到初始位置，而回到初始位置的逻辑是在ACTION_CANCEL里做的。即ACTION_UP后的逻辑还需要ACTION_CANCEL里面的逻辑。
 
解决方法：将ACTION_CANCEL中的逻辑拉出来做成一个函数，ACTION_UP逻辑后调用这个函数后再做break操作。
 
8、BUG: Unread field
BUG描述：This field is never read.  Consider removing it from the class.（类中定义的属性从未被调用，建议删除。）
 
问题原因：在类中定义了成员变量private HwViewProperty mCondition = null;代码中只有赋值操作mCondition = new HwViewProperty(mContext,value, ViewPropertyType.TYPE_CONDITION, mCallback);但没有使用这个变量mCondition的地方。
 
解决方法：去掉mCondition的定义及赋值语句。但需注意，mCondition = new HwViewProperty(mContext,value, ViewPropertyType.TYPE_CONDITION, mCallback);赋值中，虽然mCondition变量后续没有使用到，但new HwViewProperty对象调用HwViewProperty的构造方法时，其实是做了功能操作的。因此，去掉mCondition，但需要保留new HwViewProperty(mContext,value, ViewPropertyType.TYPE_CONDITION, mCallback);
 
9、BUG: Write to static field from instance method
BUG描述：This instance method writes to a static field. This is tricky to get correct if multiple instances are being manipulated, and generally bad practice.（实例方法直接写静态变量。）
 
问题原因：sInstance是类的静态成员变量，非静态方法unregisterCallbaks直接对其赋值，非静态方法是与对象相关联的，当多个对象同时对该变量进行赋值时可能出现问题。
 
解决方法：在使用静态成员变量时使用get和set方法。

May expose internal representation by incorporating reference to mutable object
分析
主要是针对Date类的get/set方法，涉及了深拷贝和浅拷贝的问题。
解决方法：
public Date getCreateDate() {
       return (Date) createDate.clone();
   }

   public void setCreateDate(Date createDate) {
       this.createDate = (Date) createDate.clone();
   }

Write to static field from instance method
原因：在其他类对static属性进行直接修改
解决方法：将static属性设置为private或者protected形式，使用set/get方法进行修改和获取。

Field should be package protected
A mutable static field could be changed by malicious code or by accident. The field could be made package protected to avoid this vulnerability.

一、Security 关于代码安全性防护

DMI_CONSTANT_DB_PASSWORD
代码中创建DB的密码时采用了写死的密码。

DMI_EMPTY_DB_PASSWORD
创建数据库连接时没有为数据库设置密码，这会使数据库没有必要的保护。

HRS_REQUEST_PARAMETER_TO_COOKIE
此代码使用不受信任的HTTP参数构造一个HTTP Cookie。

HRS_REQUEST_PARAMETER_TO_HTTP_HEADER
在代码中直接把一个HTTP的参数写入一个HTTP头文件中，它为HTTP的响应暴露了漏洞。

SQL_NONCONSTANT_STRING_PASSED_TO_EXECUTE
该方法以字符串的形式来调用SQLstatement的execute方法，它似乎是动态生成SQL语句的方法。这会更容易受到SQL注入攻击。

XSS_REQUEST_PARAMETER_TO_JSP_WRITER
在代码中在JSP输出中直接写入一个HTTP参数，这会造成一个跨站点的脚本漏洞。

二、Experimental 关于代码实验性问题
LG_LOST_LOGGER_DUE_TO_WEAK_REFERENCE
OpenJDK的引入了一种潜在的不兼容问题，特别是，java.util.logging.Logger的行为改变时。它现在使用内部弱引用，而不是强引用。–logger配置改变，它就是丢失对logger的引用，这本是一个合理的变化，但不幸的是一些代码对旧的行为有依赖关系。这意味着，当进行垃圾收集时对logger配置将会丢失。例如：

public static void initLogging() throws Exception {

 Logger logger = Logger.getLogger("edu.umd.cs");

 logger.addHandler(new FileHandler()); // call to change logger configuration

 logger.setUseParentHandlers(false); // another call to change logger configuration

}

该方法结束时logger的引用就丢失了，如果你刚刚结束调用initLogging方法后进行垃圾回收，logger的配置将会丢失（因为只有保持记录器弱引用）。

public static void main(String[] args) throws Exception {

 initLogging(); // adds a file handler to the logger

 System.gc(); // logger configuration lost

 Logger.getLogger("edu.umd.cs").info("Some message"); // this isn't logged to the file as expected

}

OBL_UNSATISFIED_OBLIGATION
这种方法可能无法清除（关闭，处置）一个流，数据库对象，或其他资源需要一个明确的清理行动。

一般来说，如果一个方法打开一个流或其他资源，该方法应该使用try / finally块来确保在方法返回之前流或资源已经被清除了。这种错误模式基本上和OS_OPEN_STREAM和ODR_OPEN_DATABASE_RESOURCE错误模式相同，但是是在不同在静态分析技术。我们正为这个错误模式的效用收集反馈意见。

 

三、Bad practice代码实现中的一些坏习惯
AM_CREATES_EMPTY_JAR_FILE_ENTRY
调用putNextEntry()方法写入新的 jar 文件条目时立即调用closeEntry()方法。这样会造成JarFile条目为空。

AM_CREATES_EMPTY_ZIP_FILE_ENTRY
调用putNextEntry()方法写入新的 zip 文件条目时立即调用closeEntry()方法。这样会造成ZipFile条目为空。

BC_EQUALS_METHOD_SHOULD_WORK_FOR_ALL_OBJECTS
equals(Object o)方法不能对参数o的类型做任何的假设。比较此对象与指定的对象。当且仅当该参数不为 null，并且是表示与此对象相同的类型的对象时，结果才为 true。

DMI_RANDOM_USED_ONLY_ONCE
随机创建对象只使用过一次就抛弃

BIT_SIGNED_CHECK
检查位操作符运行是否合理

((event.detail & SWT.SELECTED) > 0)

If SWT.SELECTED is a negative number, this is a candidate for a bug. Even when SWT.SELECTED is not negative, it seems good practice to use '!= 0' instead of '> 0'.

CN_IDIOM
按照惯例，实现此接口的类应该使用公共方法重写 Object.clone（它是受保护的），以获得有关重写此方法的详细信息。此接口不 包含 clone 方法。因此，因为某个对象实现了此接口就克隆它是不可能的,应该实现此接口的类应该使用公共方法重写 Object.clone

CN_IDIOM_NO_SUPER_CALL
一个非final类型的类定义了clone()方法而没有调用super.clone()方法。例如：B扩展自A，如果B中clone方法调用了spuer.clone（），而A中的clone没有调用spuer.clone()，就会造成结果类型不准确。要求A的clone方法中调用spuer.clone()方法。

CN_IMPLEMENTS_CLONE_BUT_NOT_CLONEABLE
类中定义了clone方法但是它没有实现Cloneable接口

CO_ABSTRACT_SELF
抽象类中定义了多个compareTo()方法，正确的是覆写Comparable中的compareTo方法，方法的参数为Object类型，如下例：

int compareTo(T o)  比较此对象与指定对象的顺序。

CO_SELF_NO_OBJECT
类中定义了多个compareTo()方法，正确的是覆写Comparable中的compareTo方法，方法的参数为Object类型

DE_MIGHT_DROP
方法可能抛出异常

DE_MIGHT_IGNORE
此方法可能忽略异常。通常，应该以某种方式处理或报告异常，或者将异常抛出方法。

DMI_USING_REMOVEALL_TO_CLEAR_COLLECTION
不要用removeAll方法去clear一个集合

DP_CREATE_CLASSLOADER_INSIDE_DO_PRIVILEGED
类加载器只能建立在特殊的方法体内

DM_EXIT
在方法中调用System.exit(...)语句，考虑用RuntimeException来代替

DM_RUN_FINALIZERS_ON_EXIT
在方法中调用了System.runFinalizersOnExit 或者Runtime.runFinalizersOnExit方法，因为这样做是很危险的。

ES_COMPARING_PARAMETER_STRING_WITH_EQ
用==或者!=方法去比较String类型的参数

ES_COMPARING_STRINGS_WITH_EQ
用==或者！=去比较String类型的对象

EQ_ABSTRACT_SELF
This class defines a covariant version of equals(). To correctly override the equals() method in java.lang.Object, the parameter of equals() must have type java.lang.Object.

 

这个类定义了equals()方法，但是参数却是Object的子类。正确覆盖equals()方法，参数必须是Object

 

EQ_CHECK_FOR_OPERAND_NOT_COMPATIBLE_WITH_THIS
equals方法检查不一致的操作。两个类根本就是父子关系而去调用equals方法去判读对象是否相等。

public boolean equals(Object o) {

  if (o instanceof Foo)

    return name.equals(((Foo)o).name);

  else if (o instanceof String)

    return name.equals(o);

  else return false;

EQ_COMPARETO_USE_OBJECT_EQUALS
类中定义了compareTo方法但是继承了Object中的compareTo方法

22.Eq: equals method fails for subtypes (EQ_GETCLASS_AND_CLASS_CONSTANT)

类中的equals方法可能被子类中的方法所破坏，当使用类似于Foo.class == o.getClass()的判断时考虑用this.getClass() == o.getClass()来替换

EQ_SELF_NO_OBJECT
类中定义了多个equals方法。正确的做法是覆写Object中的equals方法，它的参数为Object类型的对象。

FI_EMPTY
为空的finalizer方法应该删除。一下关于finalizer的内容省略

GC_UNCHECKED_TYPE_IN_GENERIC_CALL
This call to a generic collection method passes an argument while compile type Object where a specific type from the generic type parameters is expected. Thus, neither the standard Java type system nor static analysis can provide useful information on whether the object being passed as a parameter is of an appropriate type.

此对泛型集合方法的调用在编译类型对象时传递一个参数，其中预期泛型类型参数中的特定类型。因此，标准Java类型系统和静态分析都不能提供关于作为参数传递的对象是否属于适当类型的有用信息。

HE_EQUALS_NO_HASHCODE
方法定义了equals方法却没有定义hashCode方法

HE_HASHCODE_NO_EQUALS
 类定义了hashCode方法去没有定义equal方法

HE_EQUALS_USE_HASHCODE
一个类覆写了equals方法，没有覆写hashCode方法，使用了Object对象的hashCode方法

HE_INHERITS_EQUALS_USE_HASHCODE
子类继承了父类的equals方法却使用了Object的hashCode方法

IC_SUPERCLASS_USES_SUBCLASS_DURING_INITIALIZATION
子类在父类未初始化之前使用父类对象实例

public class CircularClassInitialization {

static class InnerClassSingleton extends CircularClassInitialization {

static InnerClassSingleton singleton = new InnerClassSingleton();

}

static CircularClassInitialization foo = InnerClassSingleton.singleton;

}

IMSE_DONT_CATCH_IMSE
捕捉违法的监控状态异常，例如当没有获取到对象锁时使用其wait和notify方法

ISC_INSTANTIATE_STATIC_CLASS
为使用静态方法而创建一个实例对象。调用静态方法时只需要使用类名+静态方法名就可以了。

IT_NO_SUCH_ELEMENT
迭代器的next方法不能够抛出NoSuchElementException

J2EE_STORE_OF_NON_SERIALIZABLE_OBJECT_INTO_SESSION
在HttpSession对象中保存非连续的对象

JCIP_FIELD_ISNT_FINAL_IN_IMMUTABLE_CLASS
 The class is annotated with net.jcip.annotations.Immutable, and the rules for that annotation require that all fields are final. .

该类使用net.jcip.annotations进行注释。不可变的，该注释的规则要求所有字段都是final

NP_BOOLEAN_RETURN_NULL
返回值为boolean类型的方法直接返回null，这样会导致空指针异常

NP_EQUALS_SHOULD_HANDLE_NULL_ARGUMENT
变量调用equals方法时没有进行是否为null的判断

NP_TOSTRING_COULD_RETURN_NULL
toString方法可能返回null

NM_CLASS_NAMING_CONVENTION
类的名称以大写字母名称开头

NM_CLASS_NOT_EXCEPTION
类的名称中含有Exception但是却不是一个异常类的子类，这种名称会造成混淆

NM_CONFUSING
令人迷惑的方面命名

NM_FIELD_NAMING_CONVENTION
非final类型的字段需要遵循驼峰命名原则

NM_FUTURE_KEYWORD_USED_AS_IDENTIFIER
验证是否是java预留关键字

NM_FUTURE_KEYWORD_USED_AS_MEMBER_IDENTIFIER
验证是否时java中的关键字

NM_METHOD_NAMING_CONVENTION
方法名称以小写字母开头

NM_SAME_SIMPLE_NAME_AS_INTERFACE
实现同一接口实现类不能使用相同的名称，即使它们位于不同的包中

NM_SAME_SIMPLE_NAME_AS_SUPERCLASS
继承同一父类的子类不能使用相同的名称，即使它们位于不同的包中

NM_VERY_CONFUSING_INTENTIONAL
很容易混淆的方法命名，例如方法的名称名称使用使用大小写来区别两个不同的方法。

NM_WRONG_PACKAGE_INTENTIONAL
由于错误引用了不同包中相同类名的对象而不能够正确的覆写父类中的方法

import alpha.Foo;

public class A {

  public int f(Foo x) { return 17; }

}

import beta.Foo;

public class B extends A {

  public int f(Foo x) { return 42; }

  public int f(alpha.Foo x) { return 27; }

}

ODR_OPEN_DATABASE_RESOURCE
方法中可能存在关闭数据连接失败的情况

OS_OPEN_STREAM
方法中可能存在关闭流失败的情况

OS_OPEN_STREAM_EXCEPTION_PATH
方法中可能存在关闭流时出现异常情况

RC_REF_COMPARISON_BAD_PRACTICE
当两者为不同类型的对象时使用equals方法来比较它们的值是否相等，而不是使用==方法。例如比较的两者为java.lang.Integer, java.lang.Float

RC_REF_COMPARISON_BAD_PRACTICE_BOOLEAN
使用== 或者 !=操作符来比较两个 Boolean类型的对象，建议使用equals方法。

RR_NOT_CHECKED
InputStream.read方法忽略返回的多个字符，如果对结果没有检查就没法正确处理用户读取少量字符请求的情况。

SR_NOT_CHECKED
InputStream.skip()方法忽略返回的多个字符，如果对结果没有检查就没法正确处理用户跳过少量字符请求的情况

RV_RETURN_VALUE_IGNORED_BAD_PRACTICE
方法忽略返回值的异常信息

SI_INSTANCE_BEFORE_FINALS_ASSIGNED
在所有的static final字段赋值之前去使用静态初始化的方法创建一个类的实例。

SE_BAD_FIELD_STORE
非序列化的值保存在声明为序列化的的非序列化字段中

SE_COMPARATOR_SHOULD_BE_SERIALIZABLE
Comparator接口没有实现Serializable接口

SE_INNER_CLASS
序列化内部类

SE_NONFINAL_SERIALVERSIONID
关于UID类的检查内容省略

SE_NO_SUITABLE_CONSTRUCTOR
子类序列化时父类没有提供一个void的构造函数

SE_NO_SUITABLE_CONSTRUCTOR_FOR_EXTERNALIZATION
Externalizable 实例类没有定义一个void类型的构造函数

SE_READ_RESOLVE_MUST_RETURN_OBJECT
readResolve从流中读取类的一个实例，此方法必须声明返回一个Object类型的对象

SE_TRANSIENT_FIELD_NOT_RESTORED
This class contains a field that is updated at multiple places in the class, thus it seems to be part of the state of the class. However, since the field is marked as transient and not set in readObject or readResolve, it will contain the default value in any deserialized instance of the class.

该类包含一个在类中的多个位置更新的字段，因此它似乎是类状态的一部分。但是，由于字段被标记为transient，并且没有在readObject或readResolve中设置，所以它将在类的任何反序列化实例中包含默认值。

SE_NO_SERIALVERSIONID
一个类实现了Serializable接口但是没有定义serialVersionUID类型的变量。序列化运行时使用一个称为 serialVersionUID 的版本号与每个可序列化类相关联，该序列号在反序列化过程中用于验证序列化对象的发送者和接收者是否为该对象加载了与序列化兼容的类。如果接收者加载的该对象的类的 serialVersionUID 与对应的发送者的类的版本号不同，则反序列化将会导致 InvalidClassException。可序列化类可以通过声明名为 "serialVersionUID" 的字段（该字段必须是静态 (static)、最终 (final) 的 long 型字段）显式声明其自己的 serialVersionUID： 

 ANY-ACCESS-MODIFIER static final long serialVersionUID = 42L;

UI_INHERITANCE_UNSAFE_GETRESOURCE
当一个类被子类继承后不要使用this.getClass().getResource(...)来获取资源

四、Correctness关于代码正确性相关方面的
BC_IMPOSSIBLE_CAST
不可能的类转换，执行时会抛出ClassCastException

BC_IMPOSSIBLE_DOWNCAST
父类在向下进行类型转换时抛出ClassCastException

BC_IMPOSSIBLE_DOWNCAST_OF_TOARRAY
集合转换为数组元素时发生的类转换错误。

This code is casting the result of calling toArray() on a collection to a type more specific than Object[], as in: 

String[] getAsArray(Collection<String> c) {

  return (String[]) c.toArray();

  }

This will usually fail by throwing a ClassCastException. The toArray() of almost all collections return an Object[]. They can't really do anything else, since the Collection object has no reference to the declared generic type of the collection. 

The correct way to do get an array of a specific type from a collection is to use c.toArray(new String[]); or c.toArray(new String[c.size()]); (the latter is slightly more efficient). 

BC_IMPOSSIBLE_INSTANCEOF
采用instaneof方法进行比较时总是返回false。前提是保证它不是由于某些逻辑错误造成的。

BIT_AND
错误的使用&位操作符，例如(e & C)

BIT_AND_ZZ
检查恒等的逻辑错误

BIT_IOR
错误的使用|位操作符，例如(e | C)

BIT_SIGNED_CHECK_HIGH_BIT
检查逻辑运算符操作返回的标识。例如((event.detail & SWT.SELECTED) > 0)，建议采用!=0代替>0

BOA_BADLY_OVERRIDDEN_ADAPTER
子类错误的覆写父类中用于适配监听其他事件的方法，从而导致当触发条件发生时不能被监听者调用

BX_UNBOXED_AND_COERCED_FOR_TERNARY_OPERATOR
在三元运算符操作时如果没有对值进行封装或者类型转换。例如：b ? e1 : e2

DLS_DEAD_STORE_OF_CLASS_LITERAL
以类的字面名称方式为一个字段赋值后再也没有去使用它，在1.4jdk中它会自动调用静态的初始化方法，而在jdk1.5中却不会去执行。

DLS_OVERWRITTEN_INCREMENT
覆写增量增加错误i = i++

DMI_BAD_MONTH
hashNext方法调用next方法。

DMI_COLLECTIONS_SHOULD_NOT_CONTAIN_THEMSELVES
集合没有包含他们自己本身。

DMI_INVOKING_HASHCODE_ON_ARRAY
数组直接使用hashCode方法来返回哈希码。

int [] a1 = new int[]{1,2,3,4}; System.out.println(a1.hashCode()); System.out.println(java.util.Arrays.hashCode(a1));

DMI_LONG_BITS_TO_DOUBLE_INVOKED_ON_INT
Double.longBitsToDouble invoked on an int
在int上调用了Double.longBitsToDouble

DMI_VACUOUS_SELF_COLLECTION_CALL
集合的调用不能被感知。例如c.containsAll(c)总是返回true，而c.retainAll(c)的返回值不能被感知。

DMI_ANNOTATION_IS_NOT_VISIBLE_TO_REFLECTION
Unless an annotation has itself been annotated with @Retention(RetentionPolicy.RUNTIME), the annotation can't be observed using reflection (e.g., by using the isAnnotationPresent method). .

除非注释本身使用@Retention(RetentionPolicy.RUNTIME)进行了注释，否则不能使用反射(例如，使用isAnnotationPresent方法)观察注释。

DMI_VACUOUS_CALL_TO_EASYMOCK_METHOD
While ScheduledThreadPoolExecutor inherits from ThreadPoolExecutor, a few of the inherited tuning methods are not useful for it. In particular, because it acts as a fixed-sized pool using corePoolSize threads and an unbounded queue, adjustments to maximumPoolSize have no useful effect.

虽然ScheduledThreadPoolExecutor继承自ThreadPoolExecutor，但是一些继承的调优方法对它并不有用。特别是，由于它使用corePoolSize线程和无界队列充当固定大小的池，所以对maximumPoolSize的调整没有任何有用的效果。

EC_ARRAY_AND_NONARRAY
数组对象使用equals方法和非数组对象进行比较。即使比较的双方都是数组对象也不应该使用equals方法，而应该比较它们的内容是否相等使用java.util.Arrays.equals(Object[], Object[]);

EC_INCOMPATIBLE_ARRAY_COMPARE
使用equls方法去比较类型不相同的数组。例如：String[] and StringBuffer[], or String[] and int[]

EC_NULL_ARG
调用equals的对象为null

EC_UNRELATED_CLASS_AND_INTERFACE
使用equals方法比较不相关的类和接口

EC_UNRELATED_INTERFACES
调用equals方法比较不同类型的接口

EC_UNRELATED_TYPES
调用equals方法比较不同类型的类

EC_UNRELATED_TYPES_USING_POINTER_EQUALITY
This method uses using pointer equality to compare two references that seem to be of different types. The result of this comparison will always be false at runtime.

该方法使用指针相等来比较两个似乎属于不同类型的引用。这个比较的结果在运行时总是错误的。

EQ_ALWAYS_FALSE
使用equals方法返回值总是false

EQ_ALWAYS_TRUE
equals方法返回值总是true

EQ_COMPARING_CLASS_NAMES
使用equals方法去比较一个类的实例和类的类型

EQ_DONT_DEFINE_EQUALS_FOR_ENUM
This class defines an enumeration, and equality on enumerations are defined using object identity. Defining a covariant equals method for an enumeration value is exceptionally bad practice, since it would likely result in having two different enumeration values that compare as equals using the covariant enum method, and as not equal when compared normally. Don't do it.

该类定义枚举，并且使用对象标识定义枚举上的等式。为枚举值定义一个协变等于方法是一个非常糟糕的实践，因为它可能会导致使用协变枚举方法将两个不同的枚举值作为等于进行比较，而在正常情况下比较时作为不相等。不要这样做。

EQ_OTHER_NO_OBJECT
类中定义的equals方法时不要覆写equals（Object）方法

EQ_OTHER_USE_OBJECT
类中定义的equals方法时不要覆写Object中的equals（Object）方法

EQ_OVERRIDING_EQUALS_NOT_SYMMETRIC
name=错误用法 - equals方法覆盖了父类的equals可能功能不符

EQ_SELF_USE_OBJECT
类中定义了一组equals方法，但是都是继承的java.lang.Object class中的equals(Object)方法

FE_TEST_IF_EQUAL_TO_NOT_A_NUMBER
This code checks to see if a floating point value is equal to the special Not A Number value (e.g., if (x == Double.NaN)). However, because of the special semantics of NaN, no value is equal to Nan, including NaN. Thus, x == Double.NaN always evaluates to false. To check to see if a value contained in x is the special Not A Number value, use Double.isNaN(x) (or Float.isNaN(x) if x is floating point precision).

name=错误用法 - 测试是否与NaN相等

VA_FORMAT_STRING_BAD_ARGUMENT
错误使用参数类型来格式化字符串

VA_FORMAT_STRING_BAD_CONVERSION
指定的格式字符串和参数类型不匹配，例如：String.format("%d", "1")

VA_FORMAT_STRING_EXPECTED_MESSAGE_FORMAT_SUPPLIED
但用String的format方法时实际调用了ＭｅｓｓａｇｅＦｏｒｍａｔ中干的格式化方法而引起格式化结果出错。

VA_FORMAT_STRING_EXTRA_ARGUMENTS_PASSED
使用String的format方法时有非法的参数也经过了格式化操作。

VA_FORMAT_STRING_ILLEGAL
格式化String对象语句错误

VA_FORMAT_STRING_MISSING_ARGUMENT
String的format操作缺少必要的参数。

VA_FORMAT_STRING_NO_PREVIOUS_ARGUMENT
格式字符串定义错误，例如：formatter.format("%<s %s", "a", "b"); 抛出MissingFormatArgumentException异常

GC_UNRELATED_TYPES
This call to a generic collection method contains an argument with an incompatible class from that of the collection's parameter (i.e., the type of the argument is neither a supertype nor a subtype of the corresponding generic type argument). Therefore, it is unlikely that the collection contains any objects that are equal to the method argument used here. Most likely, the wrong value is being passed to the method.

In general, instances of two unrelated classes are not equal. For example, if the Foo and Bar classes are not related by subtyping, then an instance of Foo should not be equal to an instance of Bar. Among other issues, doing so will likely result in an equals method that is not symmetrical. For example, if you define the Foo class so that a Foo can be equal to a String, your equals method isn't symmetrical since a String can only be equal to a String. 

In rare cases, people do define nonsymmetrical equals methods and still manage to make their code work. Although none of the APIs document or guarantee it, it is typically the case that if you check if a Collection<String> contains a Foo, the equals method of argument (e.g., the equals method of the Foo class) used to perform the equality checks. 

这个对泛型集合方法的调用包含一个与集合的参数(即，参数的类型既不是相应泛型类型参数的超类型，也不是相应泛型类型参数的子类型)。因此，集合不太可能包含与这里使用的方法参数相同的任何对象。很可能，传递给方法的值是错误的。

HE_SIGNATURE_DECLARES_HASHING_OF_UNHASHABLE_CLASS
A method, field or class declares a generic signature where a non-hashable class is used in context where a hashable class is required. A class that declares an equals method but inherits a hashCode() method from Object is unhashable, since it doesn't fulfill the requirement that equal objects have equal hashCodes.

方法、字段或类声明一个泛型签名，其中在需要可耐洗类的上下文中使用不可耐洗类。声明equals方法但从对象继承hashCode()方法的类是不可挂起的，因为它不满足equal对象具有相等hashCode的要求。

HE_USE_OF_UNHASHABLE_CLASS
A class defines an equals(Object) method but not a hashCode() method, and thus doesn't fulfill the requirement that equal objects have equal hashCodes. An instance of this class is used in a hash data structure, making the need to fix this problem of highest importance.

类定义了equals(Object)方法，但没有定义hashCode()方法，因此不能满足equal对象具有相等hashCode的要求。该类的一个实例用于哈希数据结构中，这使得修复这个问题变得极为重要。

ICAST_INT_CAST_TO_DOUBLE_PASSED_TO_CEIL
integral的值转换为double后使用了Math.ceil方法

ICAST_INT_CAST_TO_FLOAT_PASSED_TO_ROUND
int 类型的值转换为float类型之后调用了Math.round方法

IJU_ASSERT_METHOD_INVOKED_FROM_RUN_METHOD
在JUnit中的断言在run方法中不会被告知

IJU_BAD_SUITE_METHOD
在一个JUnit类中声明的一个suite()方法必须声明为

public static junit.framework.Test suite()

或者

public static junit.framework.TestSuite suite()的形式。

IL_CONTAINER_ADDED_TO_ITSELF
集合本身作为add方法的参数，这样会引起内容溢出。

IL_INFINITE_LOOP
方法的自调用引起的死循环

IM_MULTIPLYING_RESULT_OF_IREM
和整数余数进行乘法运算。例如：i % 60 * 1000 是进行(i % 60) * 1000运算而不是 i % (60 * 1000)

INT_BAD_COMPARISON_WITH_NONNEGATIVE_VALUE
保证非负数和负数进行比较

INT_BAD_COMPARISON_WITH_SIGNED_BYTE
比较有符合数，要先把有符号数转换为无符合数再进行比较

IO_APPENDING_TO_OBJECT_OUTPUT_STREAM
宣布试图在对象的输出流处添加元素，如果你希望能够添加进一个对象的输出流中必须保证对象的输出流处于打开状态。

IP_PARAMETER_IS_DEAD_BUT_OVERWRITTEN
The initial value of this parameter is ignored, and the parameter is overwritten here. This often indicates a mistaken belief that the write to the parameter will be conveyed back to the caller.

传入参数的值被忽略，但是对传入值进行了修改，并返回给了调用者

MF_CLASS_MASKS_FIELD
子类中定义了和父类中同名的字段。在调用时会出错

MF_METHOD_MASKS_FIELD
在方法中定义的局部变量和类变量或者父类变量同名，从而引起字段混淆。

NP_ALWAYS_NULL
对象赋为null值后 没有被重新赋值

NP_ALWAYS_NULL_EXCEPTION
A pointer which is null on an exception path is dereferenced here.  This will lead to a NullPointerException when the code is executed.  Note that because FindBugs currently does not prune infeasible exception paths, this may be a false warning.

Also note that FindBugs considers the default case of a switch statement to be an exception path, since the default case is often infeasible.

空指针引用上调用去除引用方法，将发生空指针异常

NP_ARGUMENT_MIGHT_BE_NULL
方法没有判断参数是否为空

NP_CLOSING_NULL
一个为空的对象调用close方法

NP_GUARANTEED_DEREF
There is a statement or branch that if executed guarantees that a value is null at this point, and that value that is guaranteed to be dereferenced (except on forward paths involving runtime exceptions).

在正常的null判断分支上，对象去除引用操作是受保护的不允许的

NP_GUARANTEED_DEREF_ON_EXCEPTION_PATH
There is a statement or branch on an exception path that if executed guarantees that a value is null at this point, and that value that is guaranteed to be dereferenced (except on forward paths involving runtime exceptions).

异常路径上有一个语句或分支，如果执行该语句或分支，则该语句或分支将确保此时值为空，并且该值将被取消引用(涉及运行时异常的正向路径除外)。

NP_NONNULL_PARAM_VIOLATION
方法中为null的参数没有被重新赋值 void test(){

String ss = null;

sya(ss); } public void sya(String ad){

ad.getBytes(); }

NP_NONNULL_RETURN_VIOLATION
方法声明了返回值不能为空，但是方法中有可能返回null

NP_NULL_INSTANCEOF
检查一个为null的值是否是想要的类型对象，而不是由于粗心或者逻辑错误引起的

NP_NULL_ON_SOME_PATH
对象可能没有重新赋值

NP_NULL_ON_SOME_PATH_EXCEPTION
A reference value which is null on some exception control path is dereferenced here.  This may lead to a NullPointerException when the code is executed.  Note that because FindBugs currently does not prune infeasible exception paths, this may be a false warning.

Also note that FindBugs considers the default case of a switch statement to be an exception path, since the default case is often infeasible.

在异常null值处理分支调用的方法上，可能存在对象去除引用操作

NP_NULL_PARAM_DEREF_ALL_TARGETS_DANGEROUS
方法参数中声明为nonnull类型的参数为null

void test(){

String ss = null;

sya(ss); } public void sya(@nonnull String ad){

ad.getBytes(); }

NP_STORE_INTO_NONNULL_FIELD
为一个已经声明为不能为null值的属性赋值为null。

NM_BAD_EQUAL
类中定义了一个equal方法但是却不是覆写的Object对象的equals方法

NM_LCASE_HASHCODE
类中定义了一个hashCode方法但是却不是覆写的Object中的hashCode方法

NM_LCASE_TOSTRING
类中定义了一个toString方法但是却不是覆写的Object中的toString方法

NM_METHOD_CONSTRUCTOR_CONFUSION
构造方法定义混乱，保证一个标准的构造函数。 例如： SA(){ } void SA(){ }

NM_VERY_CONFUSING
混乱的方法命名，如getName和getname方法同时出现的时候

NM_WRONG_PACKAGE
方法因为取了不同包中的同名的对象而没有正确覆写父类中的同名方法

import alpha.Foo;

public class A {

  public int f(Foo x) { return 17; }

}

----

import beta.Foo;

public class B extends A {

  public int f(Foo x) { return 42; }

}

QBA_QUESTIONABLE_BOOLEAN_ASSIGNMENT
再if或者while表达式中使用boolean类型的值时应该使用==去判断，而不是采用=操作

RC_REF_COMPARISON
比较两个对象值是否相等时应该采用equals方法，而不是==方法

RE_BAD_SYNTAX_FOR_REGULAR_EXPRESSION
对正则表达式使用了错误的语法，会抛出未经检查的异常，表明正则表达式模式中的语法错误。

RE_CANT_USE_FILE_SEPARATOR_AS_REGULAR_EXPRESSION
使用正则表达式使用了错误的文件分隔符，在windows系统中正则表达式不会匹配’\’而应该使用'\\'

RV_01_TO_INT
从0到1随机值被强制为整数值0。在强制得到一个整数之前，你可能想得到多个随机值。或使用Random.nextInt（n）的方法。

RV_ABSOLUTE_VALUE_OF_HASHCODE
此代码生成一个哈希码，然后计算该哈希码的绝对值。如果哈希码是Integer.MIN_VALUE的，那么结果将是负数（因为Math.abs（Integer.MIN_VALUE的）== Integer.MIN_VALUE的）。

在2^ 32值之外字符串有一个Integer.MIN_VALUE的hashCode包括“polygenelubricants”，“GydZG_”和“，”DESIGNING WORKHOUSES “。

RV_ABSOLUTE_VALUE_OF_RANDOM_INT
此代码生成一个随机的符号整数，然后计算该随机整数的绝对值。如果随机数生成数绝对值为Integer.MIN_VALUE的，那么结果将是负数（因为Math.abs（Integer.MIN_VALUE的）== Integer.MIN_VALUE的）。

RV_EXCEPTION_NOT_THROWN
此代码创建一个异常（或错误）的对象，但不会用它做任何事情。例如：if (x < 0)

  new IllegalArgumentException("x must be nonnegative");

这可能是程序员的意图抛出创建的异常：

if (x < 0)

  throw new IllegalArgumentException("x must be nonnegative");

86.RV: Method ignores return value (RV_RETURN_VALUE_IGNORED)
该方法的返回值应该进行检查。这种警告通常出现在调用一个不可变对象的方法，认为它更新了对象的值。例如：String dateString = getHeaderField(name);

dateString.trim();

程序员似乎以为trim（）方法将更新dateString引用的字符串。但由于字符串是不可改变的，trim（）函数返回一个新字符串值，在这里它是被忽略了。该代码应更正：

String dateString = getHeaderField(name);

dateString = dateString.trim();

RpC_REPEATED_CONDITIONAL_TEST
该代码包含对同一个条件试验了两次，两边完全一样例如：（如X == 0 | | x == 0）。可能第二次出现是打算判断别的不同条件（如X == 0 | | y== 0）。

SA_FIELD_DOUBLE_ASSIGNMENT
方法中的字段包含了双重任务，例如： 

 int x;

  public void foo() {

   x = x = 17;

  }

这种为变量赋值是无用的，并可能表明一个逻辑错误或拼写错误。

SA_FIELD_SELF_ASSIGNMENT
方法中包含自己对自己赋值的字段。例如：

int x;

  public void foo() {

    x = x;

  }

SA_FIELD_SELF_COMPARISON
字段自己进行自比较可能表明错误或逻辑错误。

SA_LOCAL_SELF_COMPARISON
方法中对一个局部变量自身进行比较运算，并可说明错误或逻辑错误。请确保您是比较正确的事情。

SA_LOCAL_SELF_COMPUTATION
此方法对同一变量执行了荒谬的计算（如x&x或x-x）操作。由于计算的性质，这一行动似乎没有意义，并可能表明错误或逻辑错误。

SF_DEAD_STORE_DUE_TO_SWITCH_FALLTHROUGH
在swtich中先前的case值因为swtich执行失败而被覆写，这就像是忘记使用break推出或者没有使用return语句放回先前的值一样。

SF_DEAD_STORE_DUE_TO_SWITCH_FALLTHROUGH_TO_THROW
在swtich中因为出现异常而忽略了对case值的保存。

SIC_THREADLOCAL_DEADLY_EMBRACE
如果是一个静态内部类。实际上，在内部类和当前线程有死锁的可能。由于内部类不是静态的，它保留了对外部类的引用。如果线程包含对一个内部类实例的引用，那么内外实例的实例都可以被获取，这样就不具备垃圾会回收的资格。

SIO_SUPERFLUOUS_INSTANCEOF
在进行instanceof操作时进行没有必要的类型检查

STI_INTERRUPTED_ON_CURRENTTHREAD
此方法调用Thread.currentThread（）调用，只需调用interrupted（）方法。由于interrupted（）是一个静态方法， Thread.interrupted（）更简单易用。

STI_INTERRUPTED_ON_UNKNOWNTHREAD
调用不是当前线程对象的Thread.interrupted()方法，由于interrupted（）方法是静态的，interrupted方法将会调用一个和作者原计划不同的对象。

SE_METHOD_MUST_BE_PRIVATE
这个类实现了Serializable接口，并定义自定义序列化的方法/反序列化。但由于这种方法不能声明为private，将被序列化/反序列化的API忽略掉。

SE_READ_RESOLVE_IS_STATIC
为使readResolve方法得到序列化机制的识别，不能作为一个静态方法来声明。

UMAC_UNCALLABLE_METHOD_OF_ANONYMOUS_CLASS
在匿名类中定义了一个既没有覆写超类中方法也不能直接调用的方法。因为在其他类的方法不能直接引用匿名类声明的方法，似乎这种方法不能被调用，这种方法可能只是没有任何作用的代码，但也可能覆写超类中声明。

UR_UNINIT_READ
此构造方法中使用了一个尚未赋值的字段或属性。 String a; public SA() {

String abc = a;

System.out.println(abc); }

UR_UNINIT_READ_CALLED_FROM_SUPER_CONSTRUCTOR
方法被超类的构造函数调用时，在当前类中的字段或属性还没有被初始化。例如：

abstract class A {

  int hashCode;

  abstract Object getValue();

  A() {

    hashCode = getValue().hashCode();

    }

  }

class B extends A {

  Object value;

  B(Object v) {

    this.value = v;

    }

  Object getValue() {

    return value;

  }

  }

当B是创建时，A的构造函数将在B为value赋值之前触发，然而在A的初始化方法调用getValue方法时value这个变量还没有被初始化。

DMI_INVOKING_TOSTRING_ON_ANONYMOUS_ARRAY
该代码调用上匿名数组的toString（）方法，产生的结果形如[@ 16f0472并没有实际的意义。考虑使用Arrays.toString方法来转换成可读的字符串，提供该数组的内容数组。例如：

String[] a = { "a" };

System.out.println(a.toString());

//正确的使用为

System.out.println(Arrays.toString(a));

DMI_INVOKING_TOSTRING_ON_ARRAY
该代码调用上数组的toString（）方法，产生的结果形如[@ 16f0472并不能显示数组的真实内容。考虑使用Arrays.toString方法来转换成可读的字符串，提供该数组的内容数组

UWF_NULL_FIELD
字段的值总是为null值，所有读取该字段的值都为null。检查错误，如果它确实没有用就删除掉。

107.UwF: Unwritten field (UWF_UNWRITTEN_FIELD)
此字段是永远不会写入值。所有读取将返回默认值。检查错误（如果它被初始化？），如果它确实没有用就删除掉。

五：Performance关于代码性能相关方面的
BX_UNBOXING_IMMEDIATELY_REBOXED
 

装箱的值被取消装箱，然后立即重新装箱

 

BX_BOXING_IMMEDIATELY_UNBOXED
 

对原始值进行装箱，然后立即取消装箱。这可能是在一个未要求装箱的地方进行了手动装箱，从而迫使编译器进行立即撤消装箱的操作

 

BX_BOXING_IMMEDIATELY_UNBOXED_TO_PERFORM_COERCION
 

对原始值进行装箱然后立即把它强制转换为另外一种原始类型。例如：

new Double(d).intValue()应该直接进行强制转换例如：(int) d

 

DM_BOXED_PRIMITIVE_TOSTRING
 

仅仅为了调用封装类的toString()而对原始类型进行封装操作。比这种方法更有效的是调用封装类的toString(…)方法例如：

new Integer(1).toString()    替换为   Integer.toString(1)

new Long(1).toString()    替换为   Long.toString(1) 

new Float(1.0).toString()    替换为   Float.toString(1.0) 

new Double(1.0).toString()    替换为   Double.toString(1.0) 

new Byte(1).toString()    替换为   Byte.toString(1) 

new Short(1).toString()    替换为   Short.toString(1) 

new Boolean(true).toString()    替换为   Boolean.toString(true)

 

DM_FP_NUMBER_CTOR
 

使用new Double(double)方法总是会创建一个新的对象，然而使用Double.valueOf(double)方法可以把值保存在编辑器或者class library、JVM中。使用存储值的方式来避免对象的分配可以或得更好的代码性能

除非类必须符合Java 1.5以前的JVM，否则请使用自动装箱或valueOf（）方法创建Double和Float实例。

 

DM_NUMBER_CTOR
 

使用new Integer(int)方法总是会创建一个新的对象，然而使用Integer.valueOf(int)方法可以把值保存在编辑器或者class library、JVM中。使用存储值的方式来避免对象的分配可以或得更好的代码性能

除非类必须符合Java 1.5以前的JVM，否则请使用自动装箱或valueOf（）方法创建Long, Integer, Short, Character, Byte实例。

 

DMI_BLOCKING_METHODS_ON_URL
 

使用equals和hashCode方法来对url进行资源标识符解析时会引起堵塞。考虑使用java.net.URI来代替。

 

DMI_COLLECTION_OF_URLS
 

方法或者字段使用url的map/set集合。因为equals方法或者hashCode方法来进行资源标识符解析时都会引起堵塞。考虑使用java.net.URI来代替。

 

DM_BOOLEAN_CTOR
 

使用new方法创建一个java.lang.Boolean类型能够的实例对象是浪费空间的，因为Boolean对象是不可变的而且只有两个有用的值。使用Boolean.valueOf()或者Java1.5中的自动装箱功能来创建一个Boolean实例。

DM_GC
 

在代码中显式的调用垃圾回收命名，这样做并不能起作用。在过去，有人在关闭操作或者finalize方法中调用垃圾回收方法导致了很多的性能浪费。这样大规模回收对象时会造成处理器运行缓慢。

 

DM_NEXTINT_VIA_NEXTDOUBLE
 

如果r是一个java.util.Random对象，你可以使r.nextInt(n)生成一个0到n-1之前的随机数，而不是使用(int)(r.nextDouble() * n)

 

DM_STRING_CTOR
 

使用java.lang.String(String)构造函数会浪费内存因为这种构造方式和String作为参数在功能上容易混乱。只是使用String直接作为参数的形式

 

DM_STRING_TOSTRING
 

调用String.toString()是多余的操作，只要使用String就可以了。

 

DM_STRING_VOID_CTOR
 

使用没有参数的构造方法去创建新的String对象是浪费内存空间的，因为这样创建会和空字符串“”混淆。Java中保证完成相同的构造方法会产生描绘相同的String对象。所以你只要使用空字符串来创建就可以了。

 

ITA_INEFFICIENT_TO_ARRAY
 

当使用集合的toArray()方法时使用数组长度为0的数组作为参数。比这更有效的一种方法是

myCollection.toArray(new Foo[myCollection.size()])，如果数组的长度足够大就可以直接把集合中的内容包装到数组中直接返回从而避免了第二次创建一个新的数组来存放集合中值。

 

LO_APPENDED_STRING_IN_FORMAT_STRING
 

此方法使用SLF4J记录器记录一个字符串，其中第一个（格式）字符串是使用串联创建的。您应该使用{}标记将动态内容注入到字符串中，以便延迟String的构建，直到需要实际的日志字符串为止。如果日志级别足够高，以致不使用此日志语句，则将永远不会执行附加操作。

 

NAB_NEEDLESS_BOXING_VALUEOF
 

此方法将String传递给包装的原始对象的parse方法，该方法又调用valueOf（）方法以转换为装箱的原始对象。当需要从String转换为装箱的原始对象时，使用BoxedPrimitive.valueOf（String）方法更简单。

而不是这样的：

Boolean bo = Boolean.valueOf(Boolean.parseBoolean("true"));

Float f = Float.valueOf(Float.parseFloat("1.234"));

只需做：

 

Boolean bo = Boolean.valueOf("true");

Float f = Float.valueOf("1.234");

 

NAB_NEEDLESS_BOXING_PARSE
 

该方法将String传递给包装的原始对象的valueOf方法，该方法进而调用boxedValue（）方法以转换为原始对象。当需要从String转换为原始值时，使用BoxedPrimitive.parseBoxedPrimitive（String）方法更简单。

而不是这样的：

 

public int someMethod(String data) {

long l = Long.valueOf(data).longValue();

float f = Float.valueOf(data).floatValue();

return Integer.valueOf(data); // There is an implicit .intValue() call

}

只需做：

 

public int someMethod(String data) {

long l = Long.parseLong(data);

float f = Float.parseFloat(data);

return Integer.parseInt(data);

}

 

NAB_NEEDLESS_BOOLEAN_CONSTANT_CONVERSION
 

此方法将一个装箱的布尔常量分配给一个原始布尔变量，或将一个装箱的布尔常量分配给一个装箱的布尔变量。对需要的变量使用正确的常数

 

PRMC_POSSIBLY_REDUNDANT_METHOD_CALLS
 

此方法在相同实例上使用相同的常量参数连续两次调用相同的方法，而无需对对象进行任何中间更改。如果此方法未对对象进行更改（看上去没有更改），则进行两次调用将很浪费。通过将结果分配给一个临时变量，然后第二次使用该变量，可以合并这些方法调用。

 

PSC_PRESIZE_COLLECTIONS
 

这个方法使用默认构造函数分配集合，即使它预先知道集合中将放置多少项(或者至少可以合理地猜测)，从而不必要地导致集合的中间重新分配。

您可以使用具有初始大小的构造函数，这样会好得多，但是由于映射和集合的加载因子，甚至这也不是一个正确的估计值。

如果您正在使用Guava，请使用它的方法来分配具有预先确定大小的映射和集合，以获得不重新分配的最佳机会，例如:

Sets.newHashSetWithExpectedSize (int)

Maps.newHashMapWithExpectedSize (int)

如果没有，一个很好的估计值应该是expectedSize / {LOADING_FACTOR}默认值为0.75

 

SBSC_USE_STRINGBUFFER_CONCATENATION
 

在循环中构建一个String对象时从性能上讲使用StringBuffer来代替String对象

例如：

// This is bad

  String s = "";

  for (int i = 0; i < field.length; ++i) {

    s = s + field[i];

  }

 

 

  // This is better

  StringBuffer buf = new StringBuffer();

  for (int i = 0; i < field.length; ++i) {

    buf.append(field[i]);

  }

  String s = buf.toString();

 

SEO_SUBOPTIMAL_EXPRESSION_ORDER
 

例如，此方法在if或while语句中构建条件表达式，该表达式既包含简单的局部变量比较，又包含方法调用的比较。表达式对这些命令进行排序，以便在简单的局部变量比较之前进行方法调用。这导致方法调用在不需要的条件下执行，因此有可能导致大量代码无执行。通过对表达式进行排序，以便首先包含局部变量条件的简单条件，可以消除这种浪费。假定方法调用没有副作用。如果该方法确实有副作用，则最好将这些调用从条件中拉出并先执行，然后将值分配给局部变量。

例：

if ((calculateHaltingProbability() > 0) && shouldCalcHalting) { }

Better：

if (shouldCalcHalting && (calculateHaltingProbability() > 0) { }

 

SS_SHOULD_BE_STATIC
 

类中所包含的final属性字段在编译器中初始化为静态的值。考虑在定义时就把它定义为static类型的。

 

SPP_STRINGBUFFER_WITH_EMPTY_STRING
 

这个方法调用StringBuffer或StringBuilder构造函数，传入一个常量空字符串("")。这与调用默认构造函数相同，但是会使代码更加困难。考虑传入一个默认大小。

 

UCPM_USE_CHARACTER_PARAMETERIZED_METHOD
 

此方法将String长度为1 的常量文字作为参数传递给方法，该方法公开了一个类似的方法，该方法采用char。处理一个字符而不是一个字符串更简单方便。

而不是像这样：

String myString = ...

if (myString.indexOf("e") != -1) {

    int i = myString.lastIndexOf("e");

    System.out.println(myString + ":" + i);  //the Java compiler will use a StringBuilder internally here [builder.append(":")]

    ...

    return myString.replace("m","z");

}

将单字母Strings 替换为它们的char等价物，如下所示：

String myString = ...

if (myString.indexOf('e') != -1) {

    int i = myString.lastIndexOf('e');

    System.out.println(myString + ':' + i);  //the Java compiler will use a StringBuilder internally here [builder.append(':')]

    ...

    return myString.replace('m','z');

}

 

UM_UNNECESSARY_MATH
 

在方法中使用了java.lang.Math的静态方法代替常量来使用，使用常量速度和准确度会更好。 以下为Math中的方法产生的值。

Method Parameter 

abs -any- 

acos 0.0 or 1.0 

asin 0.0 or 1.0 

atan 0.0 or 1.0 

atan2 0.0 cbrt 0.0 or 1.0 

ceil -any- 

cos 0.0 

cosh 0.0 

exp 0.0 or 1.0 

expm1 0.0 

floor -any- 

log 0.0 or 1.0 

log10 0.0 or 1.0 

rint -any- 

round -any- 

sin 0.0 

sinh 0.0 

sqrt 0.0 or 1.0 

tan 0.0 

tanh 0.0 

toDegrees 0.0 or 1.0 

toRadians 0.0

 

UPM_UNCALLED_PRIVATE_METHOD
 

定义为Private类型方法从未被调用，应该被删除。

 

URF_UNREAD_FIELD
 

类中定义的属性从未被调用，建议删除。

 

UUF_UNUSED_FIELD
 

类中定义的属性从未被使用，建议删除。

 

WMI_WRONG_MAP_ITERATOR
 

当方法中接受一个Map类型的参数时，使用keySet的迭代器比使用entrySet的迭代器效率要高。

六：Internationalization关于代码国际化相关方面的
DM_CONVERT_CASE
 

使用平台默认的编码格式对字符串进行大小写转换，这可能导致国际字符的转换不当。使用以下方式对字符进行转换

String.toUpperCase( Locale l )

String.toLowerCase( Locale l )

七：Multithreaded correctness关于代码多线程正确性相关方面的
DL_SYNCHRONIZATION_ON_BOOLEAN
 

该代码同步一个封装的原始常量，例如一个Boolean类型。

private static Boolean inited = Boolean.FALSE;

...

  synchronized(inited) { 

    if (!inited) {

       init();

       inited = Boolean.TRUE;

       }

     }

...

由于通常只存在两个布尔对象，此代码可能是同步的其他无关的代码中相同的对象，这时会导致反应迟钝和可能死锁

 

DL_SYNCHRONIZATION_ON_BOXED_PRIMITIVE
 

该代码同步一个封装的原始常量，例如一个Integer类型。

private static Integer count = 0;

...

  synchronized(count) { 

     count++;

     }

...

由于Integer对象可以共享和保存，此代码可能是同步的其他无关的代码中相同的对象，这时会导致反应迟钝和可能死锁

 

DL_SYNCHRONIZATION_ON_SHARED_CONSTANT
 

同步String类型的常量时，由于它被JVM中多个其他的对象所共有，这样在其他代码中会引起死锁。

 

DL_SYNCHRONIZATION_ON_UNSHARED_BOXED_PRIMITIVE
 

同步一个显然不是共有封装的原始值，例如一个Integer类型的对象。例如：

private static final Integer fileLock = new Integer(1);

...

  synchronized(fileLock) { 

     .. do something ..

     }

...

它最后被定义为以下方式来代替：private static final Object fileLock = new Object();

 

 

DM_MONITOR_WAIT_ON_CONDITION
 

方法中以java.util.concurrent.locks.Condition对象调用wait（）。等待一个条件发生时应该使用在Condition接口中定义的await()方法。

 

DM_USELESS_THREAD
 

这个方法没有通过run方法或者具体声明Thread类，也没有通过一个Runnable对象去定义一个线程，而这个线程出来浪费资源却什么也没有去做。

 

ESync_EMPTY_SYNC
 

该代码包含一个空的同步块：synchronized() {}

 

IS2_INCONSISTENT_SYNC
 

不合理的同步

 

IS_FIELD_NOT_GUARDED
 

域不是良好的同步访问---

此字段被标注为net.jcip.annotations.GuardedBy，但可以在某种程度上违反注释而去访问

 

JLM_JSR166_LOCK_MONITORENTER
 

实现java.util.concurrent.locks.Lock的对象调用了同步的方法。应该这样处理，对象被锁定/解锁时使用acquire（）/ release（）方法而不是使用同步的方法。

LI_LAZY_INIT_STATIC
 

静态域不正确的延迟初始化--

这种方法包含了一个不同步延迟初始化的非volatile静态字段。因为编译器或处理器可能会重新排列指令，如果该方法可以被多个线程调用，线程不能保证看到一个完全初始化的对象。你可以让字段可变来解决此问题

 

LI_LAZY_INIT_UPDATE_STATIC
 

这种方法包含一个不同步延迟初始化的静态字段。之后为字段赋值，对象存储到该位置后进一步更新或访问。字段后尽快让其他线程能够访问。如果该方法的进一步访问该字段为初始化对象提供服务，然后你有一个非常严重的多线程bug，除非别的东西阻止任何其他线程访问存储的对象，直到它完全初始化。

即使你有信心，该方法是永远不会被多个线程调用时，在它的值还没有被充分初始化或移动，不把它设定为static字段时它可能会更好。

 

ML_SYNC_ON_UPDATED_FIELD
 

对象获取一个可变字段时进行同步。这是没有意义的，因为不同的线程可以在不同的对象同步。

 

MSF_MUTABLE_SERVLET_FIELD
 

一个web服务一般只能创建一个servlet或者jsp的实例（例如：treates是一个单利类），它会被多个线程调用这个实例的方法服务于多个同时的请求。因此使用易变的字段属性产生竞争的情况。

 

MWN_MISMATCHED_NOTIFY
 

此方法调用Object.notify（）或Object.notifyAll（）而没有获取到该对象的对象锁。调用notify（）或notifyAll（）而没有持有该对象的对象锁，将导致IllegalMonitorStateException异常。

 

MWN_MISMATCHED_WAIT
 

此方法调用Object.wait()而没有获取到该对象的对象锁。调用wait（）而没有持有该对象的对象锁，将导致IllegalMonitorStateException异常。

 

NP_SYNC_AND_NULL_CHECK_FIELD
 

如果代码块是同步的，那么久不可能为空。如果是空，同步时就会抛出NullPointerException异常。最好是在另一个代码块中进行同步。

 

NO_NOTIFY_NOT_NOTIFYALL
 

调用notify（）而不是notifyAll（）方法。 Java的监控器通常用于多个条件。调用notify（）只唤醒一个线程，这意味着该线程被唤醒只是满足的当前的唯一条件。

 

RS_READOBJECT_SYNC
 

序列化类中定义了同步的readObject（）。通过定义，反序列化创建的对象只有一个线程可以访问，因此没有必要的readObject（）进行同步。如果的readObject（）方法本身造成对象对另一个线程可见，那么这本身就是不好的编码方式。

 

RU_INVOKE_RUN
 

这种方法显式调用一个对象的run（）。一般来说，类是实现Runnable接口的，因为在一个新的线程他们将有自己的run（）方法，在这种情况下Thread.start（）方法调用是正确的。

 

SC_START_IN_CTOR
 

在构造函数中启动一个线程。如果类曾经被子类扩展过，那么这很可能是错的，因为线程将在子类构造之前开始启动。

 

SP_SPIN_ON_FIELD
 

方法无限循环读取一个字段。编译器可合法悬挂宣读循环，变成一个无限循环的代码。这个类应该改变，所以使用适当的同步（包括等待和通知要求）

 

STCAL_INVOKE_ON_STATIC_CALENDAR_INSTANCE
 

即使JavaDoc对此不包含暗示，而Calendars本身在多线程中使用就是不安全的。探测器发现当调用Calendars的实例时将会获得一个静态对象。

Calendar rightNow = Calendar.getInstance();

 

STCAL_INVOKE_ON_STATIC_DATE_FORMAT_INSTANCE
 

在官方的JavaDoc，DateFormats多线程使用本事就是不安全的。探测器发现调用一个DateFormat的实例将会获得一个静态对象。

myString = DateFormat.getDateInstance().format(myDate);

 

STCAL_STATIC_CALENDAR_INSTANCE
 

Calendar在多线程中本身就是不安全的，如果在线程范围中共享一个Calendarde 实例而不使用一个同步的方法在应用中就会出现一些奇怪的行为。在sun.util.calendar.BaseCalendar.getCalendarDateFromFixedDate()中会抛出ArrayIndexOutOfBoundsExceptions or IndexOutOfBoundsExceptions异常。

 

STCAL_STATIC_SIMPLE_DATE_FORMAT_INSTANCE
 

DateFormat 在多线程中本身就是不安全的，如果在线程范围中共享一个DateFormat的实例而不使用一个同步的方法在应用中就会出现一些奇怪的行为。

 

SWL_SLEEP_WITH_LOCK_HELD
 

当持有对象时调用Thread.sleep（）。这可能会导致很差的性能和可扩展性，或陷入死锁，因为其他线程可能正在等待获得锁。调用wait（）是一个更好的主意，释放对象的持有以允许其他线程运行。

 

UG_SYNC_SET_UNSYNC_GET
 

这个类包含类似命名的get和set方法。在set方法是同步方法和get方法是非同步方法。这可能会导致在运行时的不正确行为，因为调用的get方法不一定返回对象一致状态。 GET方法应该同步。

 

UL_UNRELEASED_LOCK
 

方法获得了当前的对象所，但是在方法中始终没有释放它。一个正确的示例如下：

  Lock l = ...;

    l.lock();

    try {

        // do something

    } finally {

        l.unlock();

}

 

UL_UNRELEASED_LOCK_EXCEPTION_PATH
 

方法获得了当前的对象所，但是在所有的异常处理中始终没有释放它。一个正确的示例如下：

  Lock l = ...;

    l.lock();

    try {

        // do something

    } finally {

        l.unlock();

}

 

UW_UNCOND_WAIT)
 

方法中包含调用java.lang.Object.wait（），而却没有放到条件流程控制中。该代码应确认条件尚未满足之前等待;先前任何通知将被忽略。

 

VO_VOLATILE_REFERENCE_TO_ARRAY
 

声明一个变量引用数组，这可能不是你想要的。如果一个变量引用数组，那么对引用数组的读和写都是不安全的，但是数组元素不是变量。取得数组的变量值你可以使用java.util.concurrent包中的数组的原子性特性

 

WL_USING_GETCLASS_RATHER_THAN_CLASS_LITERAL
 

实例的方法中同步this.getClass()，如果这个类有子类集合，那么子类集合中的对象将会在这个类的各个子类上进行同步，这不是我们想要的效果，我们只要同步当前的类对象而不包含它的所有子类，可以同步类名.getClass()。例如，java.awt.Label的代码：

     private static final String base = "label";

     private static int nameCounter = 0;

     String constructComponentName() {

        synchronized (getClass()) {

            return base + nameCounter++;

        }

     }

Label中的子类集合不可能在同一个子对象上进行同步，替换上面的方法为：

    private static final String base = "label";

     private static int nameCounter = 0;

     String constructComponentName() {

        synchronized (Label.class) {

            return base + nameCounter++;

        }

     }

 

WS_WRITEOBJECT_SYNC
 

这个类有一个writeObject（）方法是同步的，但是这个类中没有其他的同步方法。

 

WA_AWAIT_NOT_IN_LOOP
 

方法没有在循环中调用java.util.concurrent.await()。如果对象是用于多种条件，打算调用wait()方法的条件可能不是实际发生的。

 

WA_NOT_IN_LOOP
 

这种方法包含调用java.lang.Object.wait（），而这并不是一个循环。如果监视器用于多个条件，打算调用wait()方法的条件可能不是实际发生的。

 

八：Malicious codevulnerability关于恶意破坏代码相关方面的
EI_EXPOSE_REP
 

返回一个易变对象引用并把它保存在对象字段中时会暴露对象内部的字段描述，如果接受不守信任的代码访问或者没有检查就去改变易变对象的会涉及对象的安全和其他重要属性的安全。返回一个对象的新副本，在很多情况下更好的办法。

 

EI_EXPOSE_REP2
 

此代码把外部可变对象引用存储到对象的内部表示。如果实例受到不信任的代码的访问和没有检查的变化危及对象和重要属性的安全。存储一个对象的副本，在很多情况下是更好的办法。

 

FI_PUBLIC_SHOULD_BE_PROTECTED
 

一个类中的finalize（）方法必须声明为protected，而不能为public类型

 

MS_EXPOSE_REP
 

一个public类型的静态方法返回一个数组，可能引用内部属性的暴露。任何代码调用此方法都可以自由修改底层数组。一个解决办法是返回一个数组的副本。

 

MS_FINAL_PKGPROTECT
 

一个静态字段可能被恶意代码或另外一个包所改变的。字段可以放到protected包中也可以定义为final类型的以避免此问题。

 

MS_MUTABLE_ARRAY
 

一个定义为final类型的静态字段引用一个数组时它可以被恶意代码或在另其他包中所使用。这些代码可以自由修改数组的内容。

 

MS_MUTABLE_HASHTABLE
 

一个定义为final类型的静态字段引用一个Hashtable时可以被恶意代码或者在其他包中被调用，这些方法可以修改Hashtable的值。

 

MS_OOI_PKGPROTECT
 

将域尽量不要定义在接口中，并声明为包保护

在接口中定义了一个final类型的静态字段，如数组或哈希表等易变对象。这些对象可以被恶意代码或者在其他包中被调用，为了解决这个问题，需要把它定义到一个具体的实体类中并且声明为保护类型以避免这种错误。

 

MS_PKGPROTECT
 

一个静态字段是可以改变的恶意代码或其他的包访问修改。可以把这种类型的字段声明为final类型的以防止这种错误。

 

九：Dodgy、style糟糕的代码
 

BC_BAD_CAST_TO_ABSTRACT_COLLECTION
 

在代码投把一个集合强制类型转换为一个抽象的集合（如list，set或map）。保证该对象类型和将要转换的类型是一致的。如果你只是想要便利一个集合，那么你就不必将它转换为Set或List。

 

BC_BAD_CAST_TO_CONCRETE_COLLECTION
 

代码把抽象的集合（如List，Set，或Collection）强制转换为具体落实类型（如一个ArrayList或HashSet）。这可能不正确，也可能使您的代码很脆弱，因为它使得难以在今后的切换指向其他具体实现。除非你有特别理由这样做，否则只需要使用抽象的集合类。

 

BC_UNCONFIRMED_CAST
 

强制类型转换操作没有经过验证，而且不是所有的此种类型装换过的类都可以再强制类型转换为原类型。在代码中需要进行逻辑判断以保证可以进行这样的操作。

 

BC_VACUOUS_INSTANCEOF
 

instanceof测试将始终返回真（除非被测试的值为空）。虽然这是安全，确保它是不是说明一些误解或其他一些逻辑错误。如果你真的想测试是空的价值，也许会更清楚这样做的更好空试验，而不是一个instanceof测试。

 

CC_CYCLOMATIC_COMPLEXITY
 

该方法具有较高的圈复杂度，可以计算出分支点的个数。它很可能难以测试，并且很容易更改。考虑将此方法重构为多个方法以降低风险。

 

CFS_CONFUSING_FUNCTION_SEMANTICS
 

此方法将修改参数，然后将此参数作为方法的返回值返回。这将使这个方法的调用者感到困惑，因为传入参数的“原始”显然也不会更改。如果此方法的目的是更改参数，则将该方法更改为具有空返回值的a会更清楚。如果由于接口或超类契约需要返回类型，则可能应该复制该参数。

 

CI_CONFUSED_INHERITANCE
 

这个类被声明为final的，而是字段属性却声明为保护类型的。由于是final类，它不能再被继承，而再声明为保护类型的很容易造成混淆。为了从外部能正确的使用它应该把它们声明为private或者public类型。

 

DB_DUPLICATE_BRANCHES
 

此方法使用相同的代码，以实现两个有条件的分支。检查以确保这是不是一个编码错误。

 

DB_DUPLICATE_SWITCH_CLAUSES
 

他的方法使用相同的代码来实现两个switch的声明条款。这可能是重复代码的情况，但可能也显示出编码的错误。

 

DLS_DEAD_LOCAL_STORE
 

该指令为局部变量赋值，但在其后的没有对她做任何使用。通常，这表明一个错误，因为值从未使用过。

 

DLS_DEAD_LOCAL_STORE_IN_RETURN
 

本声明把一个局部变量放到方法的返回语句中。这对于方法中局部变量来说是没有意思的。

 

DLS_DEAD_LOCAL_STORE_OF_NULL
 

把一个本地变量赋值为null值，并且再也没有对这个变量做任何的操作。这样可能是为了垃圾回收，而是Java SE 6.0，这已不再需要。

 

DMI_HARDCODED_ABSOLUTE_FILENAME
 

此代码包含文件对象为一个绝对路径名（例如，新的文件（“/ home/dannyc/workspace/j2ee/src/share/com/sun/enterprise/deployment”）;

 

DMI_NONSERIALIZABLE_OBJECT_WRITTEN
 

代码中让一个非序列化的对象出现在ObjectOutput.writeObject()方法中，这样会引起一个错误。

 

DMI_USELESS_SUBSTRING
 

此代码调用了subString(0)方法，它将返回原来的值。

 

EXS_EXCEPTION_SOFTENING_HAS_CHECKED
 

此方法的异常签名受超类接口的约束，不抛出已捕获的已检查异常。因此，此异常被转换为未检查异常并引发。最好抛出最近的已检查异常，并使用初始原因字段用原始异常注释新异常。

 

 

EQ_DOESNT_OVERRIDE_EQUALS
 

该类扩展了定义equals方法的类并添加字段，但本身不定义equals方法。因此，该类实例上的等式将忽略子类的标识和添加的字段。确保这是您想要的，并且您不需要覆盖equals方法。即使不需要重写equals方法，也可以考虑重写它，以记录这样一个事实:子类的equals方法只返回调用super.equals(o)的结果。

FE_FLOATING_POINT_EQUALITY
 

此操作比较两个浮点值是否相等。由于浮点运算可能会涉及到舍入，计算float和double值可能不准确。如果要求值必须准确，如货币值，可以考虑使用固定精度类型，如BigDecimal类型的值来比较

 

VA_FORMAT_STRING_BAD_CONVERSION_TO_BOOLEAN
 

使用%b去格式化Boolean类型的值不正确的但是它不会抛出异常，任何非空的值都会输出true，任何为空的值都会输出false

 

IC_INIT_CIRCULARITY
 

在引用两个相互调用为环状static方法去初始化一个实例时是错误的。

 

ICAST_IDIV_CAST_TO_DOUBLE
 

整形数除法强制转换为double或者float类型。

int x = 2;

int y = 5;

// Wrong: yields result 0.0

double value1 =  x / y;

// Right: yields result 0.4

double value2 =  x / (double) y;

 

ICAST_INTEGER_MULTIPLY_CAST_TO_LONG
 

整形数做乘法运算结果转换为long值时如果采用

long convertDaysToMilliseconds(int days) { return 1000*3600*24*days; } 结果会因为超出整形的范围而出错。

如果使用：

long convertDaysToMilliseconds(int days) { return 1000L*3600*24*days; } 

或者：

static final long MILLISECONDS_PER_DAY = 24L*3600*1000; long convertDaysToMilliseconds(int days) { return days * MILLISECONDS_PER_DAY; } 

都可以避免此问题。

 

ICAST_QUESTIONABLE_UNSIGNED_RIGHT_SHIFT
 

无符号数右移后进行转换为short或者byte类型时可能会丢弃掉高位的值，这样的结果就是有符合数和无符号数无法区分（这取决于移位大小）

 

IM_AVERAGE_COMPUTATION_COULD_OVERFLOW
 

代码中使用x % 2 == 1的方法去验证运算是否存在余数的情况，但是如果出现负数的情况就不起作用了。使用x & 1 == 1, or x % 2 != 0来代替

 

IMC_IMMATURE_CLASS_PRINTSTACKTRACE
 

此方法将堆栈跟踪打印到控制台。这是不可配置的，并导致应用程序看起来不专业。切换到使用日志记录器，以便用户能够控制日志记录的内容和位置。

 

INT_VACUOUS_COMPARISON
 

整形数进行比较结果总是不变。例如：x <= Integer.MAX_VALUE

 

MOM_MISLEADING_OVERLOAD_MODEL
 

该类用实例和静态版本“重载”相同的方法。由于这两种模型的使用是不同的，这将使这些方法的用户感到困惑。

 

MTIA_SUSPECT_SERVLET_INSTANCE_FIELD
 

这个类扩展从Servlet类，并使用实例的成员变量。由于只有一个Servlet类的实例，并在多线程方式使用，这种模式有可能存在问题。考虑只使用方法的局部变量。

 

MTIA_SUSPECT_STRUTS_INSTANCE_FIELD
 

类扩展自Struts的Action类并使用这个实例的成员变量，因为在Struts框架中只存在一个Action实例对象并且使用在多线程的情况下很可能会出现问题。

 

NP_DEREFERENCE_OF_READLINE_VALUE
 

对readLine()的结果值没有进行判空操作就去重新赋值，这样的操作可以会抛出空指针异常。

 

NP_IMMEDIATE_DEREFERENCE_OF_READLINE
 

对readLine()的结果立即赋值，这样的操作可以会抛出空指针异常。

 

NP_NULL_ON_SOME_PATH_FROM_RETURN_VALUE
 

方法的返回值没有进行是否为空的检查就重新赋值，这样可能会出现空指针异常。

 

NP_PARAMETER_MUST_BE_NONNULL_BUT_MARKED_AS_NULLABLE
 

参数值在任何情况下都不能为空，但是有明确的注释它可以为空。

 

NS_DANGEROUS_NON_SHORT_CIRCUIT
 

代码中使用（& or |）代替（&& or ||）操作，这会造成潜在的危险。

 

NS_NON_SHORT_CIRCUIT
 

代码中使用（& or |）代替（&& or ||）操作，会引起不安全的操作

 

PZLA_PREFER_ZERO_LENGTH_ARRAYS
 

考虑返回一个零长度的数组，而不是null值

 

QF_QUESTIONABLE_FOR_LOOP
 

确定这个循环是正确的变量递增，看起来，另一个变量被初始化，检查的循环。这是由于for循环中太复杂的定义造成的。

 

RCN_REDUNDANT_COMPARISON_OF_NULL_AND_NONNULL_VALUE
 

方法中包含一个不能为空的赋值还包含一个可以为空的赋值。冗余比较非空值为空。

 

RCN_REDUNDANT_COMPARISON_TWO_NULL_VALUES
 

方法中对两个null值进行比较

 

RCN_REDUNDANT_NULLCHECK_OF_NONNULL_VALUE
 

方法中对不为空的值进行为空的判断。

 

REC_CATCH_EXCEPTION
 

在try/catch块中捕获异常，但是异常没有在try语句中抛出而RuntimeException又没有明确的被捕获

 

RI_REDUNDANT_INTERFACES
 

子类和父类都实现了同一个接口，这种定义是多余的。

 

RV_DONT_JUST_NULL_CHECK_READLINE
 

readLine方法的结果不为空时被抛弃

 

RV_REM_OF_RANDOM_INT
 

此代码生成一个随机的符号整数，然后计算另一个值的。由于随机数可以是负数，所以其余操作的结果也可以是负面的。考虑使用Random.nextInt（int）方法代替。

 

RV_RETURN_VALUE_IGNORED_NO_SIDE_EFFECT
 

这段代码调用一个方法并忽略返回值。然而，我们的分析表明，该方法(如果有的话，包括它在子类中的实现)除了返回值之外不会产生任何效果。因此可以删除这个调用。

可能是错误警告：-该方法旨在被覆盖，并在其他超出分析范围的项目中产生副作用。

-调用该方法来触发类加载，这可能会产生副作用。

-调用该方法只是为了获得一些异常。

可以使用@CheckReturnValue注释来指示FindBugs忽略该方法的返回值

 

SA_LOCAL_DOUBLE_ASSIGNMENT
 

为一个局部变量两次赋值，这样是没有意义的。例如：

public void foo() {

    int x,y;

    x = x = 17;

  }

 

SA_LOCAL_SELF_ASSIGNMENT
 

局部变量使用自身给自己赋值

public void foo() {

    int x = 3;

    x = x;

  }

 

SF_SWITCH_FALLTHROUGH
 

Switch语句中一个分支执行后又执行了下一个分支。通常case后面要跟break 或者return语句来跳出。

 

SF_SWITCH_NO_DEFAULT
 

Switch没有默认情况下执行的case语句。

 

SE_PRIVATE_READ_RESOLVE_NOT_INHERITED
 

声明为private的序列化方法被子类继承

 

UCF_USELESS_CONTROL_FLOW
 

没有任何作用的条件语句。

if (argv.length == 0) { // TODO: handle this case }

 

UCF_USELESS_CONTROL_FLOW_NEXT_LINE
 

无效的条件控制语句，注意if (argv.length == 1);以“;”结尾，下面的语句无论是否满足都会运行。

if (argv.length == 1);

        System.out.println("Hello, " + argv[0]);

 

UP_UNUSED_PARAMETER
 

此方法定义了从未使用过的参数。由于此方法是静态的或私有的，且不能派生，因此删除这些参数并简化方法是安全的。虽然不太可能，但您应该考虑可以反射性地使用此方法，因此您也希望更改该调用。在这种情况下，很可能一旦删除了参数，就会有一系列方法调用花费时间创建该参数并将其传递下去。所有这些都可能被移除。

 

UWF_FIELD_NOT_INITIALIZED_IN_CONSTRUCTOR
 

字段从来没有在任何构造函数初始化，对象被创建后值为空。如果该字段未被定义就重新赋值会产生一个空指针异常。

 

USBR_UNNECESSARY_STORE_BEFORE_RETURN
该方法将返回结果存储在局部变量中，然后立即返回局部变量。直接返回分配给局部变量的值会更简单。

Instead of the following:

public float average(int[] arr) {

    float sum = 0;

    for (int i = 0; i < arr.length; i++) {

        sum += arr[i];

    }

    float ave = sum / arr.length;

    return ave;

}

Simply change the method to return the result of the division:

public float average(int[] arr) {

    float sum = 0;

    for (int i = 0; i < arr.length; i++) {

        sum += arr[i];

    }

    return sum / arr.length; //Change

}

 

XFB_XML_FACTORY_BYPASS
 

方法自定义了一种XML接口的实现类。最好是使用官方提供的工厂类来创建这些对象，以便可以在运行期中改变。例如：

javax.xml.parsers.DocumentBuilderFactory

javax.xml.parsers.SAXParserFactory

javax.xml.transform.TransformerFactory

org.w3c.dom.Document.createXXXX

 

XSS_REQUEST_PARAMETER_TO_SEND_ERROR
 

在代码中在Servlet输出中直接写入一个HTTP参数，这会造成一个跨站点的脚本漏洞。

 

XSS_REQUEST_PARAMETER_TO_SERVLET_WRITER
 

代码直接写入参数的HTTP服务器错误页（使用HttpServletResponse.sendError）。表达了类似的不受信任的输入会引起跨站点脚本漏洞。


OBL_UNSATISFIED_OBLIGATION 方法可能在清理流或资源时失败。

处理方式：使用try/finally块，在finally块中关闭流。

OBL_UNSATISFIED_OBLIGATION_EXCEPTION_EDGE

原因及处理方式：原因是in关闭异常之后，out可能就关闭失败。处理方式先关闭输出流，再关闭输入流。






OS

OS_OPEN_STREAM 方法可能在关闭流时失败

处理方式：使用try/finally块，在finally块中关闭流。



RV

RV_RETURN_VALUE_IGNORED_NO_SIDE_EFFECT 有返回值，但是未使用。

处理方式：接收返回值，并且打印出来。logger.info()；


FindBugs是基于Bug Patterns概念，查找javabytecode（.class文件）中的潜在bug，主要检查bytecode中的bug patterns，如NullPoint空指针检查、没有合理关闭资源、字符串相同判断错（==，而不是equals）等

一、Security 关于代码安全性防护
1.Dm: Hardcoded constant database password (DMI_CONSTANT_DB_PASSWORD)
代码中创建DB的密码时采用了写死的密码。
2.Dm: Empty database password (DMI_EMPTY_DB_PASSWORD)
创建数据库连接时没有为数据库设置密码，这会使数据库没有必要的保护。
3.HRS: HTTP cookie formed from untrusted input (HRS_REQUEST_PARAMETER_TO_COOKIE)
此代码使用不受信任的HTTP参数构造一个HTTP Cookie。
4.HRS: HTTP Response splitting vulnerability (HRS_REQUEST_PARAMETER_TO_HTTP_HEADER)
在代码中直接把一个HTTP的参数写入一个HTTP头文件中，它为HTTP的响应暴露了漏洞。
5.SQL: Nonconstant string passed to execute method on an SQL statement (SQL_NONCONSTANT_STRING_PASSED_TO_EXECUTE)
该方法以字符串的形式来调用SQLstatement的execute方法，它似乎是动态生成SQL语句的方法。这会更容易受到SQL注入攻击。
6.XSS: JSP reflected cross site scripting vulnerability (XSS_REQUEST_PARAMETER_TO_JSP_WRITER)
在代码中在JSP输出中直接写入一个HTTP参数，这会造成一个跨站点的脚本漏洞。
二、Experimental
1.LG: Potential lost logger changes due to weak reference in OpenJDK (LG_LOST_LOGGER_DUE_TO_WEAK_REFERENCE)
OpenJDK的引入了一种潜在的不兼容问题，特别是，java.util.logging.Logger的行为改变时。它现在使用内部弱引用，而不是强引用。–logger配置改变，它就是丢失对logger的引用，这本是一个合理的变化，但不幸的是一些代码对旧的行为有依赖关系。这意味着，当进行垃圾收集时对logger配置将会丢失。例如：
public static void initLogging() throws Exception {
 Logger logger = Logger.getLogger("edu.umd.cs");
 logger.addHandler(new FileHandler()); // call to change logger configuration
 logger.setUseParentHandlers(false); // another call to change logger configuration
}
该方法结束时logger的引用就丢失了，如果你刚刚结束调用initLogging方法后进行垃圾回收，logger的配置将会丢失（因为只有保持记录器弱引用）。
public static void main(String[] args) throws Exception {
 initLogging(); // adds a file handler to the logger
 System.gc(); // logger configuration lost
 Logger.getLogger("edu.umd.cs").info("Some message"); // this isn't logged to the file as expected
}
2.OBL: Method may fail to clean up stream or resource (OBL_UNSATISFIED_OBLIGATION)
这种方法可能无法清除（关闭，处置）一个流，数据库对象，或其他资源需要一个明确的清理行动。
一般来说，如果一个方法打开一个流或其他资源，该方法应该使用try / finally块来确保在方法返回之前流或资源已经被清除了。这种错误模式基本上和OS_OPEN_STREAM和ODR_OPEN_DATABASE_RESOURCE错误模式相同，但是是在不同在静态分析技术。我们正为这个错误模式的效用收集反馈意见。

三、Bad practice代码实现中的一些坏习惯
1.AM: Creates an empty jar file entry (AM_CREATES_EMPTY_JAR_FILE_ENTRY)
调用putNextEntry()方法写入新的 jar 文件条目时立即调用closeEntry()方法。这样会造成JarFile条目为空。
2.AM: Creates an empty zip file entry (AM_CREATES_EMPTY_ZIP_FILE_ENTRY)
调用putNextEntry()方法写入新的 zip 文件条目时立即调用closeEntry()方法。这样会造成ZipFile条目为空。
3.BC: Equals method should not assume anything about the type of its argument (BC_EQUALS_METHOD_SHOULD_WORK_FOR_ALL_OBJECTS)
equals(Object o)方法不能对参数o的类型做任何的假设。比较此对象与指定的对象。当且仅当该参数不为 null，并且是表示与此对象相同的类型的对象时，结果才为 true。
4.BC: Random object created and used only once (DMI_RANDOM_USED_ONLY_ONCE)
随机创建对象只使用过一次就抛弃
5.BIT: Check for sign of bitwise operation (BIT_SIGNED_CHECK)
检查位操作符运行是否合理
((event.detail & SWT.SELECTED) > 0)
If SWT.SELECTED is a negative number, this is a candidate for a bug. Even when SWT.SELECTED is not negative, it seems good practice to use '!= 0' instead of '> 0'.
6.CN: Class implements Cloneable but does not define or use clone method (CN_IDIOM)
按照惯例，实现此接口的类应该使用公共方法重写 Object.clone（它是受保护的），以获得有关重写此方法的详细信息。此接口不 包含 clone 方法。因此，因为某个对象实现了此接口就克隆它是不可能的,应该实现此接口的类应该使用公共方法重写 Object.clone
7.CN: clone method does not call super.clone() (CN_IDIOM_NO_SUPER_CALL)
一个非final类型的类定义了clone()方法而没有调用super.clone()方法。例如：B扩展自A，如果B中clone方法调用了spuer.clone（），而A中的clone没有调用spuer.clone()，就会造成结果类型不准确。要求A的clone方法中调用spuer.clone()方法。
8.CN: Class defines clone() but doesn't implement Cloneable (CN_IMPLEMENTS_CLONE_BUT_NOT_CLONEABLE)
类中定义了clone方法但是它没有实现Cloneable接口
9.Co: Abstract class defines covariant compareTo() method (CO_ABSTRACT_SELF)
抽象类中定义了多个compareTo()方法，正确的是覆写Comparable中的compareTo方法，方法的参数为Object类型，如下例：
int compareTo(T o)  比较此对象与指定对象的顺序。
10.Co: Covariant compareTo() method defined (CO_SELF_NO_OBJECT)
类中定义了多个compareTo()方法，正确的是覆写Comparable中的compareTo方法，方法的参数为Object类型
11.DE: Method might drop exception (DE_MIGHT_DROP)
方法可能抛出异常
12.DE: Method might ignore exception (DE_MIGHT_IGNORE)
方法可能忽略异常
13.DMI: Don't use removeAll to clear a collection (DMI_USING_REMOVEALL_TO_CLEAR_COLLECTION)
不要用removeAll方法去clear一个集合
14.DP: Classloaders should only be created inside doPrivileged block (DP_CREATE_CLASSLOADER_INSIDE_DO_PRIVILEGED)
类加载器只能建立在特殊的方法体内
15.Dm: Method invokes System.exit(...) (DM_EXIT)
在方法中调用System.exit(...)语句，考虑用RuntimeException来代替
16.Dm: Method invokes dangerous method runFinalizersOnExit (DM_RUN_FINALIZERS_ON_EXIT)
在方法中调用了System.runFinalizersOnExit 或者Runtime.runFinalizersOnExit方法，因为这样做是很危险的。
17.ES: Comparison of String parameter using == or != (ES_COMPARING_PARAMETER_STRING_WITH_EQ)
用==或者!=方法去比较String类型的参数
18.ES: Comparison of String objects using == or != (ES_COMPARING_STRINGS_WITH_EQ)
用==或者！=去比较String类型的对象
19.Eq: Abstract class defines covariant equals() method (EQ_ABSTRACT_SELF)
20.Eq: Equals checks for noncompatible operand (EQ_CHECK_FOR_OPERAND_NOT_COMPATIBLE_WITH_THIS)
equals方法检查不一致的操作。两个类根本就是父子关系而去调用equals方法去判读对象是否相等。
public boolean equals(Object o) {
  if (o instanceof Foo)
    return name.equals(((Foo)o).name);
  else if (o instanceof String)
    return name.equals(o);
  else return false;
21.Eq: Class defines compareTo(...) and uses Object.equals() (EQ_COMPARETO_USE_OBJECT_EQUALS)
类中定义了compareTo方法但是继承了Object中的compareTo方法
22.Eq: equals method fails for subtypes (EQ_GETCLASS_AND_CLASS_CONSTANT)
类中的equals方法可能被子类中的方法所破坏，当使用类似于Foo.class == o.getClass()的判断时考虑用this.getClass() == o.getClass()来替换
23.Eq: Covariant equals() method defined (EQ_SELF_NO_OBJECT)
类中定义了多个equals方法。正确的做法是覆写Object中的equals方法，它的参数为Object类型的对象。
24.FI: Empty finalizer should be deleted (FI_EMPTY)
为空的finalizer方法应该删除。一下关于finalizer的内容省略
25.GC: Unchecked type in generic call (GC_UNCHECKED_TYPE_IN_GENERIC_CALL)
This call to a generic collection method passes an argument while compile type Object where a specific type from the generic type parameters is expected. Thus, neither the standard Java type system nor static analysis can provide useful information on whether the object being passed as a parameter is of an appropriate type.
26.HE: Class defines equals() but not hashCode() (HE_EQUALS_NO_HASHCODE)
方法定义了equals方法却没有定义hashCode方法
27.HE: Class defines hashCode() but not equals() (HE_HASHCODE_NO_EQUALS)
 类定义了hashCode方法去没有定义equal方法
28.HE: Class defines equals() and uses Object.hashCode() (HE_EQUALS_USE_HASHCODE)
一个类覆写了equals方法，没有覆写hashCode方法，使用了Object对象的hashCode方法
29.HE: Class inherits equals() and uses Object.hashCode() (HE_INHERITS_EQUALS_USE_HASHCODE)
子类继承了父类的equals方法却使用了Object的hashCode方法
30.IC: Superclass uses subclass during initialization (IC_SUPERCLASS_USES_SUBCLASS_DURING_INITIALIZATION)
子类在父类未初始化之前使用父类对象实例
public class CircularClassInitialization {
	static class InnerClassSingleton extends CircularClassInitialization {
static InnerClassSingleton singleton = new InnerClassSingleton();
	}	
	static CircularClassInitialization foo = InnerClassSingleton.singleton;
}
31.IMSE: Dubious catching of IllegalMonitorStateException (IMSE_DONT_CATCH_IMSE)
捕捉违法的监控状态异常，例如当没有获取到对象锁时使用其wait和notify方法
32.ISC: Needless instantiation of class that only supplies static methods (ISC_INSTANTIATE_STATIC_CLASS)
为使用静态方法而创建一个实例对象。调用静态方法时只需要使用类名+静态方法名就可以了。
33.It: Iterator next() method can't throw NoSuchElementException (IT_NO_SUCH_ELEMENT)
迭代器的next方法不能够抛出NoSuchElementException
34.J2EE: Store of non serializable object into HttpSession (J2EE_STORE_OF_NON_SERIALIZABLE_OBJECT_INTO_SESSION)
在HttpSession对象中保存非连续的对象
35.JCIP: Fields of immutable classes should be final (JCIP_FIELD_ISNT_FINAL_IN_IMMUTABLE_CLASS)
 The class is annotated with net.jcip.annotations.Immutable, and the rules for that annotation require that all fields are final. .
36.NP: Method with Boolean return type returns explicit null (NP_BOOLEAN_RETURN_NULL)
返回值为boolean类型的方法直接返回null，这样会导致空指针异常
37.NP: equals() method does not check for null argument (NP_EQUALS_SHOULD_HANDLE_NULL_ARGUMENT)
变量调用equals方法时没有进行是否为null的判断
38.NP: toString method may return null (NP_TOSTRING_COULD_RETURN_NULL)
toString方法可能返回null
39.Nm: Class names should start with an upper case letter (NM_CLASS_NAMING_CONVENTION)
类的名称以大写字母名称开头
40.Nm: Class is not derived from an Exception, even though it is named as such (NM_CLASS_NOT_EXCEPTION)
类的名称中含有Exception但是却不是一个异常类的子类，这种名称会造成混淆
41.Nm: Confusing method names (NM_CONFUSING)
令人迷惑的方面命名
42.Nm: Field names should start with a lower case letter (NM_FIELD_NAMING_CONVENTION)
非final类型的字段需要遵循驼峰命名原则
43.Nm: Use of identifier that is a keyword in later versions of Java (NM_FUTURE_KEYWORD_USED_AS_IDENTIFIER)
验证是否是java预留关键字
44.Nm: Use of identifier that is a keyword in later versions of Java (NM_FUTURE_KEYWORD_USED_AS_MEMBER_IDENTIFIER)
验证是否时java中的关键字
45.Nm: Method names should start with a lower case letter (NM_METHOD_NAMING_CONVENTION)
方法名称以小写字母开头
46.Nm: Class names shouldn't shadow simple name of implemented interface (NM_SAME_SIMPLE_NAME_AS_INTERFACE)
实现同一接口实现类不能使用相同的名称，即使它们位于不同的包中
47.Nm: Class names shouldn't shadow simple name of superclass (NM_SAME_SIMPLE_NAME_AS_SUPERCLASS)
继承同一父类的子类不能使用相同的名称，即使它们位于不同的包中
48.Nm: Very confusing method names (but perhaps intentional) (NM_VERY_CONFUSING_INTENTIONAL)
很容易混淆的方法命名，例如方法的名称名称使用使用大小写来区别两个不同的方法。
49.Nm: Method doesn't override method in superclass due to wrong package for parameter (NM_WRONG_PACKAGE_INTENTIONAL)
由于错误引用了不同包中相同类名的对象而不能够正确的覆写父类中的方法
import alpha.Foo;
public class A {
  public int f(Foo x) { return 17; }
}
import beta.Foo;
public class B extends A {
  public int f(Foo x) { return 42; }
  public int f(alpha.Foo x) { return 27; }
}
50.ODR: Method may fail to close database resource (ODR_OPEN_DATABASE_RESOURCE)
方法中可能存在关闭数据连接失败的情况
51.OS: Method may fail to close stream (OS_OPEN_STREAM)
方法中可能存在关闭流失败的情况
52.OS: Method may fail to close stream on exception (OS_OPEN_STREAM_EXCEPTION_PATH)
方法中可能存在关闭流时出现异常情况
53.RC: Suspicious reference comparison to constant (RC_REF_COMPARISON_BAD_PRACTICE)
当两者为不同类型的对象时使用equals方法来比较它们的值是否相等，而不是使用==方法。例如比较的两者为java.lang.Integer, java.lang.Float
54.RC: Suspicious reference comparison of Boolean values (RC_REF_COMPARISON_BAD_PRACTICE_BOOLEAN)
使用== 或者 !=操作符来比较两个 Boolean类型的对象，建议使用equals方法。
55.RR: Method ignores results of InputStream.read() (RR_NOT_CHECKED)
InputStream.read方法忽略返回的多个字符，如果对结果没有检查就没法正确处理用户读取少量字符请求的情况。
56.RR: Method ignores results of InputStream.skip() (SR_NOT_CHECKED)
InputStream.skip()方法忽略返回的多个字符，如果对结果没有检查就没法正确处理用户跳过少量字符请求的情况
57.RV: Method ignores exceptional return value (RV_RETURN_VALUE_IGNORED_BAD_PRACTICE)
方法忽略返回值的异常信息
58.SI: Static initializer creates instance before all static final fields assigned (SI_INSTANCE_BEFORE_FINALS_ASSIGNED)
在所有的static final字段赋值之前去使用静态初始化的方法创建一个类的实例。
59.Se: Non-serializable value stored into instance field of a serializable class (SE_BAD_FIELD_STORE)
非序列化的值保存在声明为序列化的的非序列化字段中
60.Se: Comparator doesn't implement Serializable (SE_COMPARATOR_SHOULD_BE_SERIALIZABLE)
Comparator接口没有实现Serializable接口
61.Se: Serializable inner class (SE_INNER_CLASS)
序列化内部类
62.Se: serialVersionUID isn't final (SE_NONFINAL_SERIALVERSIONID)
关于UID类的检查内容省略
63.Se: Class is Serializable but its superclass doesn't define a void constructor (SE_NO_SUITABLE_CONSTRUCTOR)
子类序列化时父类没有提供一个void的构造函数
64.Se: Class is Externalizable but doesn't define a void constructor (SE_NO_SUITABLE_CONSTRUCTOR_FOR_EXTERNALIZATION)
Externalizable 实例类没有定义一个void类型的构造函数
65.Se: The readResolve method must be declared with a return type of Object. (SE_READ_RESOLVE_MUST_RETURN_OBJECT)
readResolve从流中读取类的一个实例，此方法必须声明返回一个Object类型的对象
66.Se: Transient field that isn't set by deserialization. (SE_TRANSIENT_FIELD_NOT_RESTORED)
This class contains a field that is updated at multiple places in the class, thus it seems to be part of the state of the class. However, since the field is marked as transient and not set in readObject or readResolve, it will contain the default value in any deserialized instance of the class.
67.SnVI: Class is Serializable, but doesn't define serialVersionUID (SE_NO_SERIALVERSIONID)
一个类实现了Serializable接口但是没有定义serialVersionUID类型的变量。序列化运行时使用一个称为 serialVersionUID 的版本号与每个可序列化类相关联，该序列号在反序列化过程中用于验证序列化对象的发送者和接收者是否为该对象加载了与序列化兼容的类。如果接收者加载的该对象的类的 serialVersionUID 与对应的发送者的类的版本号不同，则反序列化将会导致 InvalidClassException。可序列化类可以通过声明名为 "serialVersionUID" 的字段（该字段必须是静态 (static)、最终 (final) 的 long 型字段）显式声明其自己的 serialVersionUID： 
 ANY-ACCESS-MODIFIER static final long serialVersionUID = 42L;
68.UI: Usage of GetResource may be unsafe if class is extended (UI_INHERITANCE_UNSAFE_GETRESOURCE)
当一个类被子类继承后不要使用this.getClass().getResource(...)来获取资源
四、Correctness关于代码正确性相关方面的
1.BC: Impossible cast (BC_IMPOSSIBLE_CAST)
不可能的类转换，执行时会抛出ClassCastException
2.BC: Impossible downcast (BC_IMPOSSIBLE_DOWNCAST)
父类在向下进行类型转换时抛出ClassCastException
3.BC: Impossible downcast of toArray() result (BC_IMPOSSIBLE_DOWNCAST_OF_TOARRAY)
集合转换为数组元素时发生的类转换错误。
This code is casting the result of calling toArray() on a collection to a type more specific than Object[], as in: 
String[] getAsArray(Collection<String> c) {
  return (String[]) c.toArray();
  }
This will usually fail by throwing a ClassCastException. The toArray() of almost all collections return an Object[]. They can't really do anything else, since the Collection object has no reference to the declared generic type of the collection. 
The correct way to do get an array of a specific type from a collection is to use c.toArray(new String[]); or c.toArray(new String[c.size()]); (the latter is slightly more efficient). 
4.BC: instanceof will always return false (BC_IMPOSSIBLE_INSTANCEOF)
采用instaneof方法进行比较时总是返回false。前提是保证它不是由于某些逻辑错误造成的。
5.BIT: Incompatible bit masks (BIT_AND)
错误的使用&位操作符，例如(e & C)
6.BIT: Check to see if ((...) & 0) == 0 (BIT_AND_ZZ)
检查恒等的逻辑错误
7.BIT: Incompatible bit masks (BIT_IOR)
错误的使用|位操作符，例如(e | C)
8.BIT: Check for sign of bitwise operation (BIT_SIGNED_CHECK_HIGH_BIT)
检查逻辑运算符操作返回的标识。例如((event.detail & SWT.SELECTED) > 0)，建议采用!=0代替>0
9.BOA: Class overrides a method implemented in super class Adapter wrongly (BOA_BADLY_OVERRIDDEN_ADAPTER)
子类错误的覆写父类中用于适配监听其他事件的方法，从而导致当触发条件发生时不能被监听者调用
10.Bx: Primitive value is unboxed and coerced for ternary operator (BX_UNBOXED_AND_COERCED_FOR_TERNARY_OPERATOR)
在三元运算符操作时如果没有对值进行封装或者类型转换。例如：b ? e1 : e2
11.DLS: Dead store of class literal (DLS_DEAD_STORE_OF_CLASS_LITERAL)
以类的字面名称方式为一个字段赋值后再也没有去使用它，在1.4jdk中它会自动调用静态的初始化方法，而在jdk1.5中却不会去执行。
12.DLS: Overwritten increment (DLS_OVERWRITTEN_INCREMENT)
覆写增量增加错误i = i++
13.DMI: Bad constant value for month (DMI_BAD_MONTH)
hashNext方法调用next方法。
14.DMI: Collections should not contain themselves (DMI_COLLECTIONS_SHOULD_NOT_CONTAIN_THEMSELVES)
集合没有包含他们自己本身。
15.DMI: Invocation of hashCode on an array (DMI_INVOKING_HASHCODE_ON_ARRAY)
数组直接使用hashCode方法来返回哈希码。
int [] a1 = new int[]{1,2,3,4};
	System.out.println(a1.hashCode());
	System.out.println(java.util.Arrays.hashCode(a1));
16.DMI: Double.longBitsToDouble invoked on an int (DMI_LONG_BITS_TO_DOUBLE_INVOKED_ON_INT)
17.DMI: Vacuous call to collections (DMI_VACUOUS_SELF_COLLECTION_CALL)
集合的调用不能被感知。例如c.containsAll(c)总是返回true，而c.retainAll(c)的返回值不能被感知。
18.Dm: Can't use reflection to check for presence of annotation without runtime retention (DMI_ANNOTATION_IS_NOT_VISIBLE_TO_REFLECTION)
Unless an annotation has itself been annotated with @Retention(RetentionPolicy.RUNTIME), the annotation can't be observed using reflection (e.g., by using the isAnnotationPresent method). .
19.Dm: Useless/vacuous call to EasyMock method (DMI_VACUOUS_CALL_TO_EASYMOCK_METHOD)
While ScheduledThreadPoolExecutor inherits from ThreadPoolExecutor, a few of the inherited tuning methods are not useful for it. In particular, because it acts as a fixed-sized pool using corePoolSize threads and an unbounded queue, adjustments to maximumPoolSize have no useful effect.
20.EC: equals() used to compare array and nonarray (EC_ARRAY_AND_NONARRAY)
数组对象使用equals方法和非数组对象进行比较。即使比较的双方都是数组对象也不应该使用equals方法，而应该比较它们的内容是否相等使用java.util.Arrays.equals(Object[], Object[]);
21.EC: equals(...) used to compare incompatible arrays (EC_INCOMPATIBLE_ARRAY_COMPARE)
使用equls方法去比较类型不相同的数组。例如：String[] and StringBuffer[], or String[] and int[]
22.EC: Call to equals() with null argument (EC_NULL_ARG)
调用equals的对象为null
23.EC: Call to equals() comparing unrelated class and interface (EC_UNRELATED_CLASS_AND_INTERFACE)
使用equals方法比较不相关的类和接口
24.EC: Call to equals() comparing different interface types (EC_UNRELATED_INTERFACES)
调用equals方法比较不同类型的接口
25.EC: Call to equals() comparing different types (EC_UNRELATED_TYPES)
调用equals方法比较不同类型的类
26.EC: Using pointer equality to compare different types (EC_UNRELATED_TYPES_USING_POINTER_EQUALITY)
This method uses using pointer equality to compare two references that seem to be of different types. The result of this comparison will always be false at runtime.
27.Eq: equals method always returns false (EQ_ALWAYS_FALSE)
使用equals方法返回值总是false
28.Eq: equals method always returns true (EQ_ALWAYS_TRUE)
equals方法返回值总是true
29.Eq: equals method compares class names rather than class objects (EQ_COMPARING_CLASS_NAMES)
使用equals方法去比较一个类的实例和类的类型
30.Eq: Covariant equals() method defined for enum (EQ_DONT_DEFINE_EQUALS_FOR_ENUM)
This class defines an enumeration, and equality on enumerations are defined using object identity. Defining a covariant equals method for an enumeration value is exceptionally bad practice, since it would likely result in having two different enumeration values that compare as equals using the covariant enum method, and as not equal when compared normally. Don't do it.
31.Eq: equals() method defined that doesn't override equals(Object) (EQ_OTHER_NO_OBJECT)
类中定义的equals方法时不要覆写equals（Object）方法
32.Eq: equals() method defined that doesn't override Object.equals(Object) (EQ_OTHER_USE_OBJECT)
类中定义的equals方法时不要覆写Object中的equals（Object）方法
33.Eq: equals method overrides equals in superclass and may not be symmetric (EQ_OVERRIDING_EQUALS_NOT_SYMMETRIC)
34.Eq: Covariant equals() method defined, Object.equals(Object) inherited (EQ_SELF_USE_OBJECT)
类中定义了一组equals方法，但是都是继承的java.lang.Object class中的equals(Object)方法
35.FE: Doomed test for equality to NaN (FE_TEST_IF_EQUAL_TO_NOT_A_NUMBER)
This code checks to see if a floating point value is equal to the special Not A Number value (e.g., if (x == Double.NaN)). However, because of the special semantics of NaN, no value is equal to Nan, including NaN. Thus, x == Double.NaN always evaluates to false. To check to see if a value contained in x is the special Not A Number value, use Double.isNaN(x) (or Float.isNaN(x) if x is floating point precision).
36.FS: Format string placeholder incompatible with passed argument (VA_FORMAT_STRING_BAD_ARGUMENT)
错误使用参数类型来格式化字符串
37.FS: The type of a supplied argument doesn't match format specifier (VA_FORMAT_STRING_BAD_CONVERSION)
指定的格式字符串和参数类型不匹配，例如：String.format("%d", "1")
38.FS: MessageFormat supplied where printf style format expected (VA_FORMAT_STRING_EXPECTED_MESSAGE_FORMAT_SUPPLIED)
但用String的format方法时实际调用了ＭｅｓｓａｇｅＦｏｒｍａｔ中干的格式化方法而引起格式化结果出错。
39.FS: More arguments are passed than are actually used in the format string (VA_FORMAT_STRING_EXTRA_ARGUMENTS_PASSED)
使用String的format方法时有非法的参数也经过了格式化操作。
40.FS: Illegal format string (VA_FORMAT_STRING_ILLEGAL)
格式化String对象语句错误
41.FS: Format string references missing argument (VA_FORMAT_STRING_MISSING_ARGUMENT)
String的format操作缺少必要的参数。
42.FS: No previous argument for format string (VA_FORMAT_STRING_NO_PREVIOUS_ARGUMENT)
格式字符串定义错误，例如：formatter.format("%<s %s", "a", "b"); 抛出MissingFormatArgumentException异常
43.GC: No relationship between generic parameter and method argument (GC_UNRELATED_TYPES)
This call to a generic collection method contains an argument with an incompatible class from that of the collection's parameter (i.e., the type of the argument is neither a supertype nor a subtype of the corresponding generic type argument). Therefore, it is unlikely that the collection contains any objects that are equal to the method argument used here. Most likely, the wrong value is being passed to the method.
In general, instances of two unrelated classes are not equal. For example, if the Foo and Bar classes are not related by subtyping, then an instance of Foo should not be equal to an instance of Bar. Among other issues, doing so will likely result in an equals method that is not symmetrical. For example, if you define the Foo class so that a Foo can be equal to a String, your equals method isn't symmetrical since a String can only be equal to a String. 
In rare cases, people do define nonsymmetrical equals methods and still manage to make their code work. Although none of the APIs document or guarantee it, it is typically the case that if you check if a Collection<String> contains a Foo, the equals method of argument (e.g., the equals method of the Foo class) used to perform the equality checks. 
44.HE: Signature declares use of unhashable class in hashed construct (HE_SIGNATURE_DECLARES_HASHING_OF_UNHASHABLE_CLASS)
A method, field or class declares a generic signature where a non-hashable class is used in context where a hashable class is required. A class that declares an equals method but inherits a hashCode() method from Object is unhashable, since it doesn't fulfill the requirement that equal objects have equal hashCodes.
45.HE: Use of class without a hashCode() method in a hashed data structure (HE_USE_OF_UNHASHABLE_CLASS)
A class defines an equals(Object) method but not a hashCode() method, and thus doesn't fulfill the requirement that equal objects have equal hashCodes. An instance of this class is used in a hash data structure, making the need to fix this problem of highest importance.
46.ICAST: integral value cast to double and then passed to Math.ceil (ICAST_INT_CAST_TO_DOUBLE_PASSED_TO_CEIL)
integral的值转换为double后使用了Math.ceil方法
47.ICAST: int value cast to float and then passed to Math.round (ICAST_INT_CAST_TO_FLOAT_PASSED_TO_ROUND)
int 类型的值转换为float类型之后调用了Math.round方法
48.IJU: JUnit assertion in run method will not be noticed by JUnit (IJU_ASSERT_METHOD_INVOKED_FROM_RUN_METHOD)
在JUnit中的断言在run方法中不会被告知
49.IJU: TestCase declares a bad suite method (IJU_BAD_SUITE_METHOD)
在一个JUnit类中声明的一个suite()方法必须声明为
public static junit.framework.Test suite()
或者
public static junit.framework.TestSuite suite()的形式。
50.IL: A collection is added to itself (IL_CONTAINER_ADDED_TO_ITSELF)
集合本身作为add方法的参数，这样会引起内容溢出。
51.IL: An apparent infinite loop (IL_INFINITE_LOOP)
方法的自调用引起的死循环
52.IM: Integer multiply of result of integer remainder (IM_MULTIPLYING_RESULT_OF_IREM)
和整数余数进行乘法运算。例如：i % 60 * 1000 是进行(i % 60) * 1000运算而不是 i % (60 * 1000)
53.INT: Bad comparison of nonnegative value with negative constant (INT_BAD_COMPARISON_WITH_NONNEGATIVE_VALUE)
保证非负数和负数进行比较
54.INT: Bad comparison of signed byte (INT_BAD_COMPARISON_WITH_SIGNED_BYTE)
比较有符合数，要先把有符号数转换为无符合数再进行比较
55.IO: Doomed attempt to append to an object output stream (IO_APPENDING_TO_OBJECT_OUTPUT_STREAM)
宣布试图在对象的输出流处添加元素，如果你希望能够添加进一个对象的输出流中必须保证对象的输出流处于打开状态。
56.IP: A parameter is dead upon entry to a method but overwritten (IP_PARAMETER_IS_DEAD_BUT_OVERWRITTEN)
The initial value of this parameter is ignored, and the parameter is overwritten here. This often indicates a mistaken belief that the write to the parameter will be conveyed back to the caller.
传入参数的值被忽略，但是对传入值进行了修改，并返回给了调用者
57.MF: Class defines field that masks a superclass field (MF_CLASS_MASKS_FIELD)
子类中定义了和父类中同名的字段。在调用时会出错
58.MF: Method defines a variable that obscures a field (MF_METHOD_MASKS_FIELD)
在方法中定义的局部变量和类变量或者父类变量同名，从而引起字段混淆。
59.NP: Null pointer dereference (NP_ALWAYS_NULL)
对象赋为null值后 没有被重新赋值
60.NP: Null pointer dereference in method on exception path (NP_ALWAYS_NULL_EXCEPTION)
A pointer which is null on an exception path is dereferenced here.  This will lead to a NullPointerException when the code is executed.  Note that because FindBugs currently does not prune infeasible exception paths, this may be a false warning.
Also note that FindBugs considers the default case of a switch statement to be an exception path, since the default case is often infeasible.
空指针引用上调用去除引用方法，将发生空指针异常
61.NP: Method does not check for null argument (NP_ARGUMENT_MIGHT_BE_NULL)
方法没有判断参数是否为空
62.NP: close() invoked on a value that is always null (NP_CLOSING_NULL)
一个为空的对象调用close方法
63.NP: Null value is guaranteed to be dereferenced (NP_GUARANTEED_DEREF)
There is a statement or branch that if executed guarantees that a value is null at this point, and that value that is guaranteed to be dereferenced (except on forward paths involving runtime exceptions).
在正常的null判断分支上，对象去除引用操作是受保护的不允许的
64.NP: Value is null and guaranteed to be dereferenced on exception path (NP_GUARANTEED_DEREF_ON_EXCEPTION_PATH)
There is a statement or branch on an exception path that if executed guarantees that a value is null at this point, and that value that is guaranteed to be dereferenced (except on forward paths involving runtime exceptions).
65.NP: Method call passes null to a nonnull parameter (NP_NONNULL_PARAM_VIOLATION)

方法中为null的参数没有被重新赋值

	void test(){

String ss = null;

sya(ss);

	}	

	public void sya(String ad){

ad.getBytes();

	}

66.NP: Method may return null, but is declared @NonNull (NP_NONNULL_RETURN_VIOLATION)

方法声明了返回值不能为空，但是方法中有可能返回null

67.NP: A known null value is checked to see if it is an instance of a type (NP_NULL_INSTANCEOF)

检查一个为null的值是否是想要的类型对象，而不是由于粗心或者逻辑错误引起的

68.NP: Possible null pointer dereference (NP_NULL_ON_SOME_PATH)

对象可能没有重新赋值

69.NP: Possible null pointer dereference in method on exception path (NP_NULL_ON_SOME_PATH_EXCEPTION)

A reference value which is null on some exception control path is dereferenced here.  This may lead to a NullPointerException when the code is executed.  Note that because FindBugs currently does not prune infeasible exception paths, this may be a false warning.

Also note that FindBugs considers the default case of a switch statement to be an exception path, since the default case is often infeasible.

在异常null值处理分支调用的方法上，可能存在对象去除引用操作

70.NP: Method call passes null for nonnull parameter (NP_NULL_PARAM_DEREF_ALL_TARGETS_DANGEROUS)

方法参数中声明为nonnull类型的参数为null

void test(){

String ss = null;

sya(ss);

	}	

	public void sya(@nonnull String ad){

ad.getBytes();

	}

71.NP: Store of null value into field annotated NonNull (NP_STORE_INTO_NONNULL_FIELD)

为一个已经声明为不能为null值的属性赋值为null。

72.Nm: Class defines equal(Object); should it be equals(Object)? (NM_BAD_EQUAL)

类中定义了一个equal方法但是却不是覆写的Object对象的equals方法

73.Nm: Class defines hashcode(); should it be hashCode()? (NM_LCASE_HASHCODE)

类中定义了一个hashCode方法但是却不是覆写的Object中的hashCode方法

74.Nm: Class defines tostring(); should it be toString()? (NM_LCASE_TOSTRING)

类中定义了一个toString方法但是却不是覆写的Object中的toString方法

75.Nm: Apparent method/constructor confusion (NM_METHOD_CONSTRUCTOR_CONFUSION)

构造方法定义混乱，保证一个标准的构造函数。	例如：

	SA(){	}

	void SA(){

	}

76.Nm: Very confusing method names (NM_VERY_CONFUSING)

混乱的方法命名，如getName和getname方法同时出现的时候

77.Nm: Method doesn't override method in superclass due to wrong package for parameter (NM_WRONG_PACKAGE)

方法因为取了不同包中的同名的对象而没有正确覆写父类中的同名方法

import alpha.Foo;

public class A {

  public int f(Foo x) { return 17; }

}

import beta.Foo;

public class B extends A {

  public int f(Foo x) { return 42; }

}

78.QBA: Method assigns boolean literal in boolean expression (QBA_QUESTIONABLE_BOOLEAN_ASSIGNMENT)

再if或者while表达式中使用boolean类型的值时应该使用去判断，而不是采用=操作

79.RC: Suspicious reference comparison (RC_REF_COMPARISON)

比较两个对象值是否相等时应该采用equals方法，而不是方法

80.RE: Invalid syntax for regular expression (RE_BAD_SYNTAX_FOR_REGULAR_EXPRESSION)

对正则表达式使用了错误的语法，会抛出未经检查的异常，表明正则表达式模式中的语法错误。

81.RE: File.separator used for regular expression (RE_CANT_USE_FILE_SEPARATOR_AS_REGULAR_EXPRESSION)

使用正则表达式使用了错误的文件分隔符，在windows系统中正则表达式不会匹配’\’而应该使用'\'

82.RV: Random value from 0 to 1 is coerced to the integer 0 (RV_01_TO_INT)

从0到1随机值被强制为整数值0。在强制得到一个整数之前，你可能想得到多个随机值。或使用Random.nextInt（n）的方法。

83.RV: Bad attempt to compute absolute value of signed 32-bit hashcode (RV_ABSOLUTE_VALUE_OF_HASHCODE)

此代码生成一个哈希码，然后计算该哈希码的绝对值。如果哈希码是Integer.MIN_VALUE的，那么结果将是负数（因为Math.abs（Integer.MIN_VALUE的）== Integer.MIN_VALUE的）。

在2^ 32值之外字符串有一个Integer.MIN_VALUE的hashCode包括“polygenelubricants”，“GydZG_”和“，”DESIGNING WORKHOUSES “。

84.RV: Bad attempt to compute absolute value of signed 32-bit random integer (RV_ABSOLUTE_VALUE_OF_RANDOM_INT)

此代码生成一个随机的符号整数，然后计算该随机整数的绝对值。如果随机数生成数绝对值为Integer.MIN_VALUE的，那么结果将是负数（因为Math.abs（Integer.MIN_VALUE的）== Integer.MIN_VALUE的）。

85.RV: Exception created and dropped rather than thrown (RV_EXCEPTION_NOT_THROWN)

此代码创建一个异常（或错误）的对象，但不会用它做任何事情。例如：if (x < 0)

  new IllegalArgumentException("x must be nonnegative");

这可能是程序员的意图抛出创建的异常：

if (x < 0)

  throw new IllegalArgumentException("x must be nonnegative");

86.RV: Method ignores return value (RV_RETURN_VALUE_IGNORED)

该方法的返回值应该进行检查。这种警告通常出现在调用一个不可变对象的方法，认为它更新了对象的值。例如：String dateString = getHeaderField(name);

dateString.trim();

程序员似乎以为trim（）方法将更新dateString引用的字符串。但由于字符串是不可改变的，trim（）函数返回一个新字符串值，在这里它是被忽略了。该代码应更正：

String dateString = getHeaderField(name);

dateString = dateString.trim();

87.RpC: Repeated conditional tests (RpC_REPEATED_CONDITIONAL_TEST)

该代码包含对同一个条件试验了两次，两边完全一样例如：（如X == 0 | | x == 0）。可能第二次出现是打算判断别的不同条件（如X == 0 | | y== 0）。

88.SA: Double assignment of field (SA_FIELD_DOUBLE_ASSIGNMENT)

方法中的字段包含了双重任务，例如： 

 int x;

  public void foo() {

   x = x = 17;

  }

这种为变量赋值是无用的，并可能表明一个逻辑错误或拼写错误。

89.SA: Self assignment of field (SA_FIELD_SELF_ASSIGNMENT)

方法中包含自己对自己赋值的字段。例如：

int x;

  public void foo() {

    x = x;

  }

90.SA: Self comparison of field with itself (SA_FIELD_SELF_COMPARISON)

字段自己进行自比较可能表明错误或逻辑错误。

91.SA: Self comparison of value with itself (SA_LOCAL_SELF_COMPARISON)

方法中对一个局部变量自身进行比较运算，并可说明错误或逻辑错误。请确保您是比较正确的事情。

92.SA: Nonsensical self computation involving a variable (e.g., x & x) (SA_LOCAL_SELF_COMPUTATION)

此方法对同一变量执行了荒谬的计算（如x&x或x-x）操作。由于计算的性质，这一行动似乎没有意义，并可能表明错误或逻辑错误。

93.SF: Dead store due to switch statement fall through (SF_DEAD_STORE_DUE_TO_SWITCH_FALLTHROUGH)

在swtich中先前的case值因为swtich执行失败而被覆写，这就像是忘记使用break推出或者没有使用return语句放回先前的值一样。

94.SF: Dead store due to switch statement fall through to throw (SF_DEAD_STORE_DUE_TO_SWITCH_FALLTHROUGH_TO_THROW)

在swtich中因为出现异常而忽略了对case值的保存。

95.SIC: Deadly embrace of non-static inner class and thread local (SIC_THREADLOCAL_DEADLY_EMBRACE)

如果是一个静态内部类。实际上，在内部类和当前线程有死锁的可能。由于内部类不是静态的，它保留了对外部类的引用。如果线程包含对一个内部类实例的引用，那么内外实例的实例都可以被获取，这样就不具备垃圾会回收的资格。

96.SIO: Unnecessary type check done using instanceof operator (SIO_SUPERFLUOUS_INSTANCEOF)

在进行instanceof操作时进行没有必要的类型检查

97.STI: Unneeded use of currentThread() call, to call interrupted() (STI_INTERRUPTED_ON_CURRENTTHREAD)

此方法调用Thread.currentThread（）调用，只需调用interrupted（）方法。由于interrupted（）是一个静态方法， Thread.interrupted（）更简单易用。

98.STI: Static Thread.interrupted() method invoked on thread instance (STI_INTERRUPTED_ON_UNKNOWNTHREAD)

调用不是当前线程对象的Thread.interrupted()方法，由于interrupted（）方法是静态的，interrupted方法将会调用一个和作者原计划不同的对象。

99.Se: Method must be private in order for serialization to work (SE_METHOD_MUST_BE_PRIVATE)

这个类实现了Serializable接口，并定义自定义序列化的方法/反序列化。但由于这种方法不能声明为private，将被序列化/反序列化的API忽略掉。

100.Se: The readResolve method must not be declared as a static method. (SE_READ_RESOLVE_IS_STATIC)

为使readResolve方法得到序列化机制的识别，不能作为一个静态方法来声明。

101.UMAC: Uncallable method defined in anonymous class (UMAC_UNCALLABLE_METHOD_OF_ANONYMOUS_CLASS)

在匿名类中定义了一个既没有覆写超类中方法也不能直接调用的方法。因为在其他类的方法不能直接引用匿名类声明的方法，似乎这种方法不能被调用，这种方法可能只是没有任何作用的代码，但也可能覆写超类中声明。

102.UR: Uninitialized read of field in constructor (UR_UNINIT_READ)

此构造方法中使用了一个尚未赋值的字段或属性。

	String a;

	public SA() {

String abc = a;

System.out.println(abc);

	}

103.UR: Uninitialized read of field method called from constructor of superclass (UR_UNINIT_READ_CALLED_FROM_SUPER_CONSTRUCTOR)

方法被超类的构造函数调用时，在当前类中的字段或属性还没有被初始化。例如：

abstract class A {

  int hashCode;

  abstract Object getValue();

  A() {

    hashCode = getValue().hashCode();

    }

  }

class B extends A {

  Object value;

  B(Object v) {

    this.value = v;

    }

  Object getValue() {

    return value;

  }

  }

当B是创建时，A的构造函数将在B为value赋值之前触发，然而在A的初始化方法调用getValue方法时value这个变量还没有被初始化。

104.USELESS_STRING: Invocation of toString on an array (DMI_INVOKING_TOSTRING_ON_ANONYMOUS_ARRAY)

该代码调用上匿名数组的toString（）方法，产生的结果形如[@ 16f0472并没有实际的意义。考虑使用Arrays.toString方法来转换成可读的字符串，提供该数组的内容数组。例如：

String[] a = { "a" };

System.out.println(a.toString());

//正确的使用为

System.out.println(Arrays.toString(a));

105.USELESS_STRING: Invocation of toString on an array (DMI_INVOKING_TOSTRING_ON_ARRAY)

该代码调用上数组的toString（）方法，产生的结果形如[@ 16f0472并不能显示数组的真实内容。考虑使用Arrays.toString方法来转换成可读的字符串，提供该数组的内容数组

106.UwF: Field only ever set to null (UWF_NULL_FIELD)

字段的值总是为null值，所有读取该字段的值都为null。检查错误，如果它确实没有用就删除掉。

107.UwF: Unwritten field (UWF_UNWRITTEN_FIELD

此字段是永远不会写入值。所有读取将返回默认值。检查错误（如果它被初始化？），如果它确实没有用就删除掉。


五：Performance关于代码性能相关方面的
1.Bx: Primitive value is boxed and then immediately unboxed (BX_BOXING_IMMEDIATELY_UNBOXED)
对原始值进行装箱，然后立即取消装箱。这可能是在一个未要求装箱的地方进行了手动装箱，从而迫使编译器进行立即撤消装箱的操作
2.Bx: Primitive value is boxed then unboxed to perform primitive coercion (BX_BOXING_IMMEDIATELY_UNBOXED_TO_PERFORM_COERCION)
对原始值进行装箱然后立即把它强制转换为另外一种原始类型。例如：
new Double(d).intValue()应该直接进行强制转换例如：(int) d
3.Bx: Method allocates a boxed primitive just to call toString (DM_BOXED_PRIMITIVE_TOSTRING)
仅仅为了调用封装类的toString()而对原始类型进行封装操作。比这种方法更有效的是调用封装类的toString(…)方法例如：
new Integer(1).toString()    替换为   Integer.toString(1)
new Long(1).toString()    替换为   Long.toString(1) 
new Float(1.0).toString()    替换为   Float.toString(1.0) 
new Double(1.0).toString()    替换为   Double.toString(1.0) 
new Byte(1).toString()    替换为   Byte.toString(1) 
new Short(1).toString()    替换为   Short.toString(1) 
new Boolean(true).toString()    替换为   Boolean.toString(true)
4.Bx: Method invokes inefficient floating-point Number constructor; use static valueOf instead (DM_FP_NUMBER_CTOR)
使用new Double(double)方法总是会创建一个新的对象，然而使用Double.valueOf(double)方法可以把值保存在编辑器或者class library、JVM中。使用存储值的方式来避免对象的分配可以或得更好的代码性能
除非类必须符合Java 1.5以前的JVM，否则请使用自动装箱或valueOf（）方法创建Double和Float实例。
5.Bx: Method invokes inefficient Number constructor; use static valueOf instead (DM_NUMBER_CTOR)
使用new Integer(int)方法总是会创建一个新的对象，然而使用Integer.valueOf(int)方法可以把值保存在编辑器或者class library、JVM中。使用存储值的方式来避免对象的分配可以或得更好的代码性能
除非类必须符合Java 1.5以前的JVM，否则请使用自动装箱或valueOf（）方法创建Long, Integer, Short, Character, Byte实例。
6.Dm: The equals and hashCode methods of URL are blocking (DMI_BLOCKING_METHODS_ON_URL)
使用equals和hashCode方法来对url进行资源标识符解析时会引起堵塞。考虑使用java.net.URI来代替。
7.Dm: Maps and sets of URLs can be performance hogs (DMI_COLLECTION_OF_URLS)
方法或者字段使用url的map/set集合。因为equals方法或者hashCode方法来进行资源标识符解析时都会引起堵塞。考虑使用java.net.URI来代替。
8.Dm: Method invokes inefficient Boolean constructor; use Boolean.valueOf(...) instead (DM_BOOLEAN_CTOR)
使用new方法创建一个java.lang.Boolean类型能够的实例对象是浪费空间的，因为Boolean对象是不可变的而且只有两个有用的值。使用Boolean.valueOf()或者Java1.5中的自动装箱功能来创建一个Boolean实例。
9.Dm: Explicit garbage collection; extremely dubious except in benchmarking code (DM_GC)
在代码中显式的调用垃圾回收命名，这样做并不能起作用。在过去，有人在关闭操作或者finalize方法中调用垃圾回收方法导致了很多的性能浪费。这样大规模回收对象时会造成处理器运行缓慢。
10.Dm: Use the nextInt method of Random rather than nextDouble to generate a random integer (DM_NEXTINT_VIA_NEXTDOUBLE)
 如果r是一个java.util.Random对象，你可以使r.nextInt(n)生成一个0到n-1之前的随机数，而不是使用(int)(r.nextDouble() * n)
11.Dm: Method invokes inefficient new String(String) constructor (DM_STRING_CTOR)
使用java.lang.String(String)构造函数会浪费内存因为这种构造方式和String作为参数在功能上容易混乱。只是使用String直接作为参数的形式
12.Dm: Method invokes toString() method on a String (DM_STRING_TOSTRING)
调用String.toString()是多余的操作，只要使用String就可以了。
13.Dm: Method invokes inefficient new String() constructor (DM_STRING_VOID_CTOR)
使用没有参数的构造方法去创建新的String对象是浪费内存空间的，因为这样创建会和空字符串“”混淆。Java中保证完成相同的构造方法会产生描绘相同的String对象。所以你只要使用空字符串来创建就可以了。
14.ITA: Method uses toArray() with zero-length array argument (ITA_INEFFICIENT_TO_ARRAY)
当使用集合的toArray()方法时使用数组长度为0的数组作为参数。比这更有效的一种方法是
myCollection.toArray(new Foo[myCollection.size()])，如果数组的长度足够大就可以直接把集合中的内容包装到数组中直接返回从而避免了第二次创建一个新的数组来存放集合中值。
15.SBSC: Method concatenates strings using + in a loop (SBSC_USE_STRINGBUFFER_CONCATENATION)
在循环中构建一个String对象时从性能上讲使用StringBuffer来代替String对象
例如：
// This is bad
  String s = "";
  for (int i = 0; i < field.length; ++i) {
    s = s + field[i];
  }


  // This is better
  StringBuffer buf = new StringBuffer();
  for (int i = 0; i < field.length; ++i) {
    buf.append(field[i]);
  }
  String s = buf.toString();
16.SS: Unread field: should this field be static? (SS_SHOULD_BE_STATIC)
类中所包含的final属性字段在编译器中初始化为静态的值。考虑在定义时就把它定义为static类型的。
17.UM: Method calls static Math class method on a constant value (UM_UNNECESSARY_MATH)
在方法中使用了java.lang.Math的静态方法代替常量来使用，使用常量速度和准确度会更好。 以下为Math中的方法产生的值。
Method Parameter 
abs -any- 
acos 0.0 or 1.0 
asin 0.0 or 1.0 
atan 0.0 or 1.0 
atan2 0.0 cbrt 0.0 or 1.0 
ceil -any- 
cos 0.0 
cosh 0.0 
exp 0.0 or 1.0 
expm1 0.0 
floor -any- 
log 0.0 or 1.0 
log10 0.0 or 1.0 
rint -any- 
round -any- 
sin 0.0 
sinh 0.0 
sqrt 0.0 or 1.0 
tan 0.0 
tanh 0.0 
toDegrees 0.0 or 1.0 
toRadians 0.0
18.UPM: Private method is never called (UPM_UNCALLED_PRIVATE_METHOD)
定义为Private类型方法从未被调用，应该被删除。
19.UrF: Unread field (URF_UNREAD_FIELD)
类中定义的属性从未被调用，建议删除。
20.UuF: Unused field (UUF_UNUSED_FIELD)
类中定义的属性从未被使用，建议删除。
21.WMI: Inefficient use of keySet iterator instead of entrySet iterator (WMI_WRONG_MAP_ITERATOR)
当方法中接受一个Map类型的参数时，使用keySet的迭代器比使用entrySet的迭代器效率要高。
六：Internationalization关于代码国际化相关方面的
Dm: Consider using Locale parameterized version of invoked method (DM_CONVERT_CASE)
使用平台默认的编码格式对字符串进行大小写转换，这可能导致国际字符的转换不当。使用以下方式对字符进行转换
String.toUpperCase( Locale l )
String.toLowerCase( Locale l )
七：Multithreaded correctness关于代码多线程正确性相关方面的
1.DL: Synchronization on Boolean could lead to deadlock (DL_SYNCHRONIZATION_ON_BOOLEAN)
该代码同步一个封装的原始常量，例如一个Boolean类型。
private static Boolean inited = Boolean.FALSE;
...
  synchronized(inited) { 
    if (!inited) {
       init();
       inited = Boolean.TRUE;
       }
     }
...
由于通常只存在两个布尔对象，此代码可能是同步的其他无关的代码中相同的对象，这时会导致反应迟钝和可能死锁
2.DL: Synchronization on boxed primitive could lead to deadlock (DL_SYNCHRONIZATION_ON_BOXED_PRIMITIVE)
该代码同步一个封装的原始常量，例如一个Integer类型。
private static Integer count = 0;
...
  synchronized(count) { 
     count++;
     }
...
由于Integer对象可以共享和保存，此代码可能是同步的其他无关的代码中相同的对象，这时会导致反应迟钝和可能死锁
3.DL: Synchronization on interned String could lead to deadlock (DL_SYNCHRONIZATION_ON_SHARED_CONSTANT)
同步String类型的常量时，由于它被JVM中多个其他的对象所共有，这样在其他代码中会引起死锁。
4.DL: Synchronization on boxed primitive values (DL_SYNCHRONIZATION_ON_UNSHARED_BOXED_PRIMITIVE)
同步一个显然不是共有封装的原始值，例如一个Integer类型的对象。例如：
private static final Integer fileLock = new Integer(1);
...
  synchronized(fileLock) { 
     .. do something ..
     }
...
它最后被定义为以下方式来代替：private static final Object fileLock = new Object();


5.Dm: Monitor wait() called on Condition (DM_MONITOR_WAIT_ON_CONDITION)
方法中以java.util.concurrent.locks.Condition对象调用wait（）。等待一个条件发生时应该使用在Condition接口中定义的await()方法。
6.Dm: A thread was created using the default empty run method (DM_USELESS_THREAD)
这个方法没有通过run方法或者具体声明Thread类，也没有通过一个Runnable对象去定义一个线程，而这个线程出来浪费资源却什么也没有去做。
7.ESync: Empty synchronized block (ESync_EMPTY_SYNC)
该代码包含一个空的同步块：synchronized() {}
8.IS: Inconsistent synchronization (IS2_INCONSISTENT_SYNC)
不合理的同步
9.IS: Field not guarded against concurrent access (IS_FIELD_NOT_GUARDED)
域不是良好的同步访问---


此字段被标注为net.jcip.annotations.GuardedBy，但可以在某种程度上违反注释而去访问
10.JLM: Synchronization performed on Lock (JLM_JSR166_LOCK_MONITORENTER)
实现java.util.concurrent.locks.Lock的对象调用了同步的方法。应该这样处理，对象被锁定/解锁时使用acquire（）/ release（）方法而不是使用同步的方法。
11.LI: Incorrect lazy initialization of static field (LI_LAZY_INIT_STATIC)
静态域不正确的延迟初始化--
这种方法包含了一个不同步延迟初始化的非volatile静态字段。因为编译器或处理器可能会重新排列指令，如果该方法可以被多个线程调用，线程不能保证看到一个完全初始化的对象。你可以让字段可变来解决此问题
12.LI: Incorrect lazy initialization and update of static field (LI_LAZY_INIT_UPDATE_STATIC)
这种方法包含一个不同步延迟初始化的静态字段。之后为字段赋值，对象存储到该位置后进一步更新或访问。字段后尽快让其他线程能够访问。如果该方法的进一步访问该字段为初始化对象提供服务，然后你有一个非常严重的多线程bug，除非别的东西阻止任何其他线程访问存储的对象，直到它完全初始化。
即使你有信心，该方法是永远不会被多个线程调用时，在它的值还没有被充分初始化或移动，不把它设定为static字段时它可能会更好。
13.ML: Method synchronizes on an updated field (ML_SYNC_ON_UPDATED_FIELD)
对象获取一个可变字段时进行同步。这是没有意义的，因为不同的线程可以在不同的对象同步。
14.MSF: Mutable servlet field (MSF_MUTABLE_SERVLET_FIELD)
一个web服务一般只能创建一个servlet或者jsp的实例（例如：treates是一个单利类），它会被多个线程调用这个实例的方法服务于多个同时的请求。因此使用易变的字段属性产生竞争的情况。
15.MWN: Mismatched notify() (MWN_MISMATCHED_NOTIFY)
此方法调用Object.notify（）或Object.notifyAll（）而没有获取到该对象的对象锁。调用notify（）或notifyAll（）而没有持有该对象的对象锁，将导致IllegalMonitorStateException异常。
16.MWN: Mismatched wait() (MWN_MISMATCHED_WAIT)
此方法调用Object.wait()而没有获取到该对象的对象锁。调用wait（）而没有持有该对象的对象锁，将导致IllegalMonitorStateException异常。
17.NP: Synchronize and null check on the same field. (NP_SYNC_AND_NULL_CHECK_FIELD)
如果代码块是同步的，那么久不可能为空。如果是空，同步时就会抛出NullPointerException异常。最好是在另一个代码块中进行同步。
18.No: Using notify() rather than notifyAll() (NO_NOTIFY_NOT_NOTIFYALL)
调用notify（）而不是notifyAll（）方法。 Java的监控器通常用于多个条件。调用notify（）只唤醒一个线程，这意味着该线程被唤醒只是满足的当前的唯一条件。
19.RS: Class's readObject() method is synchronized (RS_READOBJECT_SYNC)
序列化类中定义了同步的readObject（）。通过定义，反序列化创建的对象只有一个线程可以访问，因此没有必要的readObject（）进行同步。如果的readObject（）方法本身造成对象对另一个线程可见，那么这本身就是不好的编码方式。
20.Ru: Invokes run on a thread (did you mean to start it instead?) (RU_INVOKE_RUN)
这种方法显式调用一个对象的run（）。一般来说，类是实现Runnable接口的，因为在一个新的线程他们将有自己的run（）方法，在这种情况下Thread.start（）方法调用是正确的。
21.SC: Constructor invokes Thread.start() (SC_START_IN_CTOR)
在构造函数中启动一个线程。如果类曾经被子类扩展过，那么这很可能是错的，因为线程将在子类构造之前开始启动。
22.SP: Method spins on field (SP_SPIN_ON_FIELD)
方法无限循环读取一个字段。编译器可合法悬挂宣读循环，变成一个无限循环的代码。这个类应该改变，所以使用适当的同步（包括等待和通知要求）
23.STCAL: Call to static Calendar (STCAL_INVOKE_ON_STATIC_CALENDAR_INSTANCE)
即使JavaDoc对此不包含暗示，而Calendars本身在多线程中使用就是不安全的。探测器发现当调用Calendars的实例时将会获得一个静态对象。
Calendar rightNow = Calendar.getInstance();
24.STCAL: Call to static DateFormat (STCAL_INVOKE_ON_STATIC_DATE_FORMAT_INSTANCE)
在官方的JavaDoc，DateFormats多线程使用本事就是不安全的。探测器发现调用一个DateFormat的实例将会获得一个静态对象。
myString = DateFormat.getDateInstance().format(myDate);


25.STCAL: Static Calendar (STCAL_STATIC_CALENDAR_INSTANCE)
Calendar在多线程中本身就是不安全的，如果在线程范围中共享一个Calendarde 实例而不使用一个同步的方法在应用中就会出现一些奇怪的行为。在sun.util.calendar.BaseCalendar.getCalendarDateFromFixedDate()中会抛出ArrayIndexOutOfBoundsExceptions or IndexOutOfBoundsExceptions异常。
26.STCAL: Static DateFormat (STCAL_STATIC_SIMPLE_DATE_FORMAT_INSTANCE)
DateFormat 在多线程中本身就是不安全的，如果在线程范围中共享一个DateFormat的实例而不使用一个同步的方法在应用中就会出现一些奇怪的行为。
27.SWL: Method calls Thread.sleep() with a lock held (SWL_SLEEP_WITH_LOCK_HELD)
当持有对象时调用Thread.sleep（）。这可能会导致很差的性能和可扩展性，或陷入死锁，因为其他线程可能正在等待获得锁。调用wait（）是一个更好的主意，释放对象的持有以允许其他线程运行。
28.UG: Unsynchronized get method, synchronized set method (UG_SYNC_SET_UNSYNC_GET)
这个类包含类似命名的get和set方法。在set方法是同步方法和get方法是非同步方法。这可能会导致在运行时的不正确行为，因为调用的get方法不一定返回对象一致状态。 GET方法应该同步。
29.UL: Method does not release lock on all paths (UL_UNRELEASED_LOCK)
方法获得了当前的对象所，但是在方法中始终没有释放它。一个正确的示例如下：
  Lock l = ...;
    l.lock();
    try {
        // do something
    } finally {
        l.unlock();
    }
30.UL: Method does not release lock on all exception paths (UL_UNRELEASED_LOCK_EXCEPTION_PATH)
方法获得了当前的对象所，但是在所有的异常处理中始终没有释放它。一个正确的示例如下：
  Lock l = ...;
    l.lock();
    try {
        // do something
    } finally {
        l.unlock();
    }
31.UW: Unconditional wait (UW_UNCOND_WAIT)
方法中包含调用java.lang.Object.wait（），而却没有放到条件流程控制中。该代码应确认条件尚未满足之前等待;先前任何通知将被忽略。
32.VO: A volatile reference to an array doesn't treat the array elements as volatile (VO_VOLATILE_REFERENCE_TO_ARRAY)
声明一个变量引用数组，这可能不是你想要的。如果一个变量引用数组，那么对引用数组的读和写都是不安全的，但是数组元素不是变量。取得数组的变量值你可以使用java.util.concurrent包中的数组的原子性特性
33.WL: Sychronization on getClass rather than class literal (WL_USING_GETCLASS_RATHER_THAN_CLASS_LITERAL)
实例的方法中同步this.getClass()，如果这个类有子类集合，那么子类集合中的对象将会在这个类的各个子类上进行同步，这不是我们想要的效果，我们只要同步当前的类对象而不包含它的所有子类，可以同步类名.getClass()。例如，java.awt.Label的代码：
     private static final String base = "label";
     private static int nameCounter = 0;
     String constructComponentName() {
        synchronized (getClass()) {
            return base + nameCounter++;
        }
     }
Label中的子类集合不可能在同一个子对象上进行同步，替换上面的方法为：
    private static final String base = "label";
     private static int nameCounter = 0;
     String constructComponentName() {
        synchronized (Label.class) {
            return base + nameCounter++;
        }
     }
34.WS: Class's writeObject() method is synchronized but nothing else is (WS_WRITEOBJECT_SYNC)
这个类有一个writeObject（）方法是同步的，但是这个类中没有其他的同步方法。
35.Wa: Condition.await() not in loop (WA_AWAIT_NOT_IN_LOOP)
方法没有在循环中调用java.util.concurrent.await()。如果对象是用于多种条件，打算调用wait()方法的条件可能不是实际发生的。
36.Wa: Wait not in loop (WA_NOT_IN_LOOP)
这种方法包含调用java.lang.Object.wait（），而这并不是一个循环。如果监视器用于多个条件，打算调用wait()方法的条件可能不是实际发生的。
八：Malicious codevulnerability关于恶意破坏代码相关方面的
1.EI: May expose internal representation by returning reference to mutable object (EI_EXPOSE_REP)
返回一个易变对象引用并把它保存在对象字段中时会暴露对象内部的字段描述，如果接受不守信任的代码访问或者没有检查就去改变易变对象的会涉及对象的安全和其他重要属性的安全。返回一个对象的新副本，在很多情况下更好的办法。
2.EI2: May expose internal representation by incorporating reference to mutable object (EI_EXPOSE_REP2)
此代码把外部可变对象引用存储到对象的内部表示。如果实例受到不信任的代码的访问和没有检查的变化危及对象和重要属性的安全。存储一个对象的副本，在很多情况下是更好的办法。
3.FI: Finalizer should be protected, not public (FI_PUBLIC_SHOULD_BE_PROTECTED)
一个类中的finalize（）方法必须声明为protected，而不能为public类型
4.MS: Public static method may expose internal representation by returning array (MS_EXPOSE_REP)
一个public类型的静态方法返回一个数组，可能引用内部属性的暴露。任何代码调用此方法都可以自由修改底层数组。一个解决办法是返回一个数组的副本。
5.MS: Field should be both final and package protected (MS_FINAL_PKGPROTECT)
一个静态字段可能被恶意代码或另外一个包所改变的。字段可以放到protected包中也可以定义为final类型的以避免此问题。
6.MS: Field is a mutable array (MS_MUTABLE_ARRAY)
一个定义为final类型的静态字段引用一个数组时它可以被恶意代码或在另其他包中所使用。这些代码可以自由修改数组的内容。
7.MS: Field is a mutable Hashtable (MS_MUTABLE_HASHTABLE)
一个定义为final类型的静态字段引用一个Hashtable时可以被恶意代码或者在其他包中被调用，这些方法可以修改Hashtable的值。
8.MS: Field should be moved out of an interface and made package protected (MS_OOI_PKGPROTECT)
将域尽量不要定义在接口中，并声明为包保护
在接口中定义了一个final类型的静态字段，如数组或哈希表等易变对象。这些对象可以被恶意代码或者在其他包中被调用，为了解决这个问题，需要把它定义到一个具体的实体类中并且声明为保护类型以避免这种错误。
9.MS: Field should be package protected (MS_PKGPROTECT)
一个静态字段是可以改变的恶意代码或其他的包访问修改。可以把这种类型的字段声明为final类型的以防止这种错误。
十：Dodgy关于代码运行期安全方面的
1.XSS: Servlet reflected cross site scripting vulnerability (XSS_REQUEST_PARAMETER_TO_SEND_ERROR)
在代码中在Servlet输出中直接写入一个HTTP参数，这会造成一个跨站点的脚本漏洞。
2.XSS: Servlet reflected cross site scripting vulnerability (XSS_REQUEST_PARAMETER_TO_SERVLET_WRITER)
代码直接写入参数的HTTP服务器错误页（使用HttpServletResponse.sendError）。表达了类似的不受信任的输入会引起跨站点脚本漏洞。
3.BC: Questionable cast to abstract collection (BC_BAD_CAST_TO_ABSTRACT_COLLECTION)
在代码投把一个集合强制类型转换为一个抽象的集合（如list，set或map）。保证该对象类型和将要转换的类型是一致的。如果你只是想要便利一个集合，那么你就不必将它转换为Set或List。
4.BC: Questionable cast to concrete collection (BC_BAD_CAST_TO_CONCRETE_COLLECTION)
代码把抽象的集合（如List，Set，或Collection）强制转换为具体落实类型（如一个ArrayList或HashSet）。这可能不正确，也可能使您的代码很脆弱，因为它使得难以在今后的切换指向其他具体实现。除非你有特别理由这样做，否则只需要使用抽象的集合类。
5.BC: Unchecked/unconfirmed cast (BC_UNCONFIRMED_CAST)
强制类型转换操作没有经过验证，而且不是所有的此种类型装换过的类都可以再强制类型转换为原类型。在代码中需要进行逻辑判断以保证可以进行这样的操作。
6.BC: instanceof will always return true (BC_VACUOUS_INSTANCEOF)
instanceof测试将始终返回真（除非被测试的值为空）。虽然这是安全，确保它是不是说明一些误解或其他一些逻辑错误。如果你真的想测试是空的价值，也许会更清楚这样做的更好空试验，而不是一个instanceof测试。
7.BSHIFT: Unsigned right shift cast to short/byte (ICAST_QUESTIONABLE_UNSIGNED_RIGHT_SHIFT)
无符号数右移后进行转换为short或者byte类型时可能会丢弃掉高位的值，这样的结果就是有符合数和无符号数无法区分（这取决于移位大小）
8.CI: Class is final but declares protected field (CI_CONFUSED_INHERITANCE)
这个类被声明为final的，而是字段属性却声明为保护类型的。由于是final类，它不能再被继承，而再声明为保护类型的很容易造成混淆。为了从外部能正确的使用它应该把它们声明为private或者public类型。
9.DB: Method uses the same code for two branches (DB_DUPLICATE_BRANCHES)
此方法使用相同的代码，以实现两个有条件的分支。检查以确保这是不是一个编码错误。
10.DB: Method uses the same code for two switch clauses (DB_DUPLICATE_SWITCH_CLAUSES)
他的方法使用相同的代码来实现两个switch的声明条款。这可能是重复代码的情况，但可能也显示出编码的错误。
11.DLS: Dead store to local variable (DLS_DEAD_LOCAL_STORE)
该指令为局部变量赋值，但在其后的没有对她做任何使用。通常，这表明一个错误，因为值从未使用过。
12.DLS: Useless assignment in return statement (DLS_DEAD_LOCAL_STORE_IN_RETURN)
本声明把一个局部变量放到方法的返回语句中。这对于方法中局部变量来说是没有意思的。
13.DLS: Dead store of null to local variable (DLS_DEAD_LOCAL_STORE_OF_NULL)
把一个本地变量赋值为null值，并且再也没有对这个变量做任何的操作。这样可能是为了垃圾回收，而是Java SE 6.0，这已不再需要。
14.DMI: Code contains a hard coded reference to an absolute pathname (DMI_HARDCODED_ABSOLUTE_FILENAME)
此代码包含文件对象为一个绝对路径名（例如，新的文件（“/ home/dannyc/workspace/j2ee/src/share/com/sun/enterprise/deployment”）;
15.DMI: Non serializable object written to ObjectOutput (DMI_NONSERIALIZABLE_OBJECT_WRITTEN)
代码中让一个非序列化的对象出现在ObjectOutput.writeObject()方法中，这样会引起一个错误。
16.DMI: Invocation of substring(0), which returns the original value (DMI_USELESS_SUBSTRING)
此代码调用了subString(0)方法，它将返回原来的值。
17.Eq: Class doesn't override equals in superclass (EQ_DOESNT_OVERRIDE_EQUALS)
子类定义了一个新的equals方法但是却不是覆写了父类本省的equals()方法。
18.FE: Test for floating point equality (FE_FLOATING_POINT_EQUALITY)
此操作比较两个浮点值是否相等。由于浮点运算可能会涉及到舍入，计算float和double值可能不准确。如果要求值必须准确，如货币值，可以考虑使用固定精度类型，如BigDecimal类型的值来比较
19.FS: Non-Boolean argument formatted using %b format specifier (VA_FORMAT_STRING_BAD_CONVERSION_TO_BOOLEAN)
使用%b去格式化Boolean类型的值不正确的但是它不会抛出异常，任何非空的值都会输出true，任何为空的值都会输出false
20.IC: Initialization circularity (IC_INIT_CIRCULARITY)
在引用两个相互调用为环状static方法去初始化一个实例时是错误的。
21.ICAST: integral division result cast to double or float (ICAST_IDIV_CAST_TO_DOUBLE)
整形数除法强制转换为double或者float类型。
int x = 2;
int y = 5;
// Wrong: yields result 0.0
double value1 =  x / y;
// Right: yields result 0.4
double value2 =  x / (double) y;
22.ICAST: Result of integer multiplication cast to long (ICAST_INTEGER_MULTIPLY_CAST_TO_LONG)
整形数做乘法运算结果转换为long值时如果采用
long convertDaysToMilliseconds(int days) { return 1000*3600*24*days; } 结果会因为超出整形的范围而出错。
如果使用：
long convertDaysToMilliseconds(int days) { return 1000L*3600*24*days; } 
或者：
static final long MILLISECONDS_PER_DAY = 24L*3600*1000;
	long convertDaysToMilliseconds(int days) { return days * MILLISECONDS_PER_DAY; } 
都可以避免此问题。
23.IM: Computation of average could overflow (IM_AVERAGE_COMPUTATION_COULD_OVERFLOW)
代码中使用x % 2 == 1的方法去验证运算是否存在余数的情况，但是如果出现负数的情况就不起作用了。使用x & 1 == 1, or x % 2 != 0来代替
24.INT: Vacuous comparison of integer value (INT_VACUOUS_COMPARISON)
整形数进行比较结果总是不变。例如：x <= Integer.MAX_VALUE
25.MTIA: Class extends Servlet class and uses instance variables (MTIA_SUSPECT_SERVLET_INSTANCE_FIELD)
这个类扩展从Servlet类，并使用实例的成员变量。由于只有一个Servlet类的实例，并在多线程方式使用，这种模式有可能存在问题。考虑只使用方法的局部变量。
26.MTIA: Class extends Struts Action class and uses instance variables (MTIA_SUSPECT_STRUTS_INSTANCE_FIELD)
类扩展自Struts的Action类并使用这个实例的成员变量，因为在Struts框架中只存在一个Action实例对象并且使用在多线程的情况下很可能会出现问题。
27.NP: Dereference of the result of readLine() without nullcheck (NP_DEREFERENCE_OF_READLINE_VALUE)
对readLine()的结果值没有进行判空操作就去重新赋值，这样的操作可以会抛出空指针异常。
28.NP: Immediate dereference of the result of readLine() (NP_IMMEDIATE_DEREFERENCE_OF_READLINE)
对readLine()的结果立即赋值，这样的操作可以会抛出空指针异常。
29.NP: Possible null pointer dereference due to return value of called method (NP_NULL_ON_SOME_PATH_FROM_RETURN_VALUE)
方法的返回值没有进行是否为空的检查就重新赋值，这样可能会出现空指针异常。
30.NP: Parameter must be nonnull but is marked as nullable (NP_PARAMETER_MUST_BE_NONNULL_BUT_MARKED_AS_NULLABLE)
参数值在任何情况下都不能为空，但是有明确的注释它可以为空。
31.NS: Potentially dangerous use of non-short-circuit logic (NS_DANGEROUS_NON_SHORT_CIRCUIT)
代码中使用（& or |）代替（&& or ||）操作，这会造成潜在的危险。
32.NS: Questionable use of non-short-circuit logic (NS_NON_SHORT_CIRCUIT)
代码中使用（& or |）代替（&& or ||）操作，会引起不安全的操作
33.PZLA: Consider returning a zero length array rather than null (PZLA_PREFER_ZERO_LENGTH_ARRAYS)
考虑返回一个零长度的数组，而不是null值
34.QF: Complicated, subtle or wrong increment in for-loop (QF_QUESTIONABLE_FOR_LOOP)
确定这个循环是正确的变量递增，看起来，另一个变量被初始化，检查的循环。这是由于for循环中太复杂的定义造成的。
35.RCN: Redundant comparison of non-null value to null (RCN_REDUNDANT_COMPARISON_OF_NULL_AND_NONNULL_VALUE)
方法中包含一个不能为空的赋值还包含一个可以为空的赋值。冗余比较非空值为空。
36.RCN: Redundant comparison of two null values (RCN_REDUNDANT_COMPARISON_TWO_NULL_VALUES)
方法中对两个null值进行比较
37.RCN: Redundant nullcheck of value known to be non-null (RCN_REDUNDANT_NULLCHECK_OF_NONNULL_VALUE)
方法中对不为空的值进行为空的判断。
38.REC: Exception is caught when Exception is not thrown (REC_CATCH_EXCEPTION)
在try/catch块中捕获异常，但是异常没有在try语句中抛出而RuntimeException又没有明确的被捕获
39.RI: Class implements same interface as superclass (RI_REDUNDANT_INTERFACES)
子类和父类都实现了同一个接口，这种定义是多余的。
40.RV: Method discards result of readLine after checking if it is nonnull (RV_DONT_JUST_NULL_CHECK_READLINE)
readLine方法的结果不为空时被抛弃
41.RV: Remainder of 32-bit signed random integer (RV_REM_OF_RANDOM_INT)
此代码生成一个随机的符号整数，然后计算另一个值的。由于随机数可以是负数，所以其余操作的结果也可以是负面的。考虑使用Random.nextInt（int）方法代替。
42.SA: Double assignment of local variable (SA_LOCAL_DOUBLE_ASSIGNMENT)
为一个局部变量两次赋值，这样是没有意义的。例如：
public void foo() {
    int x,y;
    x = x = 17;
  }
43.SA: Self assignment of local variable (SA_LOCAL_SELF_ASSIGNMENT)
局部变量使用自身给自己赋值
public void foo() {
    int x = 3;
    x = x;
  }
44.SF: Switch statement found where one case falls through to the next case (SF_SWITCH_FALLTHROUGH)
Switch语句中一个分支执行后又执行了下一个分支。通常case后面要跟break 或者return语句来跳出。
45.SF: Switch statement found where default case is missing (SF_SWITCH_NO_DEFAULT)
Switch没有默认情况下执行的case语句。
46.Se: private readResolve method not inherited by subclasses (SE_PRIVATE_READ_RESOLVE_NOT_INHERITED)
声明为private的序列化方法被子类继承
47.UCF: Useless control flow (UCF_USELESS_CONTROL_FLOW)
没有任何作用的条件语句。
if (argv.length == 0) {
	// TODO: handle this case
	}
48.UCF: Useless control flow to next line (UCF_USELESS_CONTROL_FLOW_NEXT_LINE)
无效的条件控制语句，注意if (argv.length == 1);以“;”结尾，下面的语句无论是否满足都会运行。
if (argv.length == 1);
        System.out.println("Hello, " + argv[0]);
49.UwF: Field not initialized in constructor (UWF_FIELD_NOT_INITIALIZED_IN_CONSTRUCTOR)
字段从来没有在任何构造函数初始化，对象被创建后值为空。如果该字段未被定义就重新赋值会产生一个空指针异常。
50.XFB: Method directly allocates a specific implementation of xml interfaces (XFB_XML_FACTORY_BYPASS)
方法自定义了一种XML接口的实现类。最好是使用官方提供的工厂类来创建这些对象，以便可以在运行期中改变。例如：
javax.xml.parsers.DocumentBuilderFactory
javax.xml.parsers.SAXParserFactory
javax.xml.transform.TransformerFactory
org.w3c.dom.Document.createXXXX


6.1、       ES_COMPARING_PARAMETER_STRING_WITH_EQ
     ES: Comparison of String parameter using == or != (ES_COMPARING_PARAMETER_STRING_WITH_EQ)

This code compares a java.lang.String parameter for reference equality using the == or != operators. Requiring callers to pass only String constants or interned strings to a method is unnecessarily fragile, and rarely leads to measurable performance gains. Consider using the equals(Object) method instead.

     使用 == 或者 != 来比较字符串或interned字符串，不会获得显著的性能提升，同时并不可靠，请考虑使用equals()方法。

6.2、       HE_EQUALS_NO_HASHCODE
     HE: Class defines equals() but not hashCode() (HE_EQUALS_NO_HASHCODE)

This class overrides equals(Object), but does not override hashCode().  Therefore, the class may violate the invariant that equal objects must have equal hashcodes.

     类定义了equals()方法但没有重写hashCode()方法，这样违背了相同对象必须具有相同的hashcodes的原则

6.3、       IT_NO_SUCH_ELEMENT
     It: Iterator next() method can't throw NoSuchElement exception (IT_NO_SUCH_ELEMENT)

This class implements the java.util.Iterator interface.  However, its next() method is not capable of throwing java.util.NoSuchElementException.  The next() method should be changed so it throws NoSuchElementException if is called when there are no more elements to return.

     迭代器Iterator无法抛出NoSuchElement异常，类实现了java.util.Iterator接口，但是next()方法无法抛出java.util.NoSuchElementException异常，因此，next()方法应该做如此修改，当被调用时，如果没有element返回，则抛出NoSuchElementException异常

6.4、       J2EE_STORE_OF_NON_SERIALIZABLE_OBJECT_INTO_SESSION
     J2EE: Store of non serializable object into HttpSession (J2EE_STORE_OF_NON_SERIALIZABLE_OBJECT_INTO_SESSION)

This code seems to be storing a non-serializable object into an HttpSession. If this session is passivated or migrated, an error will result.

     将没有实现serializable的对象放到HttpSession中，当这个session被钝化和迁移时，将会产生错误，建议放到HttpSession中的对象都实现serializable接口。

6.5、       ODR_OPEN_DATABASE_RESOURCE
     ODR: Method may fail to close database resource (ODR_OPEN_DATABASE_RESOURCE)

The method creates a database resource (such as a database connection or row set), does not assign it to any fields, pass it to other methods, or return it, and does not appear to close the object on all paths out of the method.  Failure to close database resources on all paths out of a method may result in poor performance, and could cause the application to have problems communicating with the database.

     方法可能未关闭数据库资源，未关闭数据库资源将会导致性能变差，还可能引起应用与服务器间的通讯问题。

6.6、       OS_OPEN_STREAM
     OS: Method may fail to close stream (OS_OPEN_STREAM)

The method creates an IO stream object, does not assign it to any fields, pass it to other methods that might close it, or return it, and does not appear to close the stream on all paths out of the method.  This may result in a file descriptor leak.  It is generally a good idea to use a finally block to ensure that streams are closed.

     方法可能未关闭stream，方法产生了一个IO流，却未关闭，将会导致文件描绘符的泄漏，建议使用finally block来确保io stream被关闭。

6.7、       DMI_CALLING_NEXT_FROM_HASNEXT
     DMI: hasNext method invokes next (DMI_CALLING_NEXT_FROM_HASNEXT)

The hasNext() method invokes the next() method. This is almost certainly wrong, since the hasNext() method is not supposed to change the state of the iterator, and the next method is supposed to change the state of the iterator.

6.8、       IL_INFINITE_LOOP
     IL: An apparent infinite loop (IL_INFINITE_LOOP)

This loop doesn't seem to have a way to terminate (other than by perhaps throwing an exception).

     明显的无限循环.

6.9、       IL_INFINITE_RECURSIVE_LOOP
     IL: An apparent infinite recursive loop (IL_INFINITE_RECURSIVE_LOOP)

This method unconditionally invokes itself. This would seem to indicate an infinite recursive loop that will result in a stack overflow.

     明显的无限迭代循环,将导致堆栈溢出.

6.10、   WMI_WRONG_MAP_ITERATOR
     WMI: Inefficient use of keySet iterator instead of entrySet iterator (WMI_WRONG_MAP_ITERATOR)

This method accesses the value of a Map entry, using a key that was retrieved from a keySet iterator. It is more efficient to use an iterator on the entrySet of the map, to avoid the Map.get(key) lookup.

     使用了keySet iterator和Map.get(key)来获取Map值,这种方式效率低,建议使用entrySet的iterator效率更高.

6.11、   IM_BAD_CHECK_FOR_ODD
     IM: Check for oddness that won't work for negative numbers (IM_BAD_CHECK_FOR_ODD)

The code uses x % 2 == 1 to check to see if a value is odd, but this won't work for negative numbers (e.g., (-5) % 2 == -1). If this code is intending to check for oddness, consider using x & 1 == 1, or x % 2 != 0.

     奇偶检测逻辑,未考虑负数情况.

7.实际项目中Bug类型统计

7.1、       Call to equals() comparing different types
id : EC_UNRELATED_TYPES, type : EC, category : CORRECTNESS This method calls equals(Object) on two references of different class types with no common subclasses. Therefore, the objects being compared are unlikely to be members of the same class at runtime (unless some application classes were not analyzed, or dynamic class loading can occur at runtime). According to the contract of equals(), objects of different classes should always compare as unequal; therefore, according to the contract defined by java.lang.Object.equals(Object), the result of this comparison will always be false at runtime.

原因分析：

这缺陷的意思是，大部分都是类型永远不会有这种情况， 比如a为DOUBLE类型所以EQUALS只匹配字符串 if(a.equals())或if(a.quals())这类判断是根本不会有用的；

示例：if("1".equals(DAOValue.valueofSuccess()))

7.2、       Dead store to local variable 
id: DLS_DEAD_LOCAL_STORE, type: DLS, category: STYLE

This instruction assigns a value to a local variable, but the value is not read or used in any subsequent instruction. Often, this indicates an error, because the value computed is never used.

Note that Sun's javac compiler often generates dead stores for final local variables. Because FindBugs is a bytecode-based tool, there is no easy way to eliminate these false positives.

原因分析：

DLS问题指的是给本地变量赋了一个值，但随后的代码并没有用到这个值。

7.3、       Method call passes null for nonnull parameter
id: NP_NULL_PARAM_DEREF, type: NP, category: CORRECTNESS

This method call passes a null value for a nonnull method parameter. Either the parameter is annotated as a parameter that should always be nonnull, or analysis has shown that it will always be dereferenced.

原因分析：对参数为null的情况未作处理。

例如：


7.4、       Method with Boolean return type returns explicit null  
id: NP_BOOLEAN_RETURN_NULL, type: NP, category: BAD_PRACTICE

A method that returns either Boolean.TRUE, Boolean.FALSE or null is an accident waiting to happen. This method can be invoked as though it returned a value of type boolean, and the compiler will insert automatic unboxing of the Boolean value. If a null value is returned, this will result in a NullPointerException.

原因分析：

方法如果定义为返回类型Boolean，则可以返回Boolean.TRUE, Boolean.FALSE or null （如果 return 的是 true or  false， 则AutoBoxing 成 Boolean.TRUE, Boolean.FALSE）。因为JDK 支持 基本类型和装箱类型的自动转化， 所以下面的代码中：

boolean result = test_NP_BOOLEAN_RETURN_NULL();

因为此时test_NP_BOOLEAN_RETURN_NULL() 返回的是NULL， 所以 JDK 做 automatic unboxing 的操作时， 即调用了 object. booleanValue() 方法时，抛出了空指针。

改成：boolean result = test_NP_BOOLEAN_RETURN_NULL()==null?false:true;

7.5、       No relationship between generic parameter and method argument
id: GC_UNRELATED_TYPES, type: GC, category: CORRECTNESS

This call to a generic collection method contains an argument with an incompatible class from that of the collection's parameter (i.e., the type of the argument is neither a supertype nor a subtype of the corresponding generic type argument). Therefore, it is unlikely that the collection contains any objects that are equal to the method argument used here. Most likely, the wrong value is being passed to the method.

In general, instances of two unrelated classes are not equal. For example, if the Foo and Bar classes are not related by subtyping, then an instance of Foo should not be equal to an instance of Bar. Among other issues, doing so will likely result in an equals method that is not symmetrical. For example, if you define the Foo class so that a Foo can be equal to a String, your equals method isn't symmetrical since a String can only be equal to a String.

In rare cases, people do define nonsymmetrical equals methods and still manage to make their code work. Although none of the APIs document or guarantee it, it is typically the case that if you check if a Collection<String> contains a Foo, the equals method of argument (e.g., the equals method of the Foo class) used to perform the equality checks.

原因分析：调用Collection类中的contains方法比较时，所比较的两个参数类型不致；

例如：

 

修改后：

 

7.6、       Null pointer dereference in method on exception path
id: NP_ALWAYS_NULL_EXCEPTION, type: NP, category: CORRECTNESS

A pointer which is null on an exception path is dereferenced here.  This will lead to a NullPointerException when the code is executed.  Note that because FindBugs currently does not prune infeasible exception paths, this may be a false warning.

Also note that FindBugs considers the default case of a switch statement to be an exception path, since the default case is often infeasible.

原因分析：在异常处理时，调用一个空对象的方法时可能引起空指针异常。

例如：

 

7.7、       Nullcheck of value previously dereferenced
id: RCN_REDUNDANT_NULLCHECK_WOULD_HAVE_BEEN_A_NPE, type:RCN, category: CORRECTNESS

A value is checked here to see whether it is null, but this value can't be null because it was previously dereferenced and if it were null a null pointer exception would have occurred at the earlier dereference. Essentially, this code and the previous dereference disagree as to whether this value is allowed to be null. Either the check is redundant or the previous dereference is erroneous.

原因分析：前面获取的对象，现在引用的时候没有交验是否为null。

例如：

 

7.8、       Possible null pointer dereference
id: NP_NULL_ON_SOME_PATH, type: NP, category: CORRECTNESS

There is a branch of statement that, if executed, guarantees that a null value will be dereferenced, which would generate a NullPointerException when the code is executed. Of course, the problem might be that the branch or statement is infeasible and that the null pointer exception can't ever be executed; deciding that is beyond the ability of FindBugs.

原因分析：可能存在空引用。

例如：

 

7.9、       Possible null pointer dereference in method on exception path
id: NP_NULL_ON_SOME_PATH_EXCEPTION, type: NP, category:CORRECTNESS

A reference value which is null on some exception control path is dereferenced here.  This may lead to a NullPointerException when the code is executed.  Note that because FindBugs currently does not prune infeasible exception paths, this may be a false warning.

Also note that FindBugs considers the default case of a switch statement to be an exception path, since the default case is often infeasible.

原因分析：

代码调用时， 遇到异常分支， 可能造成一个对象没有获得赋值依旧保持NULL空指针。 接下来如果对这个对象有引用， 可能造成NullPointerException 空指针异常。

例如：

7.10、   Test for floating point equality
id: FE_FLOATING_POINT_EQUALITY, type: FE, category: STYLE

This operation compares two floating point values for equality. Because floating point calculations may involve rounding, calculated float and double values may not be accurate. For values that must be precise, such as monetary values, consider using a fixed-precision type such as BigDecimal. For values that need not be precise, consider comparing for equality within some range, for example: if ( Math.abs(x - y) < .0000001 ). See the Java Language Specification, section 4.2.4.

原因分析：

Float类型的数据比较时，会存在的定的误差值，用!=来比较不是很准确，建议比较两个数的绝对值是否在一定的范围内来进行比较。如，if ( Math.abs(x - y) < .0000001 )

例如：

 

7.11、   Useless assignment in return statement
id: DLS_DEAD_LOCAL_STORE_IN_RETURN, type: DLS, category: STYLE

This statement assigns to a local variable in a return statement. This assignment has effect. Please verify that this statement does the right thing.

原因分析：

在return的对象中，没有必要通过对象赋值再进行返回。

例如：

7.12、   Write to static field from instance method
id: ST_WRITE_TO_STATIC_FROM_INSTANCE_METHOD, type: ST, category:STYLE

This instance method writes to a static field. This is tricky to get correct if multiple instances are being manipulated, and generally bad practice.

原因分析：向static字段中写入值。

例如： 
 private static DBRBO dbrBO; 
 public final void refresh() { 
        danskeBankBO = null; 
        dbrBO = null; 
        fileAndPathBO = null; 
    } 
建议改为：去掉static。

7.13、   Incorrect lazy initialization and update of static field
id: LI_LAZY_INIT_UPDATE_STATIC, type: LI, category: MT_CORRECTNESS

This method contains an unsynchronized lazy initialization of a static field. After the field is set, the object stored into that location is further updated or accessed. The setting of the field is visible to other threads as soon as it is set. If the futher accesses in the method that set the field serve to initialize the object, then you have a very seriousmultithreading bug, unless something else prevents any other thread from accessing the stored object until it is fully initialized.

Even if you feel confident that the method is never called by multiple threads, it might be better to not set the static field until the value you are setting it to is fully populated/initialized.

原因分析：

该方法的初始化中包含了一个迟缓初始化的静态变量。你的方法引用了一个静态变量，估计是类静态变量，那么多线程调用这个方法时，你的变量就会面临线程安全的问题了，除非别的东西阻止任何其他线程访问存储对象从直到它完全被初始化。

7.14、   Method ignores return value
id: RV_RETURN_VALUE_IGNORED, type: RV, category: CORRECTNESS

The return value of this method should be checked. One common cause of this warning is to invoke a method on an immutable object, thinking that it updates the object. For example, in the following code fragment,

String dateString = getHeaderField(name);

dateString.trim();

the programmer seems to be thinking that the trim() method will update the String referenced by dateString. But since Strings are immutable, the trim() function returns a new String value, which is being ignored here. The code should be corrected to:

String dateString = getHeaderField(name);

dateString = dateString.trim();

原因分析：方法忽略了设置返回值。

例如：

 

7.15、   Method might ignore exception
id: DE_MIGHT_IGNORE, type: DE, category: BAD_PRACTICE

This method might ignore an exception.Â  In general, exceptions should be handled or reported in some way, or they should be thrown out of the method.

原因分析：应该将异常 处理、打印或者抛出

例如：

 

7.16、   Unwritten field
id: UWF_UNWRITTEN_FIELD, type: UwF, category: CORRECTNESS

This field is never written.Â  All reads of it will return the default value. Check for errors (should it have been initialized?), or remove it if it is useless.


原因分析：从未被初始化的变量，调用它时，将返回默认值，要么初始化，要么删掉它。

例如：

 

7.17、   Value is null and guaranteed to be dereferenced on exception path
id: NP_GUARANTEED_DEREF_ON_EXCEPTION_PATH, type: NP, category:CORRECTNESS

There is a statement or branch on an exception path that if executed guarantees that a value is null at this point, and that value that is guaranteed to be dereferenced (except on forward paths involving runtime exceptions).

原因分析：exception分支上，存在引用一个null对象的方法，引发空指针异常。

例如：

7.18、   Very confusing method names
id: NM_VERY_CONFUSING, type: Nm, category: CORRECTNESS

The referenced methods have names that differ only by capitalization. This is very confusing because if the capitalization were identical then one of the methods would override the other.

原因分析：被引用的方法中存在容易混淆的变量。

例如：fzgsdm改成 fzgsDm 即可。

 

7.19、   Method invokes inefficient new String() constructor
id: DM_STRING_VOID_CTOR, type: Dm, category: Performance Creating a new java.lang.String object using the no-argument constructor wastes memory because the object so created will be functionally indistinguishable from the empty string constant "".  Java guarantees that identical string constants will be represented by the same String object.  Therefore, you should just use the empty string constant directly.

原因分析：不使用new String()定义空的字符串

例如：

        

7.20、   Load of known null value
id: NP_LOAD_OF_KNOWN_NULL_VALUE, type: Np, category: Dodgy

The variable referenced at this point is known to be null due to an earlier check against null. Although this is valid, it might be a mistake (perhaps you intended to refer to a different variable, or perhaps the earlier check to see if the variable is null should have been a check to see if it was nonnull).

原因分析：null值的不当使用。

例如：

 

7.21、   Method concatenates strings using + in a loop  
id: SBSC_USE_STRINGBUFFER_CONCATENATION, type: SBSC, category: Performance

The method seems to be building a String using concatenation in a loop. In each iteration, the String is converted to a StringBuffer/StringBuilder, appended to, and converted back to a String. This can lead to a cost quadratic in the number of iterations, as the growing string is recopied in each iteration. Better performance can be obtained by using a StringBuffer (or StringBuilder in Java 1.5) explicitly.

For example:

  // This is bad

  String s = "";

  for (int i = 0; i < field.length; ++i) {

    s = s + field[i];

  }

  // This is better

  StringBuffer buf = new StringBuffer();

  for (int i = 0; i < field.length; ++i) {

    buf.append(field[i]);

  }

  String s = buf.toString();

原因分析：在循环里使用字符串连接，效率低，应该使用StringBuilder/StringBuffer



findbugs 错误分析日志 findbugs 出错类型及对应解释
终于 我们又开始使用FINDBUGS来检测代码的问题了 但因没又中文版和网上错误分析实际很少 所以自己边处理BUG边写文档
   首先在装好FINDBUGS后 在 project-->properteis-->findbugs里的2个框的勾点上可以让改正BUG后自动判断是否改正了 改正了就自动去掉BUG点
   
1、Dead store to local variable 本地变量存储了闲置不用的对象
举例：
List accountCoList = new ArrayList();
我们为accountCoList新建了一个对象，但是程序的后面并没有使用这个这个新建对象。
建议改为：
List accountCoList = null;
 
This instruction assigns a value to a local variable, but the value is not read or used in any subsequent 
instruction. Often, this indicates an error, because the value computed is never used.
Note that Sun's javac compiler often generates dead stores for final local variables. Because FindBugs is 
a bytecode-based tool, there is no easy way to eliminate these false positives.
本地变量存储了闲置不用的对象，也就是说这个变量是多余的。
Hashtable   hTable = new  Hashtable();   
Object   obj = new   Object();  
obj =  hTable.put("uuid",   "abcd1234");  
 String abc = "abc";
 String xyz = new String("");
 xyz = abc;
 System.out.println(xyz); 
用findbug檢查會出現Dead store to local variable的錯誤，他的意思是“本地变量存储了闲置不用的对象”
為什么會出現這個原因呢？ 因為 String xyz = new String("");
  这一句执行3个动作：   
  1)创建一个引用xyz   
  2)创建一个String对象   
  3)把String的引用赋值给xyz
  其中，后面两个动作是多余的，因为后面的程序中你没有使用这个新建的String对象，而是重新给xyz赋值，  
xyz = abc;所以，只需要String xyz = abc; 就可以了。这样，findbugs就不会报了。


2、Write to static field from instance method 向static字段中写入值
举例：
 private static DBRBO dbrBO;
 public final void refresh() {
        danskeBankBO = null;
        dbrBO = null;
        fileAndPathBO = null;
    }
建议改为：
去掉static。
 
This instance method writes to a static field. This is tricky to get correct if multiple instances are 
being manipulated, and generally bad practice.
向static字段中写入值，如：


private static Logger logger;
 public XXXActionCtrl(){
  logger = Logger.getLogger(getClass());
 }
可改为：private static Logger logger = Logger.getLogger(getClass());Unread field: should this field be static?
This class contains an instance final field that is initialized to a compile-time static value. Consider 


3、Load of known null value 大体意思是加载了null的对象。
举例
        if (null == boList) {
            for (int i = 0; i < boList.size(); i++) {
                entityList.add(productBOToEntity(boList.get(i)));
            }
        }

4、Exception is caught when Exception is not thrown 
这个意思比较好理解：就是catch了异常但是try里并没有抛出异常
    异常被捕获但没抛出。。。。
     一般人都会这样写代码：
　　try{
　　　　//
　　}
　　catch(Exception ex){
　　　　//
　　}
    这样很省事，但是JAVA规范中并不推荐这样做，这样是属于“过泛地捕获异常”，因为try{}中可能出现的异常种类有很多，上面的做法不利于分别处理各种异常，
建议根     据业务需求，分别捕获需要特别处理的异常，例子如下：
　　try{
　　　　//
　　}
　　catch(SQLException ex){
　　　　//
　　}
　　catch(IOException ex){
　　　　//
　　}
　　catch(Exception ex){
　　　　//
　　}
    另外一个是，捕获到的Exception应尽量记录到LOG文件里。 

5、Method ignores exceptional return value 
没有对方法的异常返回值进行检查


6、Comparison of String objects using == or !=
This code compares java.lang.String objects for reference equality using the == or != operators.
Unless both strings are either constants in a source file, or have been interned using the String.intern() method,
 the same string value may be represented by two different String objects. Consider using the equals(Object) method
  instead.
  从字面意思可以理解String对象进行比较的时候：只有两种情况可以使用== or !=的，这两种情况是；在源文件中是个常数或者是调用
  String.intern()方法，使用String的规范化表示形式来进行比较,如果不是这两中情况的话推荐使用.equals(object)方式
  
7、Method names should start with a lower case letter 
这个好理解方法名的第一个字母不能是大写 函数的首字母应该小写。


8、Non-transient non-serializable instance field in serializable class
This Serializable class defines a non-primitive instance field which is neither transient, Serializable,
 or java.lang.Object, and does not appear to implement the Externalizable interface or the readObject()
  and writeObject() methods.? Objects of this class will not be deserialized correctly if a non-Serializable object
   is stored in this field.
这个错误的意思是：在可序列化的类中存在不能序列化或者不能暂存的数据
在可序列化的类中存在不能序列化或者不能暂存的数据


9.Call to equals() comparing different types
    大部分都是类型永远不会有这种情况 比如a为DOUBLE类型所以EQUALS只匹配字符串 if(a.equals())或if(a.quals())这类判断是根本不会有用的的
equals比较了不同的对象类型 说的是equals要比较相同的对象类型
This method calls equals(Object) on two references of different class types with no common subclasses. 
Therefore, the objects being compared are unlikely to be members of the same class at runtime (unless some 
application classes were not analyzed, or dynamic class loading can occur at runtime). According to the 
contract of equals(), objects of different classes should always compare as unequal; therefore, according 
to the contract defined by java.lang.Object.equals(Object), the result of this comparison will always be 
false at runtime.
调用equals()比较不同的类型。
此方法调用相当于两个不同的类类型的引用，没有共同的子类（对象）。
因此，所比较的对象是不太可能在运行时相同的类成员（除非一些
应用类没有分析或动态类加载可以发生在运行时）。据
equals()的规则，不同类的对象应始终比较不平等，因此，根据
由java.lang.Object.equals定义的合同（对象），FALSE将永远是比较的结果
在运行时错误。
 
10.Class doesn't override equals in superclass
  super.equals(obj) 调用父类equals方法 一般都是Object的方法，所以这个super可写可不写，一般都是 为了代码的可读性才加上去的
    一般就是重写equals(obj)即可 即public boolean equals(Object obj){ return super.equals(obj);}
    但是如果覆盖了equals()方法的话，则必须要覆盖hashCode()方法。否则FINDBUGS会出现下面的7号BUG:覆盖了equals()方法的话，则必须要覆盖hashCode()方法
    所以 public boolean equals(Object obj){ return super.equals(obj);} 
         public int hashCode(){
   return super.hashCode();
}


11.Class is Serializable, but doesn't define serialVersionUID
    serialVersionUID 用来表明类的不同版本间的兼容性
    简单来说，Java的序列化机制是通过在运行时判断类的serialVersionUID来验证版本一致性的。在进行反序列化时，
JVM会把传来的字节流中的serialVersionUID与本地      相应实体（类）的serialVersionUID进行比较，如果相同就认为是一致的，
可以进行反序列化，否则就会出现序列化版本不一致的异常。
    当实现java.io.Serializable接口的实体（类）没有显式地定义一个名为serialVersionUID，类型为long的变量时，
Java序列化机制会根据编译的class自动生成一个       serialVersionUID作序列化版本比较用，这种情况下，
只有同一次编译生成的class才会生成相同的serialVersionUID 。
    如果我们不希望通过编译来强制划分软件版本，即实现序列化接口的实体能够兼容先前版本，未作更改的类，
就需要显式地定义一个名为serialVersionUID，类型为long     的变量，不修改这个变量值的序列化实体都可以相互进行串行化和反串行化。
    也就是这个错误 你要定义一个名为 serialVersionUID，类型为long的变量 按照新版Eclipse自动填写规则 就是：
     private static final long serialVersionUID = 1L;
 
12.Class names shouldn't shadow simple name of superclass
   基本就是这个类的名字跟超类的名字一样但不在一个包里 所以就改下类名啦
   
13.Comparison of String parameter using == or !=
   原因：当比较两个字符串内容是否相同时，仅当两个字符串在源文件中都是常量时或者是使用intern()来比较才可以用==来比较，
   否则最好使用对象比较方法equal。附string比较：
    String str1 = "java";
    String str2 = "java";
    System.out.print(str1==str2);
    结果：true(二者都为常量)
    String str1 = new String("java");
    String str2 = new String("java");
    System.out.print(str1==str2);
    结果：false（二者为对象）
    String str1 = "java";
    String str2 = "blog";
    String s = str1+str2;
    System.out.print(s=="javablog");
    结果：false(s不为常量，为对象)
    String s1 = "java";
    String s2 = new String("java");
    System.out.print(s1.intern()==s2.intern());
    结果：true(但是intern（）方法在效率和实现方式上不统一)


14.Equals checks for noncompatible operand
    equals()方法比较的是值是否相同，而不是内存指向地址
   就实际情况来看 是因为
   public boolean equals(Object object) {
   if (!(object instanceof DmZzmm)) {
    return false;
   }
         Dxinfo rhs = (Dxinfo) object;
    return new EqualsBuilder().append(this.dxcount, rhs.dxcount).append(this.lxrsjh, rhs.lxrsjh)
      .append(this.dxnr, rhs.dxnr).append(this.fssj, rhs.fssj).append(this.fssl, rhs.fssl)
      .append(this.id,rhs.id).isEquals();
            。。。。
      } 
      问题在那里？很简单 这个白痴是拷贝的代码 既然object 不为DmZzmm就为false 
 而你要执行的是 Dxinfo rhs = (Dxinfo) object; 所以如果为DMZzmm进入代码
 会因为不是 Dxinfo 类型而不执行下面的代码 如果为Dxinfo 类型 它就直接执行为FALSE!!所以代码毫无意义 只会执行false!!
      所以只需要把
      public boolean equals(Object object) {
   if (object instanceof Dxinfo) {
    Dxinfo rhs = (Dxinfo) object;
    。。。。。。。
   }else{
    return false;
   }
 就可以了 说穿了 就是你准备用instanceof 匹配的类要跟你下面执行的类要一致。。。。

15.equals method always returns false
   equals始终返回false
      嘛。。。就是没继承超类的话要自己写个EQUALS。。。。别写的没意义只有false就是了。。
      public boolean equals(Object o) {
   return (this==o);
}


16.equals() method does not check for null argument
   equals（）方法没检查空参数
      2B错误完全不解释
 
17.Field names should start with a lower case letter
    字段名应该用小写

18.Can't close pw since it is always null
    无法关闭【PW】因为总是为NULL
 
19.AM: Creates an empty jar file entry (AM_CREATES_EMPTY_JAR_FILE_ENTRY)/AM:
 Creates an empty zip file entry (AM_CREATES_EMPTY_ZIP_FILE_ENTRY)
示例代码： 
ZipEntry entry = new ZipEntry(PATH);
zos.putNextEntry(entry);
zos.closeEntry();
原因：
代码中在调用putNextEntry()之后紧接着调用了closeEntry()函数，致使该jar文件内容为空，
因为打jar包的写内容是在putNextEntry()和closeEntry()两个函数调用之间来进行的。（有时候也许会有意的构建一个空目录，因此不一定就是bug）


20.BC: Equals method should not assume anything about the type of its argument (BC_EQUALS_METHOD_SHOULD_WORK_FOR_ALL_OBJECTS)
示例代码：
public class Foo {
   // some code
   public void equals(Object o) {
     Foo other = (Foo) o;
    // the real equals code
  }
}
原因：
当你在实现类的equals方法时，不应该对参数有任何的预先设定。如上代码所写，
则设定了参数o肯定是Foo类的一个对象.但是如果在函数调用时，参数o不是一个Foo类或其子类，
就会导致代码会抛出一个ClassCastException。因此在实现equals方法，应该加一个判断，如果参数o不是一个Foo类对象，则返回false。


21.BC: Random object created and used only once (DMI_RANDOM_USED_ONLY_ONCE)
示例代码：
public int getRandom(int seed) {
    return new Random(seed).nextInt();
}
原因：
由于java.util.Random是一个伪随机函数，如果传入的seed值相同的话，返回的随机数者是相同的 。
因此没必要每次都new一个新的random出来计算随机数。如果你想真正地获得一个不可预知的随机数，
建议使用java.security.SecureRandom，该类继承自Random，是一个强随机数生成器 。因此上述代码可以修改为：
public class Test  extends Thread{
    private SecureRandom ran;
    Test(int seed){
        ran = new SecureRandom();
    }


    public int getRandom(int seed) {
        return ran.nextInt();
    }
}


22.CN: Class implements Cloneable but does not define or use clone method (CN_IDIOM)
示例代码：
public class Foo implements Cloneable {
         public Object clone() throws CloneNotSupportedException {
                  return super.clone();
         }


}
原因：
类定义要实现了 Cloneable接口，却没有定义或使用 clone方法，即缺少红色字体部分。
 
23.CN: clone method does not call super.clone() (CN_IDIOM_NO_SUPER_CALL)
示例代码：
public class Foo implements Cloneable {
         public Object clone() throws CloneNotSupportedException {
                   return super.clone();
         }
}
原因：
clone方法没有调用super.clone()方法，如果没有调用，则会导致对象父子层级关系不能正确建立，最终导致无法正确组装对象。


24.CN: Class defines clone() but doesn't implement Cloneable (CN_IMPLEMENTS_CLONE_BUT_NOT_CLONEABLE)
示例代码：
public class Foo{
         public Object clone() throws CloneNotSupportedException {
                   return super.clone();
         }
}
原因：
这个用法的意义在于你可以规范该类的子类的clone的实现，如果你的确想这样做的话，这不是一个bug,否则的话是一个bug


25.DE: Method might drop exception (DE_MIGHT_DROP)/DE: Method might ignore exception (DE_MIGHT_IGNORE)
示例代码：
try{}catch(Exception ex){}
原因：
方法有可能抛异常或者忽略异常，需要对异常进行处理,即需要在catch体中对异常进行处理。


8.DMI: Don't use removeAll to clear a collection (DMI_USING_REMOVEALL_TO_CLEAR_COLLECTION)
原因：
建议不要使用 collection.removeAll(collection)方法来删除 collection中的所有元素，而使用collection.clear()。比较二者的代码实现就可以看出：
removeAll()源码：
    public boolean removeAll(Collection<?> c) {
              boolean modified = false;
              Iterator<?> e = iterator();
              while (e.hasNext()) {
                    if (c.contains(e.next())) {
                                e.remove();
                                 modified = true;
                  }
              }
              return modified;
    }
clear()源码：
    public void clear() {
              Iterator<E> e = iterator();
              while (e.hasNext()) {
                  e.next();
                  e.remove();
              }
    }
前者是比较参数中的collection和要移除元素的collection中是否有交集，然后将交集元素删除；
后者是直接将collenction中的元素删除。显然后者要比前者高效，而且对于某些特殊的collenction还容易抛出一些异常，
如ConcurrentModificationException


26.May expose internal representation by incorporating reference to mutable object
    JAVA里，对象是引用传递的，setObj的时候，对象不要直接赋值(this.regDate = regDate)，可改为：this.regDate = (Date)regDate.clone();，
 
27.Uninitialized read of field in constructor
      构造函数没有初始化;


28.Method concatenates strings using + in a loop
The method seems to be building a String using concatenation in a loop. In each iteration, the String is 
converted to a StringBuffer/StringBuilder, appended to, and converted back to a String. This can lead to a 
cost quadratic in the number of iterations, as the growing string is recopied in each iteration.
Better performance can be obtained by using a StringBuffer (or StringBuilder in Java 1.5) explicitly.
字符串串联使用方法在一个循环+
该方法似乎是建立在循环使用字符串串联。在每次迭代中，字符串转换为一个StringBuffer / StringBuilder的，
附加到 并转换回为String。这可能导致成本的二次迭代，因为不断增长的字符串是在每次迭代中重新复制。
更好的性能，可使用StringBuffer（或StringBuilder的）会更好一些。
For example:
  // This is bad
  String s = "";
  for (int i = 0; i < field.length; ++i) {
    s = s + field[i];
  }
  // This is better
  StringBuffer buf = new StringBuffer();
  for (int i = 0; i < field.length; ++i) {
    buf.append(field[i]);
  }
  String s = buf.toString();


29.
May expose internal representation by incorporating reference to mutable   
object.This code stores a reference to an externally mutable   
object into the internal representation of the object.If instances  
are accessed by untrusted code,and unchecked changes to the mutable object would compromise security or 
other important properties,you will need to do something different.  
Storing a copy of the object is better approach in many situations.
可能因使引用可指向多个对象而暴露内部存储结构。
这代码使一个指向外部多个对象的引用指向了一个内部对象存储地址。  
如果实例被未被信任代码访问或多个对象发生了未经检查的改变就会危及安全性或其它重要属性，
你需要去做一些不同的事情。存储一个对象的拷贝在许多情况下会是一个更好的方法。
网上findbug使用的介绍文章中写到,按下面修改findbug就没bug提示了,
为什么要放到一个临时变量中就可以了?
public class Test {
private String[] name;
public String[] getName() {
String[] temp = name;
return temp;
}public void setName(String[] name) {
String[] temp = name;
this.name = temp;
}
}
因为代码中会经常出现getter/setter，我觉得这个bug是不必要进行修改的。


30.
making the field static.
未读的领域：这个领域应该是静态的？
这个类包含一个实例的最后字段初始化为编译时静态值。考虑静态的领域。（findbugs建议修改该属性为static的）。
如：private final String FAIL_FLAG = "exit";
改为：private static final String FAIL_FLAG = "exit";


31.
Unread field
This field is never read.  Consider removing it from the class. 
未读的领域（字段，属性）
类中声明了从未用过的字段。考虑从类中删除。


32.
Inefficient use of keySet iterator instead of entrySet iterator
This method accesses the value of a Map entry, using a key that was retrieved from a keySet iterator. 
It is more efficient to use an iterator on the entrySet of the map, to avoid the Map.get(key) lookup.
低效利用，使用keySet迭代器而不是entrySet迭代器。
使用entrySet效率会比keySet高。
keySet()迭代后只能通过get()取key。
entrySet()迭代后可以e.getKey()，e.getValue()取key和value，返回的是Entry接口。


33.
Field isn't final but should be.
A mutable static field could be changed by malicious code or by accident from another package. The field 
could be made final to avoid this vulnerability.
字段应该声明为final，实际上却未声明final。
一个易变的static字段可以被恶意代码改变，使用final关键字以避免此漏洞。


34.
Call to equals() with null argument.
This method calls equals(Object), passing a null value as the argument. 
According to the contract of the equals() method, this call should always return false.
使用null参数调用equals()。
此方法调用等于（对象），作为参数传递一个空值。根据合同的equals（）方法，此调用应始终返回false。
如：queryStr.equals(null); 这样使用是不可取的，虽然能够通过编译。


35.
Invocation of toString on an array.
The code invokes toString on an array, which will generate a fairly useless result such as [C@16f0472. 
Consider using Arrays.toString to convert the array into a readable String that gives the contents of the array. 
See Programming Puzzlers, chapter 3, puzzle 12.
对数组调用toString()方法。
代码在对数组调用toString()方法时，将产生一个相当无用的形如 [C@16f0472 的结果。考虑使用 Arrays.toString方法


将数组转化为一个可读的给出数组内容的字符串。
比如：在使用System.out.println(xx.readNext());时候会碰到这样的提示，readNext() 方法放回一个String[]。
可改为：
String[] arr = reader.readNext();
System.out.println(Arrays.asList(arr).toString());
（好像有点麻烦，没想到更简洁的办法）。


36.
Nullcheck of value previously dereferenced
A value is checked here to see whether it is null, but this value can't be null because it was previously 
dereferenced and if it were null a null pointer exception would have occurred at the earlier dereference. 
Essentially, this code and the previous dereference disagree as to whether this value is allowed to be 
null. Either the check is redundant or the previous dereference is erroneous.
此代码之前废弃null值检查。
一个值被选中这里看它是否是空的，但这个值不能为空，因为它在此之前废弃null值检查，而且如果它为null，空指针异
常将会发生在此处，如果它是空一空指针异常会发生在较早取消引用。
从本质上讲，此代码和前边废弃的null值检查将会出现不一致，是否这个值是容许
空。
出现该bug有两种情况：多余的null检查；前边废弃null值检查的。
比如：我们经常会这个使用ActionForm，
String clazzId = request.getParameter("clazzId");// script1
studentForm.setClazzID(clazzId);// script2
往往会在script2会出现该错误，因为在script1出未检查clazzId是否为null才导致的。
修改为 ： 
if(clazzId != null) {
studentForm.setClazzID(clazzId);
}
在设置使用clazzId之前先判断其是否为null。


37.
Possible null pointer dereference in method on exception path
A reference value which is null on some exception control path is dereferenced here.  This may lead to a NullPointerException when the code is executed.  Note that because FindBugs currently does not prune infeasible exception paths, this may be a false warning.
Also note that FindBugs considers the default case of a switch statement to be an exception path, since the default case is often infeasible.
在异常部分放弃null值检查，可能会导致后面的代码出现空指针异常。如：
md = null;
try {
   md = MessageDigest.getInstance("SHA-256");
   md.update(bt);
  } catch (NoSuchAlgorithmException e) {
   e.printStackTrace();// script1
  }
  byte[] digest = md.digest();// script2
bug出现在script2处，在script1处处理相应的exception即可，如throw 或 return； 


Possible null pointer dereference
There is a branch of statement that, if executed, guarantees that a null value will be dereferenced, which would generate a NullPointerException when the code is executed. Of course, the problem might be that the branch or statement is infeasible and that the null pointer exception can't ever be executed; deciding that is beyond the ability of FindBugs.
可能的空指针引用。
如在JDBC编程时候，在关闭ResultSet时候(rs.close())，经常会出现这个bug，解决办法很容易想到，判断是否为null或


使用try...catch...finally。


---------------------------------------------------------------------
    
相关资料：
 
1：hyddd的FindBugs分析记录
用FindBugs分析代码漏洞 hyddd  阅读:6139 评论:13  
http://www.cnblogs.com/hyddd/tag/hyddd%E7%9A%84FindBugs%E5%88%86%E6%9E%90%E8%AE%B0%E5%BD%95/ 


1.[hyddd的FindBugs分析记录][H STCAL] Call to static DateFormat 
上面的英文解释其实应该说得比较清楚，在Java文档中，已经明确说明了DateFormats 是非线程安全的，
而在SimpleDateFormat的Jdk 的Source文件中，我们也找到这么一段注释，说明它不是线程安全的。  
导致SimpleDateFormat出现多线程安全问题的原因，是因为：SimpleDateFormat处理复杂，Jdk的实现中使用了成员变量来传递参数，这就造成在多线程的时候会出现错误。 
而Findbugs所说的“Call to static DateFormat”，其实就是一些人：
为了渐少new 的次数而把SimpleDateFormat做成成员或者静态成员，上面已经说了，这样做是不安全的。


2.[hyddd的FindBugs分析记录][M S XSS] Class defines clone() but doesn't implement Cloneable 
[H B CN] Class defines clone() but doesn't implement Cloneable [CN_IMPLEMENTS_CLONE_BUT_NOT_CLONEABLE] 
This class defines a clone() method but the class doesn't implement Cloneable. 
There are some situations in which this is OK (e.g., you want to control how subclasses can clone themselves), 
but just make sure that this is what you intended. 
　　什么代码会引起这个问题呢？先看下面： 
  1: class MyTest {
  2: public MyTest clone(){
  3: MyTest test = new MyTest();
  4: return test;
  5: }
  6: }
这段代码会引起FindBugs的这个警告，为什么？请看下面解释：
1.根据FindBugs的说明，如果一个类重写clone()函数，而不继承Cloneable接口，是一定有问题的，如果clone()方法只是简单进行克隆，
如：new一个对象并初始化，然后返回这个新创建的对象的话，不继承Cloneable接口也是可以的。
2.如果是上面这样的话，为什么还要继承Cloneable接口呢？稍微说一下，Cloneable接口是不包含任何方法的！
其实这个接口仅仅是一个标志，而且这个标志也仅仅是针对Object类中clone()方法的，如果clone类没有实现 Cloneable接口，
并调用了Object的clone()方法（也就是调用了super.Clone()方法），那么Object的clone() 方法就会抛出CloneNotSupportedException异常。
3.所以这里建议是：规范写法，如果重写clone()，最好请继承Cloneable接口。


3.[hyddd的FindBugs分析记录][M S XSS] Servlet reflected cross site scripting vulnerability 
[M S XSS] Servlet reflected cross site scripting vulnerability [XSS_REQUEST_PARAMETER_TO_SERVLET_WRITER]
This code directly writes an HTTP parameter to Servlet output, which allows for a reflected cross site scripting vulnerability. 
See http://en.wikipedia.org/wiki/Cross-site_scripting for more information.
FindBugs looks only for the most blatant, obvious cases of cross site scripting. If FindBugs found any, 
you almost certainly have more cross site scripting vulnerabilities that FindBugs doesn't report. If you are concerned about cross site scripting, you should seriously consider using a commercial static analysis or pen-testing tool.
先看下面代码：
public void doGet(HttpServletRequest request,HttpServletResponse response)throws ServletException,IOException{
　　//
　　String v = request.getParameter("v");
　　//
　　PrintWriter out = response.getWriter();
　　out.print("协议版本号不对,v="+v);
　　out.close();
　　//
}
这里字符串v没有作过滤，直接返回给用户，有可能操作XSS攻击。具体关于XSS攻击的资料，可以参考上面Findbugs说明中的连接，这里就不多说了。


4.[hyddd的FindBugs分析记录]M D ICAST] Result of integer multiplication cast to long 
[M D ICAST] Result of integer multiplication cast to long [ICAST_INTEGER_MULTIPLY_CAST_TO_LONG]
This code performs integer multiply and then converts the result to a long, as in: 
  long convertDaysToMilliseconds(int days) { return 1000*3600*24*days; } 
If the multiplication is done using long arithmetic, you can avoid the possibility that the result will overflow. 
For example, you could fix the above code to: 
long convertDaysToMilliseconds(int days) { return 1000L*3600*24*days; } 
or 
static final long MILLISECONDS_PER_DAY = 24L*3600*1000; long convertDaysToMilliseconds(int days) { return days * MILLISECONDS_PER_DAY; } 
其实看上面的例子可以看到了
long convertDaysToMilliseconds(int days) { return 1000*3600*24*days; }  
这段代码是有可能溢出的，建议把代码改为下面：
long convertDaysToMilliseconds(int days) { return 1000L*3600*24*days; } 
用过VB6的人应该了解为什么会这样，因为在VB6里面一不小心就有可能出现这种溢出，在这里，JAVA认为：int * int *
 .....它的结果还是一个int！为什么编译器不能自动识别呢？答案是：很麻烦，并且这会使用编译器的效率非常低！
 对于有汇编经验的人来说，应该了解用汇编实现动态判断一个结果应该分配到一个long的空间还是int的空间有多复杂吧，
 而编译器有不能随便随便把int * int 的结果分配一个long空间，因为这会非常浪费内存，所以出于对效率和资源的考虑，
 最后的决定就是，凡是int * int * ....这样的计算结果一律都存放到int空间里，如果是long * int * .....则一律存放到long空间里，
 这就解释了为什么上面要在1000后面加个"L"了。:>


[hyddd的FindBugs分析记录][M B DE] Method might ignore exception  
This method might ignore an exception.  In general, exceptions should be handled or reported in some way, 
or they should be thrown out of the method.
try{
//
}
Catch(Execption ex){}
//
上面这段代码没有对ex进行任何处理。Findbugs在这里想说明的就是上面这个问题。


5.[hyddd的FindBugs分析记录][M M IS] Inconsistent synchronization追加说明 


6.[hyddd的FindBugs分析记录][M B Eq] Class defines compareTo(...) and uses Object.equals() 
重写compareTo有一定的风险，因为你不知道JDK内部做对象对比时，到底使用了compareTo还是equals。
例如：在JAVA5 里，PriorityQueue.remove中使用了compareTo，但JAVA6中，PriorityQueue.remove使用了equals方法。
　　这里Findbugs强烈建议：(x.compareTo(y)==0) == (x.equals(y))，当你重写compareTo的时候，请记得这一点:>，当然这个只是建议，不是绝对的。


7.[hyddd的FindBugs分析记录][H C EC] equals() used to compare array and nonarray 
This method invokes the .equals(Object o) to compare an array and a reference that doesn't seem to be an array. 
If things being compared are of different types, they are guaranteed to be unequal and the comparison is almost certainly an error. 
Even if they are both arrays, the equals method on arrays only determines of the two arrays are the same object. 
To compare the contents of the arrays, use java.util.Arrays.equals(Object[], Object[]). 
先看下面一段代码：
String[] strs = {"1"};
//.
If("1".equals(strs)){
//todo
}
上面这段代码，if里面的代码是永远都不会被执行的，因为我们用了一个非数据（nonarray）和一个数组(array)做equals()，这里估计是写代码时的笔误。


8.[hyddd的FindBugs分析记录][H C FS] Format string references missing argument 
Not enough arguments are passed to satisfy a placeholder in the format string. 
A runtime exception will occur when this statement is executed.
看实例代码：
public static void main(String args[]) throws Exception{
    String sqlrightDate = "select state from right where user_id = '%s' and soft_code ='%s' and right_name = '%s' ";
    String str = String.format(sqlrightDate,"1" ,"2");
    System.out.println(str);
}
对，你没有眼花，format里面只传了两个参数，但sqlrightData需要的是三个参数！这可能值是你的一个笔误，
相信没人会故意写这样的代码，但编译器检查不出这种问题，这导致运行时这里会抛异常，所以用FindBugs扫扫代码还是有点用处的～！


9.[hyddd的FindBugs分析记录][M B Nm] Class names should start with an upper case letter 
JAVA里，类的首字母需要大写，这个不多说了。


10.[hyddd的FindBugs分析记录][H B BC] Random object created and used only once 
1.new一个Random的对象，保存之，然后每次都使用这个对象去获取随机数，而不要每次new一个Random对象去获取。
2.FindBugs强烈推荐使用java.security.SecureRandom代替Random。


11.[hyddd的FindBugs分析记录][M D REC] Exception is caught when Exception is not thrown 
略 上有


12.[hyddd的FindBugs分析记录][M D DLS] Dead store to local variable  
略 上有


13.[hyddd的FindBugs分析记录][M P UuF] Unused field 
说明某个类里的某个变量没有被使用。FindBugs建议你把无用东西去除掉。


14.[hyddd的FindBugs分析记录][M B ODR] Method may fail to close database resource 
请参考：[M X OBL] Method may fail to clean up stream or resource
注意：同一个问题可能引发两个不同的BUG警告～！


15.[hyddd的FindBugs分析记录][M X OBL] Method may fail to clean up stream or resource  
[M X OBL] Method may fail to clean up stream or resource [OBL_UNSATISFIED_OBLIGATION]
This method may fail to clean up (close, dispose of) a stream, database object, or other resource requiring an explicit cleanup operation.
In general, if a method opens a stream or other resource, the method should use a try/finally block to ensure that the stream
 or resource is cleaned up before the method returns. 
This bug pattern is essentially the same as the OS_OPEN_STREAM and ODR_OPEN_DATABASE_RESOURCE bug patterns, 
but is based on a different (and hopefully better) static analysis technique. We are interested is getting feedback about the 
usefulness of this bug pattern. To send feedback, either: 
send email to findbugs@cs.umd.edu 
file a bug report: http://findbugs.sourceforge.net/reportingBugs.html 
In particular, the false-positive suppression heuristics for this bug pattern have not been extensively tuned, 
so reports about false positives are helpful to us. 
See Weimer and Necula, Finding and Preventing Run-Time Error Handling Mistakes, for a description of the analysis technique.
这个BUG想说明的是：有些资源打开了，但在函数结束的时候却没有关闭。比如：数据库连接......
虽然JAVA有垃圾回收机制，但是，自己打开的资源需要自己手动关闭，否则有可能直到程序退出，JRE才会清理你打开的资源。
这里FindBugs建议在try/finally里面关闭打开的资源，在关闭之前，还要判断资源是否为Null，或者再加一层异常捕获处理噢～
以下是一些可能关闭资源失败的例子
-----------------------------------------------情况1-----------------------------------------
//
FileOutputStream fs = null;
try{
　　fs = new FileOutputStream(clearTableFile);
　　fs.close();    //当出异常时候fs关闭失败，所以应该在finally中关闭
}
catch(){
//
}
-----------------------------------------------情况2-----------------------------------------
//
Properties props = new Properties();
try{
　　props.store(new FileOutputStream(configFile), configFile);　　//new FileOutputStream(configFile)没有释放。
}
catch(){
　　//
}
-----------------------------------------------情况3-----------------------------------------
　　//
　　FileOutputStream fs = new FileOutputStream(clearTableFile);
　　//    (里没有做异常处理，如果中间出异常了，异常会抛往上层，这时fs就没能释放了。
　　fs.close();    
　　//


16.[hyddd的FindBugs分析记录][M M NP] Synchronize and null check on the same field   
 [M M NP] Synchronize and null check on the same field. [NP_SYNC_AND_NULL_CHECK_FIELD]
Since the field is synchronized on, it seems not likely to be null. If it is null and then synchronized on a 
NullPointerException will be thrown and the check would be pointless. Better to synchronize on another field.
先看一段代码：
public static Timestamp getTableLastUpdateTime(String tableName) {
    Timestamp time = null;
    synchronized (GoodsSysConfig.tableLastUpdateTime) {
　　　    if (GoodsSysConfig.tableLastUpdateTime != null) {　　//这句判断是多余的，可以去掉！
　　　　　　　time = GoodsSysConfig.tableLastUpdateTime.get(tableName);
　　　　　　} 
　　　　　　else {
　        　log.error("GoodsSysConfig.tableLastUpdateTime 为空!");
        }
    }
    return time;
}


这段代码会引发两个BUG警告，一个是：[H C RCN] Nullcheck of value previously dereferenced，
另外一个就是这里要介绍的[M M NP] Synchronize and null check on the same field。其实，
在[H C RCN] Nullcheck of value previously dereferenced的介绍里面已经提到Synchronize and null check on the same field这个问题了，
出现这个BUG的原因是：synchronized (GoodsSysConfig.tableLastUpdateTime) 这里已经对GoodsSysConfig.tableLastUpdateTime这个变量进行了是否为Null的判断，
如果为Null，synchronized (...)会抛异常，并且经过synchronized (...)后，GoodsSysConfig.tableLastUpdateTime已经被独占，
所以 if (GoodsSysConfig.tableLastUpdateTime != null) 这句可以看作是多余的，因为GoodsSysConfig.tableLastUpdateTime不可能为Null。


17.[hyddd的FindBugs分析记录][M V MS] Public static method may expose internal representation by returning array  
[M V MS] Public static method may expose internal representation by returning array [MS_EXPOSE_REP]
A public static method returns a reference to an array that is part of the static state of the class. 
Any code that calls this method can freely modify the underlying array. One fix is to return a copy of the array.
一个静态的公共函数，它返回了一个私有的静态数组的引用。任何调用这个静态公共函数的代码，都有可能改变这个私有的静态数组。实例代码如下：
public static void main(String args[]) throws Exception{
        String[] strs = Test.getStrs();
        strs[0] = "123";
        Test.myTest();
}
public class Test {
    private static String[] strs = new String[10];
    
    public static String[] getStr(){
        return strs;
    }
    public static void myTest(){
        System.out.println(strs[0]);
    }
}
运行结果是：123
防止这种问题的方法是：返回一个数组的拷贝，而不直接返回数组引用。如：
public static String[] getStr(){
        return strs.clone();
}
注意：这个BUG和下面这两个BUG比较类似，可以比较一下:>
[M V EI2] May expose internal representation by incorporating reference to mutable object
[M V EI] May expose internal representation by returning reference to mutable object


18.[hyddd的FindBugs分析记录][M C NP] Possible null pointer dereference   
[M C NP] Possible null pointer dereference [NP_NULL_ON_SOME_PATH]
There is a branch of statement that, if executed, guarantees that a null value will be dereferenced, 
which would generate a NullPointerException when the code is executed. Of course, 
the problem might be that the branch or statement is infeasible and that the null pointer exception can't ever be executed; 
deciding that is beyond the ability of FindBugs.
看一段代码：
public void test(){
  //
  ResultSet rs = cmd.executeQuery();
  if (rs != null && rs.next()) {
    goodsId = rs.getInt("goods_id");
  }
  rs.close();　　//rs可能为null
  //
}
这里不多解释，更正代码的方法很多，可以用try...catch...finally...处理可能出现的异常，也可以在rs.close之前，先判断rs是否为null。




19.[hyddd的FindBugs分析记录][M B Nm] Method names should start with a lower case letter  
在Java里，函数的首字母应该小写。
而.Net和Java不同，它要求函数的首字母必须大写。
这个问题是个规范问题，这里就不提供实例代码了。


20.[hyddd的FindBugs分析记录][M P Dm] Method invokes toString() method on a String 
[M P Dm] Method invokes toString() method on a String [DM_STRING_TOSTRING]
Calling String.toString() is just a redundant operation. Just use the String.
对一个String对象使用了toString()方法，这种操作是多余的，完全可以去掉。
public static void main(String args[]) throws Exception{
        String str = "123";
        System.out.println(str.toString());
}


21.[hyddd的FindBugs分析记录][M P Bx] Method invokes inefficient Number constructor; use static valueOf instead 
 [M P Bx] Method invokes inefficient Number constructor; use static valueOf instead [DM_NUMBER_CTOR]
Using new Integer(int) is guaranteed to always result in a new object whereas Integer.valueOf(int) 
allows caching of values to be done by the compiler, class library, or JVM. 
Using of cached values avoids object allocation and the code will be faster. 
Values between -128 and 127 are guaranteed to have corresponding cached instances and using valueOf is 
approximately 3.5 times faster than using constructor. For values outside the constant range the performance of both styles is the same. 
Unless the class must be compatible with JVMs predating Java 1.5, use either autoboxing or the valueOf() method when creating instances of Long, 
Integer, Short, Character, and Byte.
这里，FindBugs推荐使用Integer.ValueOf(int)代替new Integer(int)，因为这样可以提高性能。
如果当你的int值介于-128～127时，Integer.ValueOf(int)的效率比Integer(int)快大约3.5倍。
下面看看JDK的源码，看看到Integer.ValueOf(int)里面做了什么优化：
public static Integer valueOf(int i) {
  final int offset = 128;
  if (i >= -128 && i <= 127) { // must cache
    return IntegerCache.cache[i + offset];
  }
  return new Integer(i);
} 
private static class IntegerCache {
  private IntegerCache(){}
    
  static final Integer cache[] = new Integer[-(-128) + 127 + 1];
  static {
  for(int i = 0; i < cache.length; i++)
     cache = new Integer(i - 128);
  }
} 
从源代码可以知道，ValueOf对-128～127这256个值做了缓存(IntegerCache)，如果int值的范围是：-128～127，在ValueOf(int)时，
他会直接返回IntegerCache的缓存给你。
所以你会看到这样的一个现象：
public static void main(String []args) {
     Integer a = 100;
     Integer b = 100;
     System.out.println(a==b);


     Integer c = new Integer(100);
     Integer d = new Integer(100);
     System.out.println(c==d);
}
结果是：
true
false
因为：java在编译的时候 Integer a = 100; 被翻译成-> Integer a = Integer.valueOf(100);，
所以a和b得到都是一个Cache对象，并且是同一个！而c和d是新创建的两个不同的对象，所以c自然不等于d。
再看看这段代码：
public static void main(String args[]) throws Exception{
        Integer a = 100;
        Integer b = a;
        a = a + 1;　　//或者a++;
        System.out.println(a==b);
}
结果是：false
因为在对a操作时(a=a+1或者a++)，a重新创建了一个对象，而b对应的还是缓存里的100，所以输出的结果为false。


22.[hyddd的FindBugs分析记录][M C NP] Method call passes null for unconditionally dereferenced parameter  
[M C NP] Method call passes null for unconditionally dereferenced parameter [NP_NULL_PARAM_DEREF]
This method call passes a null value to a method which might dereference it unconditionally.
这里FindBugs的解释是：你传入参数的值，有可能是一个NULL。下面看一段代码：
public boolean accept(File dir, String name) {
            String reg = null;
            if (this.type == ChangeConfigUtil.windowsType) {
                reg = "^(\\S+)" + ChangeConfigUtil.windowsTypeTag
                        + "\\.(\\S*)$";
            } else if (this.type == ChangeConfigUtil.linuxType) {
                reg = "^(\\S+)" + ChangeConfigUtil.linuxTypeTag + "\\.(\\S*)$";
            }
            
            Pattern p = Pattern.compile(reg, Pattern.MULTILINE);　　//这里reg可能为空
            Matcher m = p.matcher(name);
            if (m.find()) {
                return true;
            } else {
                return false;
            }
}
这里Pattern p = Pattern.compile(reg, Pattern.MULTILINE);可能会出现问题。
如果this.type既不等于windows，也不等于linux，那么reg=null，这会导致Pattern.compile(reg, Pattern.MULTILINE);抛异常。
这里，FindBugs想说明的就是这个问题。


23.[hyddd的FindBugs分析记录][M V EI] May expose internal representation by returning reference to mutable object 
 这个问题的解决方案和[M V EI2] May expose internal representation by incorporating reference to mutable object很类似，
 可以参考：http://www.cnblogs.com/hyddd/articles/1391118.html
 
24.[hyddd的FindBugs分析记录][M V EI2] May expose internal representation by incorporating reference to mutable object  
[M V EI2] May expose internal representation by incorporating reference to mutable object [EI_EXPOSE_REP2]
This code stores a reference to an externally mutable object into the internal representation of the object. 
 If instances are accessed by untrusted code, and unchecked changes to the mutable object would compromise security or other important properties, 
you will need to do something different. Storing a copy of the object is better approach in many situations.
这个问题和Inconsistent synchronization描述的问题很类似，解决方案也很类似，可以参考看看：http://www.cnblogs.com/hyddd/articles/1391098.html
先看一段代码：
public class Test  extends Thread{
    public static void main(String args[]) throws Exception{
        Test3 obj = new Test3();
        Date now = new Date();
        
        obj.setRegDate(now);    
        now.setYear(4000);　　//问题所在！
        
        System.out.println(obj.getRegDate());
    }
}
public class Test3 {
    private Date regDate ;    
    public void setRegDate(Date regDate) {
        this.regDate = regDate;
    }
    public Date getRegDate() {
        return regDate;
    }    
}
这段代码的输出是：Thu Feb 15 21:47:13 CST 5900
如果main里面不加now.setYear(4000);这句代码呢，结果是：Sun Feb 15 21:47:31 CST 2009
从这里我们发现了，修改一个对象，可能会引起其他对象的修改，因为JAVA里，对象是引用传递的......所以这里我的建议是：
setObj的时候，对象不要直接赋值(this.regDate = regDate)，而是赋值传入对象的拷贝(this.regDate = (Date)regDate.clone();)。
OK~现在我们把代码this.regDate = regDate替换成this.regDate = (Date)regDate.clone();，
运行一下看看结果，噢，输出是：Sun Feb 15 21:47:31 CST 2009。


25.[hyddd的FindBugs分析记录][M M IS] Inconsistent synchronization  
[M M IS] Inconsistent synchronization [IS2_INCONSISTENT_SYNC]
The fields of this class appear to be accessed inconsistently with respect to synchronization.  
This bug report indicates that the bug pattern detector judged that 
The class contains a mix of locked and unlocked accesses, 
At least one locked access was performed by one of the class's own methods, and 
The number of unsynchronized field accesses (reads and writes) was no more than one third of all accesses,
 with writes being weighed twice as high as reads 
A typical bug matching this bug pattern is forgetting to synchronize one of the methods in a class that is intended to be thread-safe.
You can select the nodes labeled "Unsynchronized access" to show the code locations where the detector believed that a 
field was accessed without synchronization.
Note that there are various sources of inaccuracy in this detector; for example, the detector cannot statically detect all
 situations in which a lock is held.  Also, even when the detector is accurate in distinguishing locked vs. unlocked accesses, 
 the code in question may still be correct.
先上一段代码：
public class Test  extends Thread{
    public static void main(String args[]) throws InterruptedException{
        ObjectClass obj = new ObjectClass(); 
        Thread t2 = new ChangeValue(obj);
        t2.start();
        Thread t1 = new AlwaysRun(obj);
        t1.start();
        sleep(10000);
        t1.stop();
    }
}
class AlwaysRun extends Thread{
    ObjectClass obj;
    public AlwaysRun(ObjectClass obj) {
        // TODO Auto-generated constructor stub
        this.obj = obj;
    }
    public void run() {
        obj.Loop();
    }
}
class ChangeValue extends Thread{
    ObjectClass obj;
    ChangeValue(ObjectClass obj){
        this.obj = obj;
    }
    
    public void run() {
        System.out.println("Thread2");
        ObjClass2 obj2 = obj.getObj();
        try {
            sleep(1500);
        } catch (InterruptedException e) {
            System.out.println("Error!");
        }
               obj2.str = "aaa";
        System.out.println("Thread2 Finish!");
    }
}
public class ObjectClass extends Thread {


    private ObjClass2 obj;
    private Object lockTable = new Object();
    
    public ObjectClass() {
        // TODO Auto-generated constructor stub
        obj = new ObjClass2();
    }
    public void setObj(ObjClass2 obj){
        synchronized (lockTable){
            this.obj = obj;      
        }
    }
    
    public ObjClass2 getObj(){
        synchronized (lockTable){
            return this.obj;      //出问题处！！
        }
    }
    
    public void Loop(){
        synchronized (lockTable){
            while(true){
                System.out.println(obj.str);
            }
        }
    }    
}
public class ObjClass2 {    
    public String str = "ddddddd";    
}
看看运行的结果：
Thread2
ddddddd
ddddddd
ddddddd
ddddddd
ddddddd
ddddddd
ddddddd
Thread2 Finish!
aaa
aaa
aaa
aaa
aaa
aaa
....


如果看明白代码，你应该会知道问题出再哪里了，为什么Loop使用了synchronized (lockTable)，但是obj.str还是被修改了？！
因为getObj()这个函数把obj对象返回了给外面，JAVA里面对象的传递是使用引用传递，如果对象传递到外面并且在做修改obj的时候没有加锁操作，
就是引起刚才的问题。所以如果getObj()函数返回的是对象，那么，请返回一个拷贝，而不要直接返回引用。
这里再总结一下值得注意问题：
1.看下面代码：
    public ObjClass2 getObj(){
        synchronized (lockTable){
            return this.obj;
        }
    }
synchronized (lockTable)不能阻止外面的函数修改obj，即：obj=getObj();当赋值完毕后，synchronized (lockTable)无效了，
如果后面需要修改obj的值，那么就得注意了！！！
另外建议的是，不直接返回this.obj，而是返回一个this.obj的拷贝。这样可以根本上避免出现上面的问题！
2.同理，在setObj(...)的时候，如果传入的是个对象，也建议是存储传入对象的拷贝，而不（this.obj=obj）这样直接赋值。
3.注意对竞争的资源都使用synchronized (lockTable)，不要像上面的Demo代码那样，一处用了，一处没有！


26.[hyddd的FindBugs分析记录][M D RCN] Redundant nullcheck of value known to be non-null  
参考文档：[hyddd的FindBugs分析记录][M C RCN] Nullcheck of value previously dereferenced


27.[hyddd的FindBugs分析记录][M D RCN] Repeated conditional tests  
[M C RpC] Repeated conditional tests [RpC_REPEATED_CONDITIONAL_TEST]
The code contains a conditional test is performed twice, one right after the other (e.g., x == 0 || x == 0). 
Perhaps the second occurrence is intended to be something else (e.g., x == 0 || y == 0).
这个不解释，一看代码就知道：
    public void test{
　　　　String str = "123";
            if(str!=null){
                if(str!=null){　　//重复了
                 System.out.println("123");
                }
            }
    }

28.[hyddd的FindBugs分析记录][M C RCN] Nullcheck of value previously dereferenced  
[M C RCN] Nullcheck of value previously dereferenced [RCN_REDUNDANT_NULLCHECK_WOULD_HAVE_BEEN_A_NPE]
A value is checked here to see whether it is null, but this value can't be null because it was previously 
dereferenced and if it were null a null pointer exception would have occurred at the earlier dereference.Essentially, 
this code and the previous dereference disagree as to whether this value is allowed to be null. 
Either the check is redundant or the previous dereference is erroneous.
先看一段代码：
public class MyTest {
    private String str = "123";
    public void setStr(String str){
        this.str = str;
    }
    public String getStr(){
        return this.str;
    }
    public void test(){
        String str2 = "123";
        synchronized (str) {
            if (str != null) {
                str2 ="123";
            } else {
                str2 ="456";
            }            
            System.out.println(str2);
        }
    }
}
这个时候这段代码就会报Nullcheck of value previously dereferenced这个Bug，看Bug定位，
发现问题出现在synchronized (str) 这里，str没有检查是否为NULL?!OK，我现在改用getStr()这个函数代替直接使用str，
即：synchronized (getStr())，重新FindBug......居然没有发现错误-_-，但事实上getStr()并没有检查str是否为Null！！
　　现在我换另外一种写法，代码如下：
public class MyTest {
    private String str = "123";
    public void setStr(String str){
        this.str = str;
    }
    public String getStr(){
        return this.str;
    }


    public void test(){
        String str2 = "123";


        if(str != null){　　　　　　　　//tag2
            synchronized (str) {
                if (str != null) {　　　　//tag1
                    str2 ="123";
                } else {
                    str2 ="456";
                }           
                System.out.println(str2);
            }
        }
        
    }
}
这次我在tag2处加了一行检查str是否为NULL的代码，看FindBugs结果，
出现了另外一个中等BUG：[M D RCN]Redundant nullcheck of value known to be non-null，跟踪发现是tag1处的代码是多余的，
因为tag2处已经检查了一遍，并且在synchronized (str)后，str被独占，它不可能被修改，也就是说synchronized (str)后，根本不需要检查str是否为空，
tag1处的代码是多余的。如果把tag1处代码去掉，[M D RCN]警告就没有了。
　　不知道大家有没有发现，上面的代码还有个问题，看这段代码：
if(str != null){
        synchronized (str) {      
            System.out.println(“123”);
        }
}
如果是多线程运行时，你不能排除它会if(str != null)和synchronized(str)之间进行线程切换，
然后把str至为null！所以上面这样写其实也是有可能出现问题的！只是FindBugs没有找出来。
我觉得最好的写法还是：
    public void test() throws Exception{
        try{
            String str2 = "123";
            synchronized (getStr()) {
                str2 ="456";
                System.out.println(str2);
            }
        }
        catch(Exception ex){
            throw ex;
        }
　　　//Do other things....
    }
其实synchronized (getStr()) 换成synchronized (str) 也是可以的，FindBugs不会再报Bug了。 
总结一下，我觉得FindBugs之所以会报[H C RCN] Nullcheck of value previously dereferenced，是因为我没有检查str的值是否为Null，
并且没有注意对可能出现的Exception的截获。而简单使用getStr()，不检查str的值，不作异常捕获，也能躲过这个Bug，
我觉得可能是FindBugs的一个BUG:<，因为估计它会认为，getStr()这个函数里面会检查str的值......但这个仅仅是个人认为而已。


29.[hyddd的FindBugs分析记录][H C FS] More arguments are passed that are actually used in the format string  
[H C FS] More arguments are passed that are actually used in the format string [VA_FORMAT_STRING_EXTRA_ARGUMENTS_PASSED]
A format-string method with a variable number of arguments is called, but more arguments are passed than are actually used by the format string. 
This won't cause a runtime exception, but the code may be silently omitting information that was intended to be included in the formatted string.
这个错误很简单，是使用String.format的时候出了问题，format里面的参数没有被全部用上。
看下面一段代码：
public void test(){
    String str1 = "123";
    String str2 = "456";
    String str3 = String.format("{0} {1}" , str1 ,str2);
    System.out.println(str3);
}
输出的结果是：{0} {1}
这个Bug描述就是这种问题，str1和str2根本没有被用上！{0}{1}这种Format格式是.NET上面的用法，java里面应该是%s %s。
这个是一个代码逻辑问题，可能是你写代码时不小心导致的，它在这段代码里不会导致异常，但往往会很可能导致其他地方异常，那时候你可能会百思不得其解。

A boxed primitive is created from a String, just to extract the unboxed primitive value. It is more efficient to just call the static parseXXX method.
修改建议：使用Long.ParseLong，避免自动装箱再拆箱
问题原因：
Long.ParseLong(String)方法，将 string 参数解析为有符号十进制 ，返回一个long的基本类型值
Long.ValueOf(String) ,方法得到的值非常相似。只是最后被转换为一个Long的包装类
This class is an inner class, but does not use its embedded reference to the object which created it. This reference makes the instances of the class larger, and may keep the reference to the creator object alive longer than necessary. If possible, the class should be made static.
修改建议：若成员类中未访问外围类的非静态成员，为避免额外的空间和时间开销，建议改用静态成员类。
问题原因：
非静态成员类和静态成员类的区别在于，非静态成员类是对象的，静态成员类是类的。非静态成员类可以访问外围类的任何成员，但前提是必须存在外围类对象。JAVA需要额外维护非静态成员类和外围类对象的关系。
This code generates a hashcode and then computes the absolute value of that hashcode. If the hashcode is Integer.MIN_VALUE, then the result will be negative as well (since Math.abs(Integer.MIN_VALUE) == Integer.MIN_VALUE).
One out of 2^32 strings have a hashCode of Integer.MIN_VALUE, including “polygenelubricants” “GydZG_” and “”DESIGNING WORKHOUSES”.
修改建议：在使用之前判断一下是否是为Integer.MIN_VALUE
问题原因：
此代码产生哈希码，然后计算该哈希码的绝对值。如果哈希码是Integer.MIN_VALUE的，那么结果将是负的（因为Math.abs（Integer.MIN_VALUE）==Integer.MIN_VALUE）。
A boxed value is unboxed and then immediately reboxed.
修改建议：三元运算符两个分支的返回类型保持一致。
问题原因：
装箱的值被拆箱，然后立刻重新装箱
This class defines a field with the same name as a visible instance field in a superclass. This is confusing, and may indicate an error if methods update or access one of the fields when they wanted the other.
修改建议：要么去掉其中一个字段，要么重新命名。
问题原因：
变量与超类中的可访问静态变量重名，使用时会造成迷惑

This private method is never called. Although it is possible that the method will be invoked through reflection, it is more likely that the method is never used, and should be removed.
修改建议：删除该方法
问题原因：
这个私有方法没有被调用。虽然可能通过反射的方法将被调用，更可能从未使用过，应该被删除。

This method calls equals(Object) on two references of different class types and analysis suggests they will be to objects of different classes at runtime. Further, examination of the equals methods that would be invoked suggest that either this call will always return false, or else the equals method is not be symmetric (which is a property required by the contract for equals in class Object).
修改建议：
去除这种无意义的判断
问题原因：
两个不同类型实例通过equals方法，通常情况下都会判断为非相同对象，其返回值也将始终为false。
例如，调用string的equals方法比较double类型，永远都会返回false，如果把这个作为逻辑判断是没有意义的

The method seems to be building a String using concatenation in a loop. In each iteration, the String is converted to a StringBuffer/StringBuilder, appended to, and converted back to a String. This can lead to a cost quadratic in the number of iterations, as the growing string is recopied in each iteration.
Better performance can be obtained by using a StringBuffer (or StringBuilder in Java 1.5) explicitly.
修改建议：使用StringBuilder/StringBuffer
问题原因：
在循环里使用字符串连接，效率低
A reference value which is null on some exception control path is dereferenced here. This may lead to a NullPointerException when the code is executed. Note that because SpotBugs currently does not prune infeasible exception paths, this may be a false warning.
Also note that SpotBugs considers the default case of a switch statement to be an exception path, since the default case is often infeasible.
修改建议：对引用对象进行判空
问题原因：
代码调用时，遇到异常分支，可能造成一个对象没有获得赋值依旧保持NULL空指针。接下来如果对这个对象有引用，可能造成NullPointerException空指针异常。
A String function is being invoked and “.” or “|” is being passed to a parameter that takes a regular expression as an argument. Is this what you intended? For example
- s.replaceAll(“.”, “/”) will return a String in which every character has been replaced by a ‘/’ character
- s.split(“.”) always returns a zero length array of String
- “ab|cd”.replaceAll(“|”, “/”) will return “/a/b/|/c/d/”
- “ab|cd”.split(“|”) will return array with six (!) elements: [, a, b, |, c, d]
修改建议：
在前面加上“\”转义符。
问题原因：
String的split方法传递的参数是正则表达式，正则表达式本身用到的字符需要转义，如：句点符号“.”，美元符号“$”，乘方符号“^”，大括号“{}”，方括号“[]”，圆括号“()” ，竖线“|”，星号“*”，加号“+”，问号“?”等等，这些需要在前面加上“\”转义符。
There is a branch of statement that, if executed, guarantees that a null value will be dereferenced, which would generate a NullPointerException when the code is executed. Of course, the problem might be that the branch or statement is infeasible and that the null pointer exception can’t ever be executed; deciding that is beyond the ability of SpotBugs.
修改建议：
判断header是否为null或使用try…catch…finally。
问题原因：可能存在空引用
This method call passes a null value for a non-null method parameter. Either the parameter is annotated as a parameter that should always be non-null, or analysis has shown that it will always be dereferenced.
修改建议：赋予函数参数默认值。
问题原因：传递了一个空值给函数参数
The return value of this method should be checked. One common cause of this warning is to invoke a method on an immutable object, thinking that it updates the object. For example, in the following code fragment,
String dateString = getHeaderField(name);
dateString.trim();
the programmer seems to be thinking that the trim() method will update the String referenced by dateString. But since Strings are immutable, the trim() function returns a new String value, which is being ignored here. The code should be corrected to:
String dateString = getHeaderField(name);
dateString = dateString.trim();
修改建议：定义变量接收方法的返回值
问题原因：忽略了该方法的返回值
This method accesses the value of a Map entry, using a key that was retrieved from a keySet iterator. It is more efficient to use an iterator on the entrySet of the map, to avoid the Map.get(key) lookup.
修改建议：遍历entry（桶）然后直接从entry得到value
问题原因：
很多人都这样遍历Map，没错，但是效率很低，先一个一个的把key遍历，然后在根据key去查找value，这不是多此一举么，为什么不遍历entry（桶）然后直接从entry得到value呢？它们的执行效率大概为1.5:1
This code creates an exception (or error) object, but doesn’t do anything with it. For example, something like

if (x < 0) {
    new IllegalArgumentException("x must be nonnegative");
}
It was probably the intent of the programmer to throw the created exception:

if (x < 0) {
    throw new IllegalArgumentException("x must be nonnegative");
}

修改建议：将创建的异常抛出
问题原因：创建了一个异常对象，但是没有对它做任何操作

Using the java.lang.String(String) constructor wastes memory because the object so constructed will be functionally indistinguishable from the String passed as a parameter. Just use the argument String directly.
修改建议：直接使用其参数
问题原因：这里调用了String的构造函数来新建一个字符串，浪费内存

This code contains a sequence of calls to a concurrent abstraction (such as a concurrent hash map). These calls will not be executed atomically.
修改建议：使用putIfAbsent()保证操作是原子级的
问题原因：在多线程访问的情况下，它是不安全的

A value is checked here to see whether it is null, but this value can’t be null because it was previously dereferenced and if it were null a null pointer exception would have occurred at the earlier dereference. Essentially, this code and the previous dereference disagree as to whether this value is allowed to be null. Either the check is redundant or the previous dereference is erroneous
修改建议：在使用变量之前先判断其是否为null
问题原因：出现该bug有两种情况，多余的null检查或者没有进行null值检查。

As the JavaDoc states, DateFormats are inherently unsafe for multithreaded use. The detector has found a call to an instance of DateFormat that has been obtained via a static field. This looks suspicious.
修改建议：使用ThreadLocal: 每个线程都将拥有自己的SimpleDateFormat对象副本。
问题原因：SimpleDateFormat不是线程安全的
————————————————

-============================================================================================================

FindBugs Bug Descriptions (推荐)
http://findbugs.sourceforge.net/bugDescriptions.html#NP_NULL_PARAM_DEREF




