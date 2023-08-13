### Example publisher class

NOTE: this is just an example to show how events work. For proper input management use the new Input System package.

This example class shows how to separate input management from other concerns.  It handles two types of keyboard events: 
- OnMoveAction - when WASD is pressed
- OnInteractAction - when E is pressed

When these assigned keys are pressed, any subscriber to these events will get notified.  Notice that `OnMoveAction` has a `Action<Vector2>` type, so that we can pass the input vector to the subscribers.  In this case input vector is represented as a Vector2. 

inputVector.x represents horizontal input, i.e. D is +1 (more right) and A is -1 (move left)
inputVector.y represents vertical input, i.e. W is +1 (move up) and S is -1 (move down)

```cs
public class InputManager : MonoBehaviour {

    public event Action<Vector2> OnMoveAction;
    public event EventHandler OnInteractAction;

    private void Update() {
        // Handle interact action. 'E' key press.
        if (Input.GetKeyDown(KeyCode.E)) {
            OnInteractAction?.Invoke(this, EventArgs.Empty);
        }

        // Handle move action. WASD keys
        Vector2 inputVector = Vector2.zero;
        if (Input.GetKey(KeyCode.W)) {
            inputVector.y += 1;
        }
        if (Input.GetKey(KeyCode.S)) {
            inputVector.y -= 1;
        }
        if (Input.GetKey(KeyCode.D)) {
            inputVector.x += 1;
        }
        if (Input.GetKey(KeyCode.A)) {
            inputVector.x -= 1;
        }
        OnMoveAction?.Invoke(inputVector);
    }
}
```

### Example subscriber class

Subscribes to publisher events in Start() using += operator.

```cs
public class PlayerController : MonoBehaviour {

    [SerializeField] InputManager inputManager;

    private void Start() {
        inputManager.OnInteractAction += InputManager_OnInteractAction;
        inputManager.OnMoveAction += InputManager_OnMoveAction;
    }

    private void InputManager_OnMoveAction(Vector2 inputVector) {
        Debug.Log("moving...");
    }

    private void InputManager_OnInteractAction(object sender, System.EventArgs e) {
        Debug.Log("interacting...");
    }
}
```
