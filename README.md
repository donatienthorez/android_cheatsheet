# Android cheatsheet #

This cheatsheet provides all useful things that every Android developer need and forget sometimes. Do not hesitate to make pull request if you think something important is missing or if there is a mistake.

## Table of Contents
  - [Activity](#activity)
    - [Activity lifecycle](#activity-lifecycle)
        - [Methods](#methods)
        - [Some common situations](#some-common-situations)
    - [Instance state](#instance-state)
    - [Define activity as launch activity](#define-activity-as-launch-activity)
    - [Start another activity](#start-another-activity)
        - [Without any data](#without-any-data)
        - [Without data](#without-data)
        - [Start activity for result](#start-activity-for-result)
  - [Layout and views](#layout-and-views)
    - [The different types of layout and views](#the-different-types-of-layout-and-views)
    - [Create a custom view](#create-a-custom-view)

## Activity ##
### Activity lifecycle ###

#### Methods ####
    protected void onCreate(Bundle savedInstanceState);

Called when the activity is first created. This is where you should do all of your normal static set up: 
create views, bind data to lists, etc. This method also provides you with a Bundle containing the 
activity's previously frozen state, if there was one. Always followed by onStart().

    protected void onStart();

Called after your activity has been stopped, prior to it being started again. Always followed by onStart().

    protected void onRestart();

Called when the activity is becoming visible to the user. Followed by onResume() if the activity comes to the foreground, or onStop() if it becomes hidden.

    protected void onResume();

Called when the activity will start interacting with the user. At this point your activity is at the top of the activity stack, with user input going to it. Always followed by onPause().

    protected void onPause();

Called as part of the activity lifecycle when an activity is going into the background, but has not (yet) been killed. The counterpart to onResume(). When activity B is launched in front of activity A, this callback will be invoked on A. B will not be created until A's onPause() returns, so be sure to not do anything lengthy here.

    protected void onStop();

Called when you are no longer visible to the user. You will next receive either onRestart(), onDestroy(), or nothing, depending on later user activity.

Note that this method may never be called, in low memory situations where the system does not have enough memory to keep your activity's process running after its onPause() method is called.

    protected void onDestroy();

The final call you receive before your activity is destroyed. This can happen either because the activity is finishing (someone called finish() on it, or because the system is temporarily destroying this instance of the activity to save space. You can distinguish between these two scenarios with the isFinishing() method.

#### Some common situations ####

* The user opens the app
>    onCreate() -> onStart() ->  onResume()

* The user presses home button (App is to the background)
>    onPaused() -> onStop()

* The user reopens the app (background to foreground)
>    onRestart() -> onStart() -> onResume()

* The user turn his phone (orientation changed)
> onPause -> onStop() -> onStop() -> onDestroy() -> onCreate() -> onStart() -> onResume()

> The activity is destroyed and recreated, this can be changed by setting the configChanges parameter in the Manifest.


The full diagram can be find [here](https://developer.android.com/images/activity_lifecycle.png)
More information [here](https://stackoverflow.com/questions/8515936/android-activity-life-cycle-what-are-all-these-methods-for) and the default documentation [here](https://developer.android.com/guide/components/activities/activity-lifecycle.html)

### Instance state

Saving instance state

    static final String STATE_SCORE = "playerScore";
    static final String STATE_LEVEL = "playerLevel";

    @Override
    public void onSaveInstanceState(Bundle savedInstanceState) {
        // Save the user's current game state
        savedInstanceState.putInt(STATE_SCORE, mCurrentScore);
        savedInstanceState.putInt(STATE_LEVEL, mCurrentLevel);

        super.onSaveInstanceState(savedInstanceState);
    }

Restoring instance state

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState); // Always call the superclass first

        // Check whether we're recreating a previously destroyed instance
        if (savedInstanceState != null) {
            // Restore value of members from saved state
            mCurrentScore = savedInstanceState.getInt(STATE_SCORE);
            mCurrentLevel = savedInstanceState.getInt(STATE_LEVEL);
        } else {
            // Probably initialize members with default values for a new instance
        }
        ...
    }

You can also choose to override onRestoreInstanceState(). In that case you don't need to check if savedInstanceState is null.

### Define activity as launch activity
```xml
    <manifest ... >
	<application ... >
		<activity android:name=".yourActivityName"> 
			<intent-filter> 
				<action android:name="android.intent.action.MAIN"/> 
				category android:name="android.intent.category.LAUNCHER"/>
			</intent-filter> 
		</activity>
	</application ... >
    </manifest >
```

### Start another activity
#### Without any data ####

    Intent intent = new Intent(this, NextActivity.class);
    startActivity(intent);

#### With data  ####
    
    Intent intent = new Intent(this, NextActivity.class);
    intent.putExtra("key","value");
    startActivity(intent);
    
    // Get the data in NextActivity :
    String message = intent.getStringExtra(FirstActivity.EXTRA_MESSAGE);
    
You also need to declare a constant that will be the key of the extra. You can pass boolean, String and even Object (that has to be either Serializable or Parcellable). However, in my opinion, it is better to not pass object and keep them in Services that you will inject in both of the activities.

#### Start activity for result ####

First activity

    Intent i = new Intent(this, SecondActivity.class); 
    startActivityForResult(i, FirstActivity.REQUEST_CODE);
    
    // When we come back from SecondActivity
    @Override protected void onActivityResult(int requestCode, int resultCode, Intent data) {
	if (requestCode == FirstActivity.REQUEST_CODE) { 
		if(resultCode == FirstActivity.RESULT_OK) { 
			String result=data.getStringExtra("result");
		} 
		if (resultCode == FirstActivity.RESULT_CANCELED) { 
		} 
	} 
    }

Second activity

    // result ok
    Intent returnIntent = new Intent(); 
    returnIntent.putExtra("result", result); 
    setResult(Activity.RESULT_OK, returnIntent); 
    finish();
    
    // result canceled
    Intent returnIntent = new Intent(); 
    setResult(Activity.RESULT_CANCELED, returnIntent);
    finish();

## Layout and views ##
### The different types of layout and views ###

Nice cheatsheets [here](http://labs.udacity.com/images/Layout-Cheat-Sheet.pdf) and [here](http://labs.udacity.com/images/Common-Android-Views-Cheat-Sheet.pdf).

### Create a custom view ###

1 - Extend the view that you want to add features. It can be extending LinearLayout, a TextView, a simple View.

2 - Implement the constructor matching super with the context :

    // without any custom attributes
    public MyView(Context context) {
        this(context);
    }

If you want to add attributes

3 - Declare attributes in attrs.xml

```xml
    <declare-styleable name="MyView">
       <attr name="attributeName" format="boolean" />
    </declare-styleable>
```

There are several format available

3 - Implement the function with attrs and get attributes to do what you want

    // with default attributes
    public MyView(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, com.android.internal.R.attr.textViewStyle);

        TypedArray a = context.getTheme().obtainStyledAttributes(
                attrs,
                R.styleable.MyView,
                0, 0);

           try {
               mShowText = a.getBoolean(R.styleable.MyView_attributeName, false);
           } finally {
               a.recycle();
           }
    }

More information [here](https://developer.android.com/training/custom-views/create-view.html#customattr)
