##### Activity Results API 的使用

```kotlin
 class MyActivityResultContract: ActivityResultContract<String,String>(){
        override fun createIntent(context: Context, input: String?): Intent {
            return Intent(context,SecondActivity::class.java).apply {
                putExtra("name",input)
            }
        }

        override fun parseResult(resultCode: Int, intent: Intent?): String? {
            val data = intent?.getStringExtra("result")
            return if (resultCode == Activity.RESULT_OK && data != null) data
            else null
        }

    }
```

在 `createIntent`方法中创建了Intent，并且携带了参数 `name`,在 `parseResult`方法中，获取了返回的数据 `result`。

注册协议，获取启动器-ActivityResultLauncher

```kotlin
private val myActivityLauncher = registerForActivityResult(MyActivityResultContract()){result ->
   Toast.makeText(applicationContext,result,Toast.LENGTH_SHORT).show()
   textView.text = "回传数据：$result"
}
```

调用

```
 button.setOnClickListener {
      // 开启页面跳转
      myActivityLauncher.launch("Hello,技术最TOP")
 }
```

##### 代码分析

先来看看调用了 `registerForActivityResult`返回了什么

主要看 `ActivityResultRegistry`

```java
/*
 * Copyright 2018 The Android Open Source Project
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package androidx.activity;

import static android.os.Build.VERSION.SDK_INT;

import static androidx.activity.result.contract.ActivityResultContracts.RequestMultiplePermissions;
import static androidx.activity.result.contract.ActivityResultContracts.RequestMultiplePermissions.ACTION_REQUEST_PERMISSIONS;
import static androidx.activity.result.contract.ActivityResultContracts.RequestMultiplePermissions.EXTRA_PERMISSIONS;
import static androidx.activity.result.contract.ActivityResultContracts.RequestMultiplePermissions.EXTRA_PERMISSION_GRANT_RESULTS;
import static androidx.activity.result.contract.ActivityResultContracts.StartActivityForResult;
import static androidx.activity.result.contract.ActivityResultContracts.StartActivityForResult.EXTRA_ACTIVITY_OPTIONS_BUNDLE;
import static androidx.activity.result.contract.ActivityResultContracts.StartIntentSenderForResult;
import static androidx.activity.result.contract.ActivityResultContracts.StartIntentSenderForResult.ACTION_INTENT_SENDER_REQUEST;
import static androidx.activity.result.contract.ActivityResultContracts.StartIntentSenderForResult.EXTRA_INTENT_SENDER_REQUEST;
import static androidx.activity.result.contract.ActivityResultContracts.StartIntentSenderForResult.EXTRA_SEND_INTENT_EXCEPTION;

import android.Manifest;
import android.annotation.SuppressLint;
import android.app.Activity;
import android.content.Context;
import android.content.Intent;
import android.content.IntentSender;
import android.content.pm.PackageManager;
import android.os.Build;
import android.os.Bundle;
import android.os.Handler;
import android.os.Looper;
import android.text.TextUtils;
import android.view.View;
import android.view.ViewGroup;
import android.view.Window;

import androidx.activity.contextaware.ContextAware;
import androidx.activity.contextaware.ContextAwareHelper;
import androidx.activity.contextaware.OnContextAvailableListener;
import androidx.activity.result.ActivityResultCallback;
import androidx.activity.result.ActivityResultCaller;
import androidx.activity.result.ActivityResultLauncher;
import androidx.activity.result.ActivityResultRegistry;
import androidx.activity.result.ActivityResultRegistryOwner;
import androidx.activity.result.IntentSenderRequest;
import androidx.activity.result.contract.ActivityResultContract;
import androidx.annotation.CallSuper;
import androidx.annotation.ContentView;
import androidx.annotation.LayoutRes;
import androidx.annotation.MainThread;
import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import androidx.core.app.ActivityCompat;
import androidx.core.app.ActivityOptionsCompat;
import androidx.core.content.ContextCompat;
import androidx.lifecycle.HasDefaultViewModelProviderFactory;
import androidx.lifecycle.Lifecycle;
import androidx.lifecycle.LifecycleEventObserver;
import androidx.lifecycle.LifecycleOwner;
import androidx.lifecycle.LifecycleRegistry;
import androidx.lifecycle.ReportFragment;
import androidx.lifecycle.SavedStateViewModelFactory;
import androidx.lifecycle.ViewModelProvider;
import androidx.lifecycle.ViewModelStore;
import androidx.lifecycle.ViewModelStoreOwner;
import androidx.lifecycle.ViewTreeLifecycleOwner;
import androidx.lifecycle.ViewTreeViewModelStoreOwner;
import androidx.savedstate.SavedStateRegistry;
import androidx.savedstate.SavedStateRegistryController;
import androidx.savedstate.SavedStateRegistryOwner;
import androidx.savedstate.ViewTreeSavedStateRegistryOwner;
import androidx.tracing.Trace;

import java.util.concurrent.atomic.AtomicInteger;

/**
 * Base class for activities that enables composition of higher level components.
 * <p>
 * Rather than all functionality being built directly into this class, only the minimal set of
 * lower level building blocks are included. Higher level components can then be used as needed
 * without enforcing a deep Activity class hierarchy or strong coupling between components.
 */
public class ComponentActivity extends androidx.core.app.ComponentActivity implements
        ContextAware,
        LifecycleOwner,
        ViewModelStoreOwner,
        HasDefaultViewModelProviderFactory,
        SavedStateRegistryOwner,
        OnBackPressedDispatcherOwner,
        ActivityResultRegistryOwner,
        ActivityResultCaller {

    static final class NonConfigurationInstances {
        Object custom;
        ViewModelStore viewModelStore;
    }

    final ContextAwareHelper mContextAwareHelper = new ContextAwareHelper();
    private final LifecycleRegistry mLifecycleRegistry = new LifecycleRegistry(this);
    @SuppressWarnings("WeakerAccess") /* synthetic access */
    final SavedStateRegistryController mSavedStateRegistryController =
            SavedStateRegistryController.create(this);

    // Lazily recreated from NonConfigurationInstances by getViewModelStore()
    private ViewModelStore mViewModelStore;
    private ViewModelProvider.Factory mDefaultFactory;

    private final OnBackPressedDispatcher mOnBackPressedDispatcher =
            new OnBackPressedDispatcher(new Runnable() {
                @Override
                public void run() {
                    // Calling onBackPressed() on an Activity with its state saved can cause an
                    // error on devices on API levels before 26. We catch that specific error and
                    // throw all others.
                    try {
                        ComponentActivity.super.onBackPressed();
                    } catch (IllegalStateException e) {
                        if (!TextUtils.equals(e.getMessage(),
                                "Can not perform this action after onSaveInstanceState")) {
                            throw e;
                        }
                    }
                }
            });

    @LayoutRes
    private int mContentLayoutId;

    private final AtomicInteger mNextLocalRequestCode = new AtomicInteger();

    private final ActivityResultRegistry mActivityResultRegistry = new ActivityResultRegistry() {

        @Override
        public <I, O> void onLaunch(
                final int requestCode,
                @NonNull ActivityResultContract<I, O> contract,
                I input,
                @Nullable ActivityOptionsCompat options) {
            ComponentActivity activity = ComponentActivity.this;

            // Immediate result path
            final ActivityResultContract.SynchronousResult<O> synchronousResult =
                    contract.getSynchronousResult(activity, input);
            if (synchronousResult != null) {
                new Handler(Looper.getMainLooper()).post(new Runnable() {
                    @Override
                    public void run() {
                        dispatchResult(requestCode, synchronousResult.getValue());
                    }
                });
                return;
            }

            // Start activity path
            Intent intent = contract.createIntent(activity, input);
            Bundle optionsBundle = null;
            // If there are any extras, we should defensively set the classLoader
            if (intent.getExtras() != null && intent.getExtras().getClassLoader() == null) {
                intent.setExtrasClassLoader(activity.getClassLoader());
            }
            if (intent.hasExtra(EXTRA_ACTIVITY_OPTIONS_BUNDLE)) {
                optionsBundle = intent.getBundleExtra(EXTRA_ACTIVITY_OPTIONS_BUNDLE);
                intent.removeExtra(EXTRA_ACTIVITY_OPTIONS_BUNDLE);
            } else if (options != null) {
                optionsBundle = options.toBundle();
            }
            if (ACTION_REQUEST_PERMISSIONS.equals(intent.getAction())) {

                // requestPermissions path
                String[] permissions = intent.getStringArrayExtra(EXTRA_PERMISSIONS);

                if (permissions == null) {
                    permissions = new String[0];
                }

                ActivityCompat.requestPermissions(activity, permissions, requestCode);
            } else if (ACTION_INTENT_SENDER_REQUEST.equals(intent.getAction())) {
                IntentSenderRequest request =
                        intent.getParcelableExtra(EXTRA_INTENT_SENDER_REQUEST);
                try {
                    // startIntentSenderForResult path
                    ActivityCompat.startIntentSenderForResult(activity, request.getIntentSender(),
                            requestCode, request.getFillInIntent(), request.getFlagsMask(),
                            request.getFlagsValues(), 0, optionsBundle);
                } catch (final IntentSender.SendIntentException e) {
                    new Handler(Looper.getMainLooper()).post(new Runnable() {
                        @Override
                        public void run() {
                            dispatchResult(requestCode, RESULT_CANCELED,
                                    new Intent().setAction(ACTION_INTENT_SENDER_REQUEST)
                                            .putExtra(EXTRA_SEND_INTENT_EXCEPTION, e));
                        }
                    });
                }
            } else {
                // startActivityForResult path
                ActivityCompat.startActivityForResult(activity, intent, requestCode, optionsBundle);
            }
        }
    };

    /**
     * Default constructor for ComponentActivity. All Activities must have a default constructor
     * for API 27 and lower devices or when using the default
     * {@link android.app.AppComponentFactory}.
     */
    public ComponentActivity() {
        Lifecycle lifecycle = getLifecycle();
        //noinspection ConstantConditions
        if (lifecycle == null) {
            throw new IllegalStateException("getLifecycle() returned null in ComponentActivity's "
                    + "constructor. Please make sure you are lazily constructing your Lifecycle "
                    + "in the first call to getLifecycle() rather than relying on field "
                    + "initialization.");
        }
        if (Build.VERSION.SDK_INT >= 19) {
            getLifecycle().addObserver(new LifecycleEventObserver() {
                @Override
                public void onStateChanged(@NonNull LifecycleOwner source,
                        @NonNull Lifecycle.Event event) {
                    if (event == Lifecycle.Event.ON_STOP) {
                        Window window = getWindow();
                        final View decor = window != null ? window.peekDecorView() : null;
                        if (decor != null) {
                            decor.cancelPendingInputEvents();
                        }
                    }
                }
            });
        }
        getLifecycle().addObserver(new LifecycleEventObserver() {
            @Override
            public void onStateChanged(@NonNull LifecycleOwner source,
                    @NonNull Lifecycle.Event event) {
                if (event == Lifecycle.Event.ON_DESTROY) {
                    // Clear out the available context
                    mContextAwareHelper.clearAvailableContext();
                    // And clear the ViewModelStore
                    if (!isChangingConfigurations()) {
                        getViewModelStore().clear();
                    }
                }
            }
        });
        getLifecycle().addObserver(new LifecycleEventObserver() {
            @Override
            public void onStateChanged(@NonNull LifecycleOwner source,
                    @NonNull Lifecycle.Event event) {
                ensureViewModelStore();
                getLifecycle().removeObserver(this);
            }
        });

        if (19 <= SDK_INT && SDK_INT <= 23) {
            getLifecycle().addObserver(new ImmLeaksCleaner(this));
        }
    }

    /**
     * Alternate constructor that can be used to provide a default layout
     * that will be inflated as part of <code>super.onCreate(savedInstanceState)</code>.
     *
     * <p>This should generally be called from your constructor that takes no parameters,
     * as is required for API 27 and lower or when using the default
     * {@link android.app.AppComponentFactory}.
     *
     * @see #ComponentActivity()
     */
    @ContentView
    public ComponentActivity(@LayoutRes int contentLayoutId) {
        this();
        mContentLayoutId = contentLayoutId;
    }

    /**
     * {@inheritDoc}
     *
     * If your ComponentActivity is annotated with {@link ContentView}, this will
     * call {@link #setContentView(int)} for you.
     */
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        // Restore the Saved State first so that it is available to
        // OnContextAvailableListener instances
        mSavedStateRegistryController.performRestore(savedInstanceState);
        mContextAwareHelper.dispatchOnContextAvailable(this);
        super.onCreate(savedInstanceState);
        mActivityResultRegistry.onRestoreInstanceState(savedInstanceState);
        ReportFragment.injectIfNeededIn(this);
        if (mContentLayoutId != 0) {
            setContentView(mContentLayoutId);
        }
    }

    @CallSuper
    @Override
    protected void onSaveInstanceState(@NonNull Bundle outState) {
        Lifecycle lifecycle = getLifecycle();
        if (lifecycle instanceof LifecycleRegistry) {
            ((LifecycleRegistry) lifecycle).setCurrentState(Lifecycle.State.CREATED);
        }
        super.onSaveInstanceState(outState);
        mSavedStateRegistryController.performSave(outState);
        mActivityResultRegistry.onSaveInstanceState(outState);
    }

    /**
     * Retain all appropriate non-config state.  You can NOT
     * override this yourself!  Use a {@link androidx.lifecycle.ViewModel} if you want to
     * retain your own non config state.
     */
    @Override
    @Nullable
    @SuppressWarnings("deprecation")
    public final Object onRetainNonConfigurationInstance() {
        // Maintain backward compatibility.
        Object custom = onRetainCustomNonConfigurationInstance();

        ViewModelStore viewModelStore = mViewModelStore;
        if (viewModelStore == null) {
            // No one called getViewModelStore(), so see if there was an existing
            // ViewModelStore from our last NonConfigurationInstance
            NonConfigurationInstances nc =
                    (NonConfigurationInstances) getLastNonConfigurationInstance();
            if (nc != null) {
                viewModelStore = nc.viewModelStore;
            }
        }

        if (viewModelStore == null && custom == null) {
            return null;
        }

        NonConfigurationInstances nci = new NonConfigurationInstances();
        nci.custom = custom;
        nci.viewModelStore = viewModelStore;
        return nci;
    }

    /**
     * Use this instead of {@link #onRetainNonConfigurationInstance()}.
     * Retrieve later with {@link #getLastCustomNonConfigurationInstance()}.
     *
     * @deprecated Use a {@link androidx.lifecycle.ViewModel} to store non config state.
     */
    @Deprecated
    @Nullable
    public Object onRetainCustomNonConfigurationInstance() {
        return null;
    }

    /**
     * Return the value previously returned from
     * {@link #onRetainCustomNonConfigurationInstance()}.
     *
     * @deprecated Use a {@link androidx.lifecycle.ViewModel} to store non config state.
     */
    @Deprecated
    @Nullable
    public Object getLastCustomNonConfigurationInstance() {
        NonConfigurationInstances nc = (NonConfigurationInstances)
                getLastNonConfigurationInstance();
        return nc != null ? nc.custom : null;
    }

    @Override
    public void setContentView(@LayoutRes int layoutResID) {
        initViewTreeOwners();
        super.setContentView(layoutResID);
    }

    @Override
    public void setContentView(@SuppressLint({"UnknownNullness", "MissingNullability"}) View view) {
        initViewTreeOwners();
        super.setContentView(view);
    }

    @Override
    public void setContentView(@SuppressLint({"UnknownNullness", "MissingNullability"}) View view,
            @SuppressLint({"UnknownNullness", "MissingNullability"})
                    ViewGroup.LayoutParams params) {
        initViewTreeOwners();
        super.setContentView(view, params);
    }

    @Override
    public void addContentView(@SuppressLint({"UnknownNullness", "MissingNullability"}) View view,
            @SuppressLint({"UnknownNullness", "MissingNullability"})
                    ViewGroup.LayoutParams params) {
        initViewTreeOwners();
        super.addContentView(view, params);
    }

    private void initViewTreeOwners() {
        // Set the view tree owners before setting the content view so that the inflation process
        // and attach listeners will see them already present
        ViewTreeLifecycleOwner.set(getWindow().getDecorView(), this);
        ViewTreeViewModelStoreOwner.set(getWindow().getDecorView(), this);
        ViewTreeSavedStateRegistryOwner.set(getWindow().getDecorView(), this);
    }

    @Nullable
    @Override
    public Context peekAvailableContext() {
        return mContextAwareHelper.peekAvailableContext();
    }

    /**
     * {@inheritDoc}
     *
     * Any listener added here will receive a callback as part of
     * <code>super.onCreate()</code>, but importantly <strong>before</strong> any other
     * logic is done (including calling through to the framework
     * {@link Activity#onCreate(Bundle)} with the exception of restoring the state
     * of the {@link #getSavedStateRegistry() SavedStateRegistry} for use in your listener.
     */
    @Override
    public final void addOnContextAvailableListener(
            @NonNull OnContextAvailableListener listener) {
        mContextAwareHelper.addOnContextAvailableListener(listener);
    }

    @Override
    public final void removeOnContextAvailableListener(
            @NonNull OnContextAvailableListener listener) {
        mContextAwareHelper.removeOnContextAvailableListener(listener);
    }

    /**
     * {@inheritDoc}
     * <p>
     * Overriding this method is no longer supported and this method will be made
     * <code>final</code> in a future version of ComponentActivity. If you do override
     * this method, you <code>must</code>:
     * <ol>
     *     <li>Return an instance of {@link LifecycleRegistry}</li>
     *     <li>Lazily initialize your LifecycleRegistry object when this is first called.
     *     Note that this method will be called in the super classes' constructor, before any
     *     field initialization or object state creation is complete.</li>
     * </ol>
     */
    @NonNull
    @Override
    public Lifecycle getLifecycle() {
        return mLifecycleRegistry;
    }

    /**
     * Returns the {@link ViewModelStore} associated with this activity
     * <p>
     * Overriding this method is no longer supported and this method will be made
     * <code>final</code> in a future version of ComponentActivity.
     *
     * @return a {@code ViewModelStore}
     * @throws IllegalStateException if called before the Activity is attached to the Application
     * instance i.e., before onCreate()
     */
    @NonNull
    @Override
    public ViewModelStore getViewModelStore() {
        if (getApplication() == null) {
            throw new IllegalStateException("Your activity is not yet attached to the "
                    + "Application instance. You can't request ViewModel before onCreate call.");
        }
        ensureViewModelStore();
        return mViewModelStore;
    }

    @SuppressWarnings("WeakerAccess") /* synthetic access */
    void ensureViewModelStore() {
        if (mViewModelStore == null) {
            NonConfigurationInstances nc =
                    (NonConfigurationInstances) getLastNonConfigurationInstance();
            if (nc != null) {
                // Restore the ViewModelStore from NonConfigurationInstances
                mViewModelStore = nc.viewModelStore;
            }
            if (mViewModelStore == null) {
                mViewModelStore = new ViewModelStore();
            }
        }
    }

    /**
     * {@inheritDoc}
     *
     * <p>The extras of {@link #getIntent()} when this is first called will be used as
     * the defaults to any {@link androidx.lifecycle.SavedStateHandle} passed to a view model
     * created using this factory.</p>
     */
    @NonNull
    @Override
    public ViewModelProvider.Factory getDefaultViewModelProviderFactory() {
        if (getApplication() == null) {
            throw new IllegalStateException("Your activity is not yet attached to the "
                    + "Application instance. You can't request ViewModel before onCreate call.");
        }
        if (mDefaultFactory == null) {
            mDefaultFactory = new SavedStateViewModelFactory(
                    getApplication(),
                    this,
                    getIntent() != null ? getIntent().getExtras() : null);
        }
        return mDefaultFactory;
    }

    /**
     * Called when the activity has detected the user's press of the back
     * key. The {@link #getOnBackPressedDispatcher() OnBackPressedDispatcher} will be given a
     * chance to handle the back button before the default behavior of
     * {@link android.app.Activity#onBackPressed()} is invoked.
     *
     * @see #getOnBackPressedDispatcher()
     */
    @Override
    @MainThread
    public void onBackPressed() {
        mOnBackPressedDispatcher.onBackPressed();
    }

    /**
     * Retrieve the {@link OnBackPressedDispatcher} that will be triggered when
     * {@link #onBackPressed()} is called.
     * @return The {@link OnBackPressedDispatcher} associated with this ComponentActivity.
     */
    @NonNull
    @Override
    public final OnBackPressedDispatcher getOnBackPressedDispatcher() {
        return mOnBackPressedDispatcher;
    }

    @NonNull
    @Override
    public final SavedStateRegistry getSavedStateRegistry() {
        return mSavedStateRegistryController.getSavedStateRegistry();
    }

    /**
     * {@inheritDoc}
     *
     * @deprecated use
     * {@link #registerForActivityResult(ActivityResultContract, ActivityResultCallback)}
     * passing in a {@link StartActivityForResult} object for the {@link ActivityResultContract}.
     */
    @Override
    @Deprecated
    public void startActivityForResult(@SuppressLint("UnknownNullness") Intent intent,
            int requestCode) {
        super.startActivityForResult(intent, requestCode);
    }

    /**
     * {@inheritDoc}
     *
     * @deprecated use
     * {@link #registerForActivityResult(ActivityResultContract, ActivityResultCallback)}
     * passing in a {@link StartActivityForResult} object for the {@link ActivityResultContract}.
     */
    @Override
    @Deprecated
    public void startActivityForResult(@SuppressLint("UnknownNullness") Intent intent,
            int requestCode, @Nullable Bundle options) {
        super.startActivityForResult(intent, requestCode, options);
    }

    /**
     * {@inheritDoc}
     *
     * @deprecated use
     * {@link #registerForActivityResult(ActivityResultContract, ActivityResultCallback)}
     * passing in a {@link StartIntentSenderForResult} object for the
     * {@link ActivityResultContract}.
     */
    @Override
    @Deprecated
    public void startIntentSenderForResult(@SuppressLint("UnknownNullness") IntentSender intent,
            int requestCode, @Nullable Intent fillInIntent, int flagsMask, int flagsValues,
            int extraFlags)
            throws IntentSender.SendIntentException {
        super.startIntentSenderForResult(intent, requestCode, fillInIntent, flagsMask, flagsValues,
                extraFlags);
    }

    /**
     * {@inheritDoc}
     *
     * @deprecated use
     * {@link #registerForActivityResult(ActivityResultContract, ActivityResultCallback)}
     * passing in a {@link StartIntentSenderForResult} object for the
     * {@link ActivityResultContract}.
     */
    @Override
    @Deprecated
    public void startIntentSenderForResult(@SuppressLint("UnknownNullness") IntentSender intent,
            int requestCode, @Nullable Intent fillInIntent, int flagsMask, int flagsValues,
            int extraFlags, @Nullable Bundle options) throws IntentSender.SendIntentException {
        super.startIntentSenderForResult(intent, requestCode, fillInIntent, flagsMask, flagsValues,
                extraFlags, options);
    }

    /**
     * {@inheritDoc}
     *
     * @deprecated use
     * {@link #registerForActivityResult(ActivityResultContract, ActivityResultCallback)}
     * with the appropriate {@link ActivityResultContract} and handling the result in the
     * {@link ActivityResultCallback#onActivityResult(Object) callback}.
     */
    @CallSuper
    @Override
    @Deprecated
    protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
        if (!mActivityResultRegistry.dispatchResult(requestCode, resultCode, data)) {
            super.onActivityResult(requestCode, resultCode, data);
        }
    }

    /**
     * {@inheritDoc}
     *
     * @deprecated use
     * {@link #registerForActivityResult(ActivityResultContract, ActivityResultCallback)} passing
     * in a {@link RequestMultiplePermissions} object for the {@link ActivityResultContract} and
     * handling the result in the {@link ActivityResultCallback#onActivityResult(Object) callback}.
     */
    @CallSuper
    @Override
    @Deprecated
    public void onRequestPermissionsResult(
            int requestCode,
            @NonNull String[] permissions,
            @NonNull int[] grantResults) {
        if (!mActivityResultRegistry.dispatchResult(requestCode, Activity.RESULT_OK, new Intent()
                .putExtra(EXTRA_PERMISSIONS, permissions)
                .putExtra(EXTRA_PERMISSION_GRANT_RESULTS, grantResults))) {
            if (Build.VERSION.SDK_INT >= 23) {
                super.onRequestPermissionsResult(requestCode, permissions, grantResults);
            }
        }
    }

    @NonNull
    @Override
    public final <I, O> ActivityResultLauncher<I> registerForActivityResult(
            @NonNull final ActivityResultContract<I, O> contract,
            @NonNull final ActivityResultRegistry registry,
            @NonNull final ActivityResultCallback<O> callback) {
        return registry.register(
                "activity_rq#" + mNextLocalRequestCode.getAndIncrement(), this, contract, callback);
    }

    @NonNull
    @Override
    public final <I, O> ActivityResultLauncher<I> registerForActivityResult(
            @NonNull ActivityResultContract<I, O> contract,
            @NonNull ActivityResultCallback<O> callback) {
        return registerForActivityResult(contract, mActivityResultRegistry, callback);
    }

    /**
     * Get the {@link ActivityResultRegistry} associated with this activity.
     *
     * @return the {@link ActivityResultRegistry}
     */
    @NonNull
    @Override
    public final ActivityResultRegistry getActivityResultRegistry() {
        return mActivityResultRegistry;
    }

    @Override
    public void reportFullyDrawn() {
        try {
            if (Trace.isEnabled()) {
                Trace.beginSection("reportFullyDrawn() for " + getComponentName());
            }

            if (Build.VERSION.SDK_INT > 19) {
                super.reportFullyDrawn();
            } else if (Build.VERSION.SDK_INT == 19 && ContextCompat.checkSelfPermission(this,
                    Manifest.permission.UPDATE_DEVICE_STATS) == PackageManager.PERMISSION_GRANTED) {
                // On API 19, the Activity.reportFullyDrawn() method requires the
                // UPDATE_DEVICE_STATS permission, otherwise it throws an exception. Instead of
                // throwing, we fall back to a no-op call.
                super.reportFullyDrawn();
            }
            // The Activity.reportFullyDrawn() got added in API 19, fall back to a no-op call if
            // this method gets called on devices with an earlier version.
        } finally {
            Trace.endSection();
        }
    }
}

```

