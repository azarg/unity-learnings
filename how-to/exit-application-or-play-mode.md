How to exit application or stop play mode.  Normally exiting the application is simply `Application.Quit()`.  However this does not stop the play mode.   To exit the play mode you need to call `EditorApplication.ExitPlaymode()`. To be able to do both at the same time, you'll have to add compiler directives:

```cs
#if UNITY_EDITOR
using UnityEditor;
#endif

public class SomeClass(){
  public void Exit()
  {
#if UNITY_EDITOR
        EditorApplication.ExitPlaymode();
#else
        Application.Quit(); // original code to quit Unity player
#endif
  }
}
```
