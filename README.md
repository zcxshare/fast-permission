# FastPermission
[![](https://jitpack.io/v/zcxshare/fast-permission.svg)](https://jitpack.io/#zcxshare/fast-permission)

FastPermission为android权限请求框架，一个注解解决android权限请求,基于aspectj实现，使用注解即可请求权限,简单方便。
NeedPermission支持在所有的方法中使用和activity的类上使用,提供了权限前操作，权限后的处理(权限拒绝或不再提醒)等功能。
# 使用
### 使用到的类:
  1. **FastPermission**:用于初始化的类
  1. **@NeedPermission**:请求权限的注解,可以作用于任何类的方法和activity类上,拥有参数:
      - value:要请求的权限
      - requestCode:请求码
      - isAllowExecution:当权限被拒绝是否继续执行
      - requestBefore:请求前调用的方法,和@PermissionBefore配合使用,会调用带有相同参数的被PermissionBefore注解的方法
      - permissionCanceled:请求被取消的方法,和@PermissionCanceled配合使用,会调用带有相同参数的被PermissionCanceled注解的方法
      - permissionDenied:请求被拒绝的方法,和@permissionDenied配合使用,会调用带有相同参数的被permissionDenied注解的方法
  1. **@PermissionBefore**:请求权限前的操作，可以在请求权限的本类中和配置类中使用，被注解的方法的参数只能是PermissionBeforeBean
,与NeedPermission联合使用,通过requestBefore匹配
  1. **@PermissionCanceled**:权限被取消(用户点击禁止权限)时调用的方法,可以在请求权限的本类中和配置类中使用
，被注解的方法的参数只能是PermissionCanceledBean,与NeedPermission联合使用,通过permissionCanceled匹配
  1. **@PermissionDenied**:权限被取消(用户勾选禁止后不再提示并点击禁止权限)时调用的方法,可以在请求权限的本类中和配置类中使用
，被注解的方法的参数只能是PermissionDeniedBean,与NeedPermission联合使用,通过permissionDenied匹配

### 使用方式
  在你的Application的onCreate方法中使用FastPermission.getInstance().init()初始化,然后在需要权限的方法
  FastPermissionConfig该类用于配置统一的权限前置操作、取消操作、拒绝操作的公共方法。
  ~~~
  public void onCreate() {
        super.onCreate();
        FastPermission.getInstance().init(getApplicationContext(),new FastPermissionConfig());//FastPermissionConfig如果不用可设置为null
    }
  ~~~
  
方法中使用注解时:
~~~
@NeedPermission({Manifest.permission.CAMERA,Manifest.permission.WRITE_EXTERNAL_STORAGE})
    private void onClickContacts() {
        FragmentManager fragmentManager = getFragmentManager();
        Fragment cameraFragment = fragmentManager.findFragmentByTag("camera");
        FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();
        if (cameraFragment ==null) {
            fragmentTransaction.replace(R.id.sample_content_fragment, CameraPreviewFragment.newInstance(), "camera")
                    .addToBackStack("camera")
                    .commitAllowingStateLoss();
        }else {
            fragmentTransaction.show(cameraFragment);
        }
    }
~~~
别忘了在manifest.xml文件中还要加权限哟
~~~
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.CAMERA"/>
~~~
以上就是简单使用方式,一个注释搞定,是不是很简单,如果有更多需求,请继续往下看

activity中使用注解时:
~~~
@NeedPermission(value = {Manifest.permission.CAMERA, Manifest.permission.WRITE_EXTERNAL_STORAGE},
        requestBefore = Manifest.permission.CAMERA, permissionCanceled = Manifest.permission.CAMERA,
        permissionDenied = Manifest.permission.CAMERA, isAllowExecution = true)
public class CameraActivity extends Activity implements PermissionListener {

    @Override
    public void onPermissionGranted() {
        mHasPermission =true;
        initCamera();
    }

    @Override
    public void onPermissionCanceled(PermissionCanceledBean bean) {

    }

    @Override
    public void onPermissionDenied(PermissionDeniedBean bean) {

    }

    @PermissionBefore
    public void before(final PermissionBeforeBean beforeBean){
        new android.support.v7.app.AlertDialog.Builder(this)
                .setTitle("我们需要相机权限来正常拍照")
                .setPositiveButton("确定", new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {
                        beforeBean.proceed(true);
                    }
                })
                .setNegativeButton("取消", new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {
                        beforeBean.proceed(false);
                    }
                })
                .show();
    }

｝
~~~

FastPermissionConfig使用如下:
~~~
public    class FastPermissionConfig   {

    @PermissionBefore(Manifest.permission.CAMERA)//该注解注解的方法参数只能是PermissionBeforeBean
    public void before(final PermissionBeforeBean beforeBean){
        new android.support.v7.app.AlertDialog.Builder(beforeBean.getContext())
                .setTitle("来自配置文件,我们需要相机权限来正常拍照")
                .setPositiveButton("确定", new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {
                        beforeBean.proceed(true);//必须调用这个方法,确保正常往下执行
                    }
                })
                .setNegativeButton("取消", new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {
                        beforeBean.proceed(false);//必须调用这个方法,确保正常往下执行
                    }
                })
                .show();
    }

    @PermissionCanceled(Manifest.permission.CAMERA)//该注解注解的方法参数只能是PermissionCanceledBean
    public void cancel(final PermissionCanceledBean canceledBean){
        new android.support.v7.app.AlertDialog.Builder(canceledBean.getContext())
                .setTitle("来自配置文件,我们需要相机权限,请同意")
                .setPositiveButton("确定", new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {
                        canceledBean.againRequest();//调用该方法重试请求
                    }
                })
                .setNegativeButton("取消", new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {
                        canceledBean.proceed();
                    }
                    }
                })
                .show();
    }

    @PermissionDenied(Manifest.permission.CAMERA)//该注解注解的方法参数只能是PermissionDeniedBean
    public void denied(final PermissionDeniedBean deniedBean){
        new android.support.v7.app.AlertDialog.Builder(deniedBean.getContext())
                .setTitle("来自配置文件,我们需要权限,是否设置")
                .setPositiveButton("确定", new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {
                        deniedBean.toSettingActivity();//打开设置界面设置权限
                    }
                })
                .setNegativeButton("取消", new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {
                        deniedBean.proceed();
                    }
                })
                .show();
    }

}
~~~
当然,如果你还有什么特殊需要也可以直接用工具类PermissionUtils
~~~
PermissionUtils.requestPermissions(context, permissions, requestCode, new PermissionListener() {
            @Override
            public void onPermissionGranted() {
               
            }

            @Override
            public void onPermissionCanceled(PermissionCanceledBean bean) {
                
            }

            @Override
            public void onPermissionDenied(PermissionDeniedBean bean) {
                
            }
        });