主要看 ` register`

```java
/*
 * Copyright (C) 2020 The Android Open Source Project
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */


package androidx.activity.result;

import android.annotation.SuppressLint;
import android.app.Activity;
import android.content.Intent;
import android.os.Bundle;
import android.util.Log;

import androidx.activity.result.contract.ActivityResultContract;
import androidx.annotation.MainThread;
import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import androidx.core.app.ActivityOptionsCompat;
import androidx.lifecycle.Lifecycle;
import androidx.lifecycle.LifecycleEventObserver;
import androidx.lifecycle.LifecycleOwner;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.Map;
import java.util.Random;

/**
 * A registry that stores {@link ActivityResultCallback activity result callbacks} for
 * {@link ActivityResultCaller#registerForActivityResult registered calls}.
 *
 * You can create your own instance for testing by overriding {@link #onLaunch} and calling
 * {@link #dispatchResult} immediately within it, thus skipping the actual
 * {@link Activity#startActivityForResult} call.
 *
 * When testing, make sure to explicitly provide a registry instance whenever calling
 * {@link ActivityResultCaller#registerForActivityResult}, to be able to inject a test instance.
 */
public abstract class ActivityResultRegistry {
    private static final String KEY_COMPONENT_ACTIVITY_REGISTERED_RCS =
            "KEY_COMPONENT_ACTIVITY_REGISTERED_RCS";
    private static final String KEY_COMPONENT_ACTIVITY_REGISTERED_KEYS =
            "KEY_COMPONENT_ACTIVITY_REGISTERED_KEYS";
    private static final String KEY_COMPONENT_ACTIVITY_LAUNCHED_KEYS =
            "KEY_COMPONENT_ACTIVITY_LAUNCHED_KEYS";
    private static final String KEY_COMPONENT_ACTIVITY_PENDING_RESULTS =
            "KEY_COMPONENT_ACTIVITY_PENDING_RESULT";
    private static final String KEY_COMPONENT_ACTIVITY_RANDOM_OBJECT =
            "KEY_COMPONENT_ACTIVITY_RANDOM_OBJECT";

    private static final String LOG_TAG = "ActivityResultRegistry";

    // Use upper 16 bits for request codes
    private static final int INITIAL_REQUEST_CODE_VALUE = 0x00010000;
    private Random mRandom = new Random();

    private final Map<Integer, String> mRcToKey = new HashMap<>();
    private final Map<String, Integer> mKeyToRc = new HashMap<>();
    private final Map<String, LifecycleContainer> mKeyToLifecycleContainers = new HashMap<>();
    ArrayList<String> mLaunchedKeys = new ArrayList<>();

    @SuppressWarnings("WeakerAccess") /* synthetic access */
    final transient Map<String, CallbackAndContract<?>> mKeyToCallback = new HashMap<>();

    @SuppressWarnings("WeakerAccess") /* synthetic access */
    final Map<String, Object> mParsedPendingResults = new HashMap<>();
    @SuppressWarnings("WeakerAccess") /* synthetic access */
    final Bundle/*<String, ActivityResult>*/ mPendingResults = new Bundle();

    /**
     * Start the process of executing an {@link ActivityResultContract} in a type-safe way,
     * using the provided {@link ActivityResultContract contract}.
     *
     * @param requestCode request code to use
     * @param contract contract to use for type conversions
     * @param input input required to execute an ActivityResultContract.
     * @param options Additional options for how the Activity should be started.
     */
    @MainThread
    public abstract <I, O> void onLaunch(
            int requestCode,
            @NonNull ActivityResultContract<I, O> contract,
            @SuppressLint("UnknownNullness") I input,
            @Nullable ActivityOptionsCompat options);

    /**
     * Register a new callback with this registry.
     *
     * This is normally called by a higher level convenience methods like
     * {@link ActivityResultCaller#registerForActivityResult}.
     *
     * @param key a unique string key identifying this call
     * @param lifecycleOwner a {@link LifecycleOwner} that makes this call.
     * @param contract the contract specifying input/output types of the call
     * @param callback the activity result callback
     *
     * @return a launcher that can be used to execute an ActivityResultContract.
     */
    @NonNull
    public final <I, O> ActivityResultLauncher<I> register(
            @NonNull final String key,
            @NonNull final LifecycleOwner lifecycleOwner,
            @NonNull final ActivityResultContract<I, O> contract,
            @NonNull final ActivityResultCallback<O> callback) {

        Lifecycle lifecycle = lifecycleOwner.getLifecycle();

        if (lifecycle.getCurrentState().isAtLeast(Lifecycle.State.STARTED)) {
            throw new IllegalStateException("LifecycleOwner " + lifecycleOwner + " is "
                    + "attempting to register while current state is "
                    + lifecycle.getCurrentState() + ". LifecycleOwners must call register before "
                    + "they are STARTED.");
        }

        final int requestCode = registerKey(key);
        LifecycleContainer lifecycleContainer = mKeyToLifecycleContainers.get(key);
        if (lifecycleContainer == null) {
            lifecycleContainer = new LifecycleContainer(lifecycle);
        }
        LifecycleEventObserver observer = new LifecycleEventObserver() {
            @Override
            public void onStateChanged(
                    @NonNull LifecycleOwner lifecycleOwner,
                    @NonNull Lifecycle.Event event) {
                if (Lifecycle.Event.ON_START.equals(event)) {
                    mKeyToCallback.put(key, new CallbackAndContract<>(callback, contract));
                    if (mParsedPendingResults.containsKey(key)) {
                        @SuppressWarnings("unchecked")
                        final O parsedPendingResult = (O) mParsedPendingResults.get(key);
                        mParsedPendingResults.remove(key);
                        callback.onActivityResult(parsedPendingResult);
                    }
                    final ActivityResult pendingResult = mPendingResults.getParcelable(key);
                    if (pendingResult != null) {
                        mPendingResults.remove(key);
                        callback.onActivityResult(contract.parseResult(
                                pendingResult.getResultCode(),
                                pendingResult.getData()));
                    }
                } else if (Lifecycle.Event.ON_STOP.equals(event)) {
                    mKeyToCallback.remove(key);
                } else if (Lifecycle.Event.ON_DESTROY.equals(event)) {
                    unregister(key);
                }
            }
        };
        lifecycleContainer.addObserver(observer);
        mKeyToLifecycleContainers.put(key, lifecycleContainer);

        return new ActivityResultLauncher<I>() {
            @Override
            public void launch(I input, @Nullable ActivityOptionsCompat options) {
                mLaunchedKeys.add(key);
                onLaunch(requestCode, contract, input, options);
            }

            @Override
            public void unregister() {
                ActivityResultRegistry.this.unregister(key);
            }

            @NonNull
            @Override
            public ActivityResultContract<I, ?> getContract() {
                return contract;
            }
        };
    }

    /**
     * Register a new callback with this registry.
     *
     * This is normally called by a higher level convenience methods like
     * {@link ActivityResultCaller#registerForActivityResult}.
     *
     * When calling this, you must call {@link ActivityResultLauncher#unregister()} on the
     * returned {@link ActivityResultLauncher} when the launcher is no longer needed to
     * release any values that might be captured in the registered callback.
     *
     * @param key a unique string key identifying this call
     * @param contract the contract specifying input/output types of the call
     * @param callback the activity result callback
     *
     * @return a launcher that can be used to execute an ActivityResultContract.
     */
    @NonNull
    public final <I, O> ActivityResultLauncher<I> register(
            @NonNull final String key,
            @NonNull final ActivityResultContract<I, O> contract,
            @NonNull final ActivityResultCallback<O> callback) {
        final int requestCode = registerKey(key);
        mKeyToCallback.put(key, new CallbackAndContract<>(callback, contract));

        if (mParsedPendingResults.containsKey(key)) {
            @SuppressWarnings("unchecked")
            final O parsedPendingResult = (O) mParsedPendingResults.get(key);
            mParsedPendingResults.remove(key);
            callback.onActivityResult(parsedPendingResult);
        }
        final ActivityResult pendingResult = mPendingResults.getParcelable(key);
        if (pendingResult != null) {
            mPendingResults.remove(key);
            callback.onActivityResult(contract.parseResult(
                    pendingResult.getResultCode(),
                    pendingResult.getData()));
        }

        return new ActivityResultLauncher<I>() {
            @Override
            public void launch(I input, @Nullable ActivityOptionsCompat options) {
                mLaunchedKeys.add(key);
                onLaunch(requestCode, contract, input, options);
            }

            @Override
            public void unregister() {
                ActivityResultRegistry.this.unregister(key);
            }

            @NonNull
            @Override
            public ActivityResultContract<I, ?> getContract() {
                return contract;
            }
        };
    }

    /**
     * Unregister a callback previously registered with {@link #register}. This shouldn't be
     * called directly, but instead through {@link ActivityResultLauncher#unregister()}.
     *
     * @param key the unique key used when registering a callback.
     */
    @MainThread
    final void unregister(@NonNull String key) {
        if (!mLaunchedKeys.contains(key)) {
            // Only remove the key -> requestCode mapping if there isn't a launch in flight
            Integer rc = mKeyToRc.remove(key);
            if (rc != null) {
                mRcToKey.remove(rc);
            }
        }
        mKeyToCallback.remove(key);
        if (mParsedPendingResults.containsKey(key)) {
            Log.w(LOG_TAG, "Dropping pending result for request " + key + ": "
                    + mParsedPendingResults.get(key));
            mParsedPendingResults.remove(key);
        }
        if (mPendingResults.containsKey(key)) {
            Log.w(LOG_TAG, "Dropping pending result for request " + key + ": "
                    + mPendingResults.<ActivityResult>getParcelable(key));
            mPendingResults.remove(key);
        }
        LifecycleContainer lifecycleContainer = mKeyToLifecycleContainers.get(key);
        if (lifecycleContainer != null) {
            lifecycleContainer.clearObservers();
            mKeyToLifecycleContainers.remove(key);
        }
    }

    /**
     * Save the state of this registry in the given {@link Bundle}
     *
     * @param outState the place to put state into
     */
    public final void onSaveInstanceState(@NonNull Bundle outState) {
        outState.putIntegerArrayList(KEY_COMPONENT_ACTIVITY_REGISTERED_RCS,
                new ArrayList<>(mRcToKey.keySet()));
        outState.putStringArrayList(KEY_COMPONENT_ACTIVITY_REGISTERED_KEYS,
                new ArrayList<>(mRcToKey.values()));
        outState.putStringArrayList(KEY_COMPONENT_ACTIVITY_LAUNCHED_KEYS,
                new ArrayList<>(mLaunchedKeys));
        outState.putBundle(KEY_COMPONENT_ACTIVITY_PENDING_RESULTS,
                (Bundle) mPendingResults.clone());
        outState.putSerializable(KEY_COMPONENT_ACTIVITY_RANDOM_OBJECT, mRandom);
    }

    /**
     * Restore the state of this registry from the given {@link Bundle}
     *
     * @param savedInstanceState the place to restore from
     */
    public final void onRestoreInstanceState(@Nullable Bundle savedInstanceState) {
        if (savedInstanceState == null) {
            return;
        }
        ArrayList<Integer> rcs =
                savedInstanceState.getIntegerArrayList(KEY_COMPONENT_ACTIVITY_REGISTERED_RCS);
        ArrayList<String> keys =
                savedInstanceState.getStringArrayList(KEY_COMPONENT_ACTIVITY_REGISTERED_KEYS);
        if (keys == null || rcs == null) {
            return;
        }
        int numKeys = keys.size();
        for (int i = 0; i < numKeys; i++) {
            bindRcKey(rcs.get(i), keys.get(i));
        }
        mLaunchedKeys =
                savedInstanceState.getStringArrayList(KEY_COMPONENT_ACTIVITY_LAUNCHED_KEYS);
        mRandom = (Random) savedInstanceState.getSerializable(KEY_COMPONENT_ACTIVITY_RANDOM_OBJECT);
        mPendingResults.putAll(
                savedInstanceState.getBundle(KEY_COMPONENT_ACTIVITY_PENDING_RESULTS));
    }

    /**
     * Dispatch a result received via {@link Activity#onActivityResult} to the callback on record,
     * or store the result if callback was not yet registered.
     *
     * @param requestCode request code to identify the callback
     * @param resultCode status to indicate the success of the operation
     * @param data an intent that carries the result data
     *
     * @return whether there was a callback was registered for the given request code which was
     * or will be called.
     */
    @MainThread
    public final boolean dispatchResult(int requestCode, int resultCode, @Nullable Intent data) {
        String key = mRcToKey.get(requestCode);
        if (key == null) {
            return false;
        }
        mLaunchedKeys.remove(key);

        doDispatch(key, resultCode, data, mKeyToCallback.get(key));
        return true;
    }

    /**
     * Dispatch a result object to the callback on record.
     *
     * @param requestCode request code to identify the callback
     * @param result the result to propagate
     *
     * @return true if there is a callback registered for the given request code, false otherwise.
     */
    @MainThread
    public final <O> boolean dispatchResult(int requestCode,
            @SuppressLint("UnknownNullness") O result) {
        String key = mRcToKey.get(requestCode);
        if (key == null) {
            return false;
        }
        mLaunchedKeys.remove(key);

        CallbackAndContract<?> callbackAndContract = mKeyToCallback.get(key);
        if (callbackAndContract == null || callbackAndContract.mCallback == null) {
            // Remove any pending result
            mPendingResults.remove(key);
            // And add these pre-parsed pending results in their place
            mParsedPendingResults.put(key, result);
        } else {
            @SuppressWarnings("unchecked")
            ActivityResultCallback<O> callback =
                    (ActivityResultCallback<O>) callbackAndContract.mCallback;
            callback.onActivityResult(result);
        }
        return true;
    }

    private <O> void doDispatch(String key, int resultCode, @Nullable Intent data,
            @Nullable CallbackAndContract<O> callbackAndContract) {
        if (callbackAndContract != null && callbackAndContract.mCallback != null) {
            ActivityResultCallback<O> callback = callbackAndContract.mCallback;
            ActivityResultContract<?, O> contract = callbackAndContract.mContract;
            callback.onActivityResult(contract.parseResult(resultCode, data));
        } else {
            // Remove any parsed pending result
            mParsedPendingResults.remove(key);
            // And add these pending results in their place
            mPendingResults.putParcelable(key, new ActivityResult(resultCode, data));
        }
    }

    private int registerKey(String key) {
        Integer existing = mKeyToRc.get(key);
        if (existing != null) {
            return existing;
        }
        int rc = generateRandomNumber();
        bindRcKey(rc, key);
        return rc;
    }

    /**
     * Generate a random number between the initial value (00010000) inclusive, and the max
     * integer value. If that number is already an existing request code, generate another until
     * we find one that is new.
     *
     * @return the number
     */
    private int generateRandomNumber() {
        int number = mRandom.nextInt((Integer.MAX_VALUE - INITIAL_REQUEST_CODE_VALUE) + 1)
                + INITIAL_REQUEST_CODE_VALUE;
        while (mRcToKey.containsKey(number)) {
            number = mRandom.nextInt((Integer.MAX_VALUE - INITIAL_REQUEST_CODE_VALUE) + 1)
                    + INITIAL_REQUEST_CODE_VALUE;
        }
        return number;
    }

    private void bindRcKey(int rc, String key) {
        mRcToKey.put(rc, key);
        mKeyToRc.put(key, rc);
    }

    private static class CallbackAndContract<O> {
        final ActivityResultCallback<O> mCallback;
        final ActivityResultContract<?, O> mContract;

        CallbackAndContract(
                ActivityResultCallback<O> callback,
                ActivityResultContract<?, O> contract) {
            mCallback = callback;
            mContract = contract;
        }
    }

    private static class LifecycleContainer {
        final Lifecycle mLifecycle;
        private final ArrayList<LifecycleEventObserver> mObservers;

        LifecycleContainer(@NonNull Lifecycle lifecycle) {
            mLifecycle = lifecycle;
            mObservers = new ArrayList<>();
        }

        void addObserver(@NonNull LifecycleEventObserver observer) {
            mLifecycle.addObserver(observer);
            mObservers.add(observer);
        }

        void clearObservers() {
            for (LifecycleEventObserver observer: mObservers) {
                mLifecycle.removeObserver(observer);
            }
            mObservers.clear();
        }
    }
}

```

