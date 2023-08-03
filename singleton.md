The singleton:
```cs
using UnityEngine;

public class MainManager : MonoBehaviour {
    public static MainManager Instance;
    public float PlayerHP;
    private void Awake() {
        if (Instance != null) {
            Destroy(gameObject);
            return;
        }
        Instance = this;
        DontDestroyOnLoad(gameObject);
    }

}
```

Usage:
```cs
public class SomeClass : MonoBehaviour {
    public void SomeFunction(float hp) {
        MainManager.Instance.PlayerHP = hp;
    }
}
```
