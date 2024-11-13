using UnityEngine;

public class FixedAspectCameraSize : MonoBehaviour
{
    public Vector2 targetWidthToHeight;
    float targetAspect;
    float originalCameraSize;
    int ScreenWidth = 0;
    int ScreenHeight = 0;
    Camera targetCamera;
    bool rescale;

    void Start() {
        if (targetWidthToHeight.x > 0 && targetWidthToHeight.y > 0) {
            rescale = true;
            targetAspect = targetWidthToHeight.x / targetWidthToHeight.y;
            targetCamera = GetComponent<Camera>();
            originalCameraSize = targetCamera.orthographicSize;
        }
    }

    private void Update() {
        if (rescale) RescaleCamera();
    }

    private void RescaleCamera() {

        if (Screen.height == ScreenHeight && Screen.width == ScreenWidth) return;

        ScreenHeight = Screen.height;
        ScreenWidth = Screen.width;

        float screenAspect = (float)ScreenWidth / (float)ScreenHeight;

        if (screenAspect < targetAspect) {
            // screen is taller than target aspect ratio
            // therefore we need to move the camera away 
            targetCamera.orthographicSize = originalCameraSize * targetAspect / screenAspect;
        }
        else {
            // screen is wider than target aspect ratio
            // there is no need to change the camera size
            // since width changes are handled automatically by the camera
            targetCamera.orthographicSize = originalCameraSize;
        }
    }
}
