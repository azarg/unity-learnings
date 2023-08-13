### Awake()
Awake() is not called on inactive game objects until they are set active, i.e. by `SetActive(true)`  
HOWEVER, even when the game object is inactive on scene load, any attached script classes are instantiated.   

