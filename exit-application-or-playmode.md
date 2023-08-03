How to exit application or stop play mode.  Normally exiting application is simply `Application.Quit()`.  However this does not stop the play mode. Therefore need to add `EditorApplication.ExitPlaymode()`. However, then we also need to add compiler directives so that it works both in production and during development.

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
