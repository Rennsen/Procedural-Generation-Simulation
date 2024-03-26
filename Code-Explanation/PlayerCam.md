### `PlayerCam.cs`
```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlayerCam : MonoBehaviour
{
    public float sensX;
    public float sensY;

    public Transform orientation;

    float xRotation;
    float yRotation;
```
- The script defines a class `PlayerCam` inheriting from `MonoBehaviour`.
- Public variables `sensX` and `sensY` control the sensitivity of mouse movement along the X and Y axes, respectively.
- A public variable `orientation` of type `Transform` is used to reference the orientation (usually the player's body).
- Two private variables `xRotation` and `yRotation` store the current rotation angles around the X and Y axes, respectively.

```csharp
    private void Start(){
        Cursor.lockState = CursorLockMode.Locked;
        Cursor.visible = false;
    }
```
- The `Start()` method is called when the script starts.
- It locks the cursor to the center of the screen and hides it.

```csharp
    private void Update(){
        //get mouse input
        float mouseX = Input.GetAxisRaw("Mouse X") * Time.deltaTime * sensX;
        float mouseY = Input.GetAxisRaw("Mouse Y") * Time.deltaTime * sensY;

        yRotation += mouseX;

        xRotation -= mouseY;
        xRotation = Mathf.Clamp(xRotation, -90f, 90f);

        //Rotate cam and orientation
        transform.rotation = Quaternion.Euler(xRotation, yRotation, 0);
        orientation.rotation = Quaternion.Euler(0, yRotation, 0);
    }
}
```
- The `Update()` method is called every frame.
- It retrieves mouse input along the X and Y axes, scaled by the sensitivity values and delta time.
- `yRotation` is updated based on mouse movement along the X axis, causing horizontal rotation.
- `xRotation` is updated based on mouse movement along the Y axis, clamped to prevent excessive vertical rotation.
- The camera's rotation is set using `Quaternion.Euler()` with the calculated rotation angles.
- The orientation's rotation (usually the player's body) is set to match the horizontal rotation, ensuring the player's body turns along with the camera.

This script controls the player camera's movement based on mouse input, allowing for both horizontal and vertical rotation, and updates the orientation (usually the player's body) accordingly.
