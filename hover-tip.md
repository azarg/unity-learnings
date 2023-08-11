Step 1. Create the HoverTip game object in the scene (Canvas)
- Add an empty game object to Canvas, rename "HoverTip"
- Add a UI panel as a child, rename to "Box"
- Add a UI text (TextMeshPro) as a child of Box, rename to "Text"\
![image](https://github.com/azarg/unity-learnings/assets/6077141/485ff725-0b3c-4511-bafa-b475afa7ccb2)

Step 2. Attach this controller script to the HoverTip game object
```cs
using System;
using TMPro;
using UnityEngine;

public class HoverTipController : MonoBehaviour
{
    public static Action<string, Vector3> ShowTip;
    public static Action HideTip;

    [SerializeField] private RectTransform tipBox;
    [SerializeField] private TextMeshProUGUI tipText;
    [SerializeField] float maxWidth = 400f;
    [SerializeField] float maxHeight = 300f;

    private void OnEnable() {
        ShowTip += ShowTipHandler;
        HideTip += HideTipHandler;
    }

    private void OnDisable() {
        ShowTip -= ShowTipHandler;
        HideTip -= HideTipHandler;
    }

    private void ShowTipHandler(string tipText, Vector3 position) {
        this.tipText.text = tipText;
        var preferredDimensions = this.tipText.GetPreferredValues(tipText, maxWidth, float.NegativeInfinity);
        preferredDimensions.x = Mathf.Min(preferredDimensions.x, maxWidth);
        preferredDimensions.y = Mathf.Min(preferredDimensions.y, maxHeight);
        tipBox.sizeDelta = preferredDimensions;
        tipBox.transform.position = position;
        tipBox.gameObject.SetActive(true);
    }
    
    private void HideTipHandler() {
        tipBox.gameObject.SetActive(false);
        tipText.text = default;
    }
}
```

Step 3. Attach this script to any game object or prefab that needs to display a tool tip on mouse hover:
```cs
using System.Collections;
using UnityEngine;
using UnityEngine.EventSystems;

public class HoverTip : MonoBehaviour, IPointerEnterHandler, IPointerExitHandler {

    [SerializeField] private string message;
    [SerializeField] private Vector3 spawn;
    [SerializeField] private float hoverTime = 0.5f;
    
    public void Initialize(string message, Vector3 spawn) {
        this.message = message;
        this.spawn = spawn;
    }

    public void OnPointerEnter(PointerEventData eventData) {
        if (message != null) {
            StopCoroutine(DelayedShowTip());
            StartCoroutine(DelayedShowTip());
        }
    }

    private IEnumerator DelayedShowTip() {
        yield return new WaitForSeconds(hoverTime);
        HoverTipController.ShowTip(message, spawn);
    }

    public void OnPointerExit(PointerEventData eventData) {
        StopCoroutine(DelayedShowTip());
        HoverTipController.HideTip();
    }
}
```
