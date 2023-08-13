### Awake()
Awake() is not called on inactive game objects until they are set active, i.e. by `SetActive(true)`  
HOWEVER, even when the game object is inactive on scene load, any attached script classes are instantiated.   

In this example `GetImage()` will return null if the attached game object was inactive on scene load
```cs
private Image image;
private void Awake(){
  image = gameObject.GetComponent<Image>();
}
public Image GetImage(){
  return image;
}
```

HOWEVER, in this version, where the image component is assigned to the image variable in the inspector, `GetImage()` WILL return the image component even if the game object was inactive on scene load and even if it is still inactive.
```cs
[SerializeField] private Image image;
public Image GetImage(){
  return image;
}
```