~~~

# 依赖
在你的根目录的build.gradle中添加配置[gradle-aspectj-plugin](https://github.com/zcxshare/gradle-aspectj-plugin)如下:
~~~
buildscript {
 
    repositories {
        maven { url "https://jitpack.io" }
    }
    dependencies {
        ...
        //下面插件二选一,kotlin项目只能用第一个
        classpath 'com.hujiang.aspectjx:gradle-android-plugin-aspectjx:2.0.8'
        classpath 'com.github.zcxshare:gradle-aspectj-plugin:1.0.3'
    }
}
allprojects {
    repositories {
        maven { url "https://jitpack.io" }
    }
}

~~~
然后在你的app的build.gradle中使用插件和依赖
~~~
apply plugin: 'com.android.application'
//下面插件二选一
apply plugin: 'android-aspectjx'//对应classpath 'com.hujiang.aspectjx:gradle-android-plugin-aspectjx:2.0.8'
apply plugin: 'aspectj-plugin'//对应classpath 'com.github.zcxshare:gradle-aspectj-plugin:1.0.3'

dependencies {
    implementation 'com.github.zcxshare:fast-permission:1.0.3'
}
~~~

混淆时添加到app的混淆文件proguard-rules.pro中
~~~
-keepclasseswithmembers class * {
@com.zcx.fast_permission_runtime.annotation.* <methods>;
}

-keepclassmembers class * {
public android.content.Context getContext();
}
~~~
有问题请issues哟,好用请点个star啦~~~
