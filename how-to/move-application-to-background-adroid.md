This is how to move app to background in Android

```cs
AndroidJavaObject activity = new AndroidJavaClass("com.unity3d.player.UnityPlayer")
                                .GetStatic<AndroidJavaObject>("currentActivity");
activity.Call<bool>("moveTaskToBack", true);
```
