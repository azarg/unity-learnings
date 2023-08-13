Custom inspector button for invoking int variable change:
```cs
using UnityEngine;
using UnityEditor;

[CustomEditor(typeof(IntVariable))]
public class IntVariableCustomInspector : Editor {
    public override void OnInspectorGUI () {
        DrawDefaultInspector();

        var variable = (IntVariable)target;
        if(GUILayout.Button("Trigger change")) {
            variable.SetValue(variable.GetValue());
        }
    }
}
```

IntVariable scriptable object:
```cs
using System;
using UnityEngine;

[CreateAssetMenu]
public class IntVariable : ScriptableObject {
    
    public event Action OnValueChanged;

    [SerializeField] private int value;

    public void SetValue(int value) {
        this.value = value;
        OnValueChanged?.Invoke();
    }

    public int GetValue() => value;

    private void OnEnable() {
        value = 0;
    }
}
```