可见 `registerForActivityResult`返回的 `ActivityResultLauncher`是由 `ActivityResultRegistry`的register方法返回的，而 `ActivityResultRegistry`是真正干活的地方。`ActivityResultLauncher `的launch 调用了 `ActivityResultRegistry`的onLaunch，里面就是我们之前都熟悉的代码使用了。到这里是 `启动`的过程。返回 `结果`要看 `ActivityResultLauncher`的dispatchResult方法。这个方法对返回结果进行分发到 `ActivityResultCallback`中。

```
private final ActivityResultRegistry mActivityResultRegistry = new ActivityResultRegistry() {

        @Override
        public <I, O> void onLaunch(
                final int requestCode,
                @NonNull ActivityResultContract<I, O> contract,
                I input,
                @Nullable ActivityOptionsCompat options) {
            ComponentActivity activity = ComponentActivity.this;

            // Immediate result path
            final ActivityResultContract.SynchronousResult<O> synchronousResult =
                    contract.getSynchronousResult(activity, input);
            if (synchronousResult != null) {
                new Handler(Looper.getMainLooper()).post(new Runnable() {
                    @Override
                    public void run() {
                        dispatchResult(requestCode, synchronousResult.getValue());
                    }
                });
                return;
            }

            // Start activity path
            Intent intent = contract.createIntent(activity, input);
            Bundle optionsBundle = null;
            // If there are any extras, we should defensively set the classLoader
            if (intent.getExtras() != null && intent.getExtras().getClassLoader() == null) {
                intent.setExtrasClassLoader(activity.getClassLoader());
            }
            if (intent.hasExtra(EXTRA_ACTIVITY_OPTIONS_BUNDLE)) {
                optionsBundle = intent.getBundleExtra(EXTRA_ACTIVITY_OPTIONS_BUNDLE);
                intent.removeExtra(EXTRA_ACTIVITY_OPTIONS_BUNDLE);
            } else if (options != null) {
                optionsBundle = options.toBundle();
            }
            if (ACTION_REQUEST_PERMISSIONS.equals(intent.getAction())) {

                // requestPermissions path
                String[] permissions = intent.getStringArrayExtra(EXTRA_PERMISSIONS);

                if (permissions == null) {
                    permissions = new String[0];
                }

                ActivityCompat.requestPermissions(activity, permissions, requestCode);
            } else if (ACTION_INTENT_SENDER_REQUEST.equals(intent.getAction())) {
                IntentSenderRequest request =
                        intent.getParcelableExtra(EXTRA_INTENT_SENDER_REQUEST);
                try {
                    // startIntentSenderForResult path
                    ActivityCompat.startIntentSenderForResult(activity, request.getIntentSender(),
                            requestCode, request.getFillInIntent(), request.getFlagsMask(),
                            request.getFlagsValues(), 0, optionsBundle);
                } catch (final IntentSender.SendIntentException e) {
                    new Handler(Looper.getMainLooper()).post(new Runnable() {
                        @Override
                        public void run() {
                            dispatchResult(requestCode, RESULT_CANCELED,
                                    new Intent().setAction(ACTION_INTENT_SENDER_REQUEST)
                                            .putExtra(EXTRA_SEND_INTENT_EXCEPTION, e));
                        }
                    });
                }
            } else {
                // startActivityForResult path
                ActivityCompat.startActivityForResult(activity, intent, requestCode, optionsBundle);
            }
        }
    };
```

##### 预定义的Contract

Google 预定义了很多Contract,把你们能想到的使用场景基本上都想到了，它们都定义在类 `ActivityResultContracts`中
