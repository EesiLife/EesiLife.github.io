---
layout: post
title:  SetContentView源码阅读
date:   2016-10-16 11:29:08 +0800
tag: Android源码
---
在Activity和FragmentActivity中都会用到setContentView(R.layout.activity_main);
进入SetContentView方法可以看到：
<pre><code>
/**
* 从布局资源中设置activity内容。资源被加载，然后添加所有顶级views到activity中。
     *
     * @param layoutResID Resource ID to be inflated.
     *
     * @see #setContentView(android.view.View)
     * @see #setContentView(android.view.View, android.view.ViewGroup.LayoutParams)
     */
    public void setContentView(@LayoutRes int layoutResID) {
        getWindow().setContentView(layoutResID);
        initWindowDecorActionBar();
    }
</code></pre>

其中，getWindow()方法如下，
<pre><code>private Window mWindow;
public Window getWindow() {
        return mWindow;
    }
</code></pre>
Window是个抽象类，必须得有子类实现才可以使用。搜索”mWindow =“可以看到
mWindow初始化实现的是PhoneWindow对象：
<pre><code>mWindow = new PhoneWindow(this, window);</code></pre>
PhoneWindow是com.android.internal.policy.PhoneWindow，继承Window:
<pre><code>/**
/* activity的主窗口构造方法
/**
public PhoneWindow(Context context, Window preservedWindow) {
        this(context);
        // 只有主acitivity窗口使用decor context,所有其他的window依    赖赋予它们的context
        //所以这里有一个mUseDecorContext布尔值,标记可以使用DecorContext,在generateDecor(int featureId)方法中判断
        mUseDecorContext = true;
        if (preservedWindow != null) {
            mDecor = (DecorView) preservedWindow.getDecorView();
            mElevation = preservedWindow.getElevation();
            mLoadElevation = false;
            mForceDecorInstall = true;
            getAttributes().token = preservedWindow.getAttributes().token;
        }
        boolean forceResizable = Settings.Global.getInt(context.getContentResolver(),
                DEVELOPMENT_FORCE_RESIZABLE_ACTIVITIES, 0) != 0;
        mSupportsPictureInPicture = forceResizable || context.getPackageManager().hasSystemFeature(
                PackageManager.FEATURE_PICTURE_IN_PICTURE);
    }</code></pre>
    
通过PhoneWindow构造方法可以发现：PhoneWindow(Context context, Window preservedWindow)是acitivity的主窗口构造方法。在mDecor = (DecorView) preservedWindow.getDecorView(),可以看到getDecor实现的是installDecor();

<pre><code>@Override
    public final View getDecorView() {
        if (mDecor == null || mForceDecorInstall) {
            installDecor();
        }
        return mDecor;
    }</code></pre>
    
在installDecor()中，
<pre><code>mForceDecorInstall = false;
        if (mDecor == null) {
            mDecor = generateDecor(-1);
            mDecor.setDescendantFocusability(ViewGroup.FOCUS_AFTER_DESCENDANTS);
            mDecor.setIsRootNamespace(true);
            if (!mInvalidatePanelMenuPosted && mInvalidatePanelMenuFeatures != 0) {
                mDecor.postOnAnimation(mInvalidatePanelMenuRunnable);
            }
        }
        ...
        </code></pre>
看下generateDecor(-1)，判断是否能使用DecorContext:能则先获取applicationContext；系统进程没有application context时，在这种情况下需要直接使用当前拥有的context(/1),否则我们想要application context时,DecorView无法附着于activity上。application context不为空，new一个DecorContext（/2）。返回DecorView.
<pre><code></code>protected DecorView generateDecor(int featureId) {
        // 系统进程没有application context时，在这种情况下需要直接使用当前拥有的context,不然我们想要application context时,DecorView无法附着于activity上。
        Context context;
        if (mUseDecorContext) {
            Context applicationContext = getContext().getApplicationContext();
            if (applicationContext == null) {
            		/1
                context = getContext();
            } else {
            		/2
                context = new DecorContext(applicationContext, getContext().getResources());
                if (mTheme != -1) {
                    context.setTheme(mTheme);
                }
            }
        } else {
            context = getContext();
        }
        return new DecorView(context, featureId, this, getAttributes());
    }</pre>
 
    
    
继续向下看,mContentParent = generateLayout(mDecor);

 <pre><code></code>if (mContentParent == null) {
            mContentParent = generateLayout(mDecor);
            ....
            </code></pre>
            
查看 generateLayout(mDecor)方法：根据window特点的不同，加载不同的布局，发现所有布局中都有一个id为content的FrameLayout。最终返回的是一个id为content的FrameLayout。
<pre><code> protected ViewGroup generateLayout(DecorView decor) {
   	....获取并设置窗口参数
   	// Inflate the window decor.
        int layoutResource;
        int features = getLocalFeatures();
        // System.out.println("Features: 0x" + Integer.toHexString(features));
        if ((features & (1 << FEATURE_SWIPE_TO_DISMISS)) != 0) {
            layoutResource = R.layout.screen_swipe_dismiss;//滑动消失布局
        } else if ((features & ((1 << FEATURE_LEFT_ICON) | (1 << FEATURE_RIGHT_ICON))) != 0) {
        		//dialog,左右2个ICON按钮
            if (mIsFloating) {
            		//悬浮
                TypedValue res = new TypedValue();
                getContext().getTheme().resolveAttribute(
                        R.attr.dialogTitleIconsDecorLayout, res, true);
                layoutResource = res.resourceId;
            } else {
                layoutResource = R.layout.screen_title_icons;
            }
            // XXX Remove this once action bar supports these features.
            removeFeature(FEATURE_ACTION_BAR);
            // System.out.println("Title Icons!");
        } else if ((features & ((1 << FEATURE_PROGRESS) | (1 << FEATURE_INDETERMINATE_PROGRESS))) != 0
                && (features & (1 << FEATURE_ACTION_BAR)) == 0) {
            // Special case for a window with only a progress bar (and title).
            // XXX Need to have a no-title version of embedded windows.
            layoutResource = R.layout.screen_progress;
            // System.out.println("Progress!");
        } else if ((features & (1 << FEATURE_CUSTOM_TITLE)) != 0) {
            // Special case for a window with a custom title.
            // If the window is floating, we need a dialog layout
            if (mIsFloating) {
                TypedValue res = new TypedValue();
                getContext().getTheme().resolveAttribute(
                        R.attr.dialogCustomTitleDecorLayout, res, true);
                layoutResource = res.resourceId;
            } else {
                layoutResource = R.layout.screen_custom_title;
            }
            // XXX Remove this once action bar supports these features.
            removeFeature(FEATURE_ACTION_BAR);
        } else if ((features & (1 << FEATURE_NO_TITLE)) == 0) {
            // If no other features and not embedded, only need a title.
            // If the window is floating, we need a dialog layout
            if (mIsFloating) {
                TypedValue res = new TypedValue();
                getContext().getTheme().resolveAttribute(
                        R.attr.dialogTitleDecorLayout, res, true);
                layoutResource = res.resourceId;
            } else if ((features & (1 << FEATURE_ACTION_BAR)) != 0) {
                layoutResource = a.getResourceId(
                        R.styleable.Window_windowActionBarFullscreenDecorLayout,
                        R.layout.screen_action_bar);
            } else {
                layoutResource = R.layout.screen_title;
            }
            // System.out.println("Title!");
        } else if ((features & (1 << FEATURE_ACTION_MODE_OVERLAY)) != 0) {
            layoutResource = R.layout.screen_simple_overlay_action_mode;
        } else {
            // Embedded, so no decoration is needed.
            layoutResource = R.layout.screen_simple;
            // System.out.println("Simple!");
        }
   	mDecor.startChanging();
   mDecor.onResourcesLoaded(mLayoutInflater, layoutResource);
   ViewGroup contentParent = (ViewGroup)findViewById(ID_ANDROID_CONTENT);
   	...
   	return contentParent;
   }
</code></pre>
再回到setContentView(int layoutResID),看（1.），mLayoutInflater将layoutResID布局加载到id为content的FrameLayout中。
<pre><code>setContentView(int layoutResID) {
   ...
   if (hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
            final Scene newScene = Scene.getSceneForLayout(mContentParent, layoutResID,
                    getContext());
            transitionTo(newScene);
        } else {
        //1.
            mLayoutInflater.inflate(layoutResID, mContentParent);
        }
        mContentParent.requestApplyInsets();
        final Callback cb = getCallback();
        if (cb != null && !isDestroyed()) {
            cb.onContentChanged();
        }
        mContentParentExplicitlySet = true;
   }</code></pre> 
至此，setContentView(@LayoutRes int layoutResID)结束了。整个流程为：获取PhoneWindow-->安装DecorView-->调用mDecor.onResourcesLoaded(mLayoutInflater, layoutResource);获取id为content的FrameLayout-->将layoutResID布局资源加载到content中。
