### 1. Activity的生命周期和启动模式
#### 1. 生命周期
1. 基本情况
	* onStart和onStop根据是否可见被回调；onResume和onPause根据是否在前台被回调。实际使用中没有其他区别。
    * onRestart当由不可见变为可见时回调。
    * 由A启动B，在A的onPause执行完之前，B不会被创建。所以在onPause中不能执行过多的操作。
2. 异常情况
    **设备配置改变**或者**内存不足**时，Activity重建。
    * onSaveInstanceState & onRestoreInstanceState
        * 在onStop之前回调onSaveInstanceState保存状态，与onPause无绝对的先后顺序（Android P开始，发生在onStop之后）；重建时在onStart之后回调onRestoreInstanceState恢复状态。
        * onSaveInstanceState在所有Activity重新返回时可能已经被销毁的情况下，都会被调用。所以，按home键/打开新Activity/锁屏操作，也会触发回调；onRestoreInstanceState只有在Activity真的被销毁重建时才被调用。
        * onCreate中也可以根据Bundle是否为空，进行数据恢复；onRestroeInstanceState被调用时Bundle一定不为空。
        * 每个View也实现了这两个回调，所以不同的View会有自己的默认保存和恢复操作。
    * android:configChanges指定希望去处理的一些configs，而不是让Activity重建。常见的如locale/orientation/screenSize/keyboardHidden，当这些configs改变时，Activity不会重建，而是回调onConfigurationChanged。screenSize为API 13引入，屏幕旋转时也会改变，所以要与orientation同时指定。
    * 内存不足时系统按照优先级去杀死目标Activity所在的进程。如果一个进程没有四大组件在执行，优先级会很低。
#### 2. 启动模式
综合[官方文档](https://developer.android.com/guide/components/activities/tasks-and-back-stack)
1. task和back stack
    * task是一系列Activity的集合，Activity通过back stack进行维护。
    * 当启动了新的task或者点击了home键，当前的task会被放置到background。
2. launch mode
    定义了启动的Activity与当前task是如何关联的。两种方式指定：
    * **通过android:launchMode**
        * standard
            创建一个新的Activity，运行在启动该Activity的Activity所在的task中。
        * singleTop
            如果要创建的Activity已经位于task的栈顶，仅回调该Activity的onNewIntent；否则创建新的Activity，无论在task中是否存在该Activity的实例。
        * singleTask
        
            ```
            task name -> taskAffinity
            if (task exist) {
                if (activity exist) {
                    clearTop
                    call onNewIntent
                } else {
                    create activity and add to task
                }
            } else {
                create task
                create activity and add to task
            }
            ```
        * singleInstance
            在singleTask的基础上，限定了该Activity独占一个task。
    * **通过intent.addFlags**。可以修改目标Activity launchMode中定义的行为。实际情况中，各种flag的组合加上launchMode的设置，最终的行为远比四种预设要复杂。
        * FLAG_ACTIVITY_NEW_TASK
        在新的task中（taskAffinity指定）启动该Activity。找到目标task之后，后续的行为则按照launchMode的设置处理。
        例如，Activity launchMode为standard，当intent中设置该flag，然后启动该Activity，则会在taskAffinity设置的task（此时一般是默认值）中创建该Activity，无论在该task中是否有其实例存在。
        * FLAG_ACTIVITY_SINGLE_TOP
        同singleTop。
        * FLAG_ACTIVITY_CLEAR_TOP
            * 会销毁目标Activity之上的所有Activity，之后的行为分两种情况：
        ①如果目标Activity是standard，则目标Activity销毁重建，新的intent会被传递进去。因为startand的Activity总是会被重建（官方解释）。
        ②如果目标Activity是其他mode，或者intent中指定了FLAG_ACTIVITY_SINGLE_TOP，则目标Activity不会被重建，通过onNewIntent传递新的intent
            * 当目标Activity为standard，与FLAG_ACTIVITY_NEW_TASK结合并不能实现singleTask的效果，因为standard的Activity会被重建。所以要另外加上FLAG_ACTIVITY_SINGLE_TOP才可以达到效果。
3. activity的一些属性设置
    * taskAffinity：指定task name，默认值为package name。用于Application，指定所有Activity的task name；用于Activity，指定该Activity要运行的task的name。
    * allowTaskReparenting：允许Activity在其taskAffinity指定的task移到前台的时刻，从之前的task转移到该task。If an APK file contains more than one "app" from the user's point of view, you probably want to use the taskAffinity attribute to assign different affinities to the activities associated with each "app".
    * 其他见[Clearing the back](
https://developer.android.com/guide/components/activities/tasks-and-back-stack#Clearing)
    * Activity设置如下IntentFilter，会在启动器中显示图标，以提供task的入口：
        ```
        <intent-filter ... >
            <action android:name="android.intent.action.MAIN" />
            <category android:name="android.intent.category.LAUNCHER" />
        </intent-filter>
        ```
        对于singleTask和singleInstance的Activity可以添加该设置，方便用户启动相应的task。
        
#### 3.IntentFilter
Activity启动分显式调用和隐式调用。
隐式调用需要Intent能够匹配目标Activity的IntentFilter中设置的过滤信息，即其中的action、category、data都要匹配。同时一个Activity可以有多个IntentFilter，其中action、category、data也可以有多个。
1. action

### IPC机制
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTY5Mzg2NjA0MiwtMjA4ODc0NjYxMl19
-->