# Android cheatsheet #

This cheatsheet provides all basic things that every Android developer should know but forgot sometimes. Do not hesitate to make pull request if you think something important is missing or if there is a mistake.

## Table of Contents
  - [Activity](#activity)
    - [Activity lifecycle](#activity-lifecycle)
        - [Methods](#methods)
        - [Some common situations](#some-common-situations)
    - [Define activity as launch activity](#define-activity-as-launch-activity)
    - [Start another activity](#start-another-activity)
        - [Without any data](#without-any-data)
        - [Without data](#without-data)
        - [Start activity for result](#start-activity-for-result)

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

The full diagram can be find [here](https://developer.android.com/images/activity_lifecycle.png)

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

