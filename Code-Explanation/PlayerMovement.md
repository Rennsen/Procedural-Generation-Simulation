### `PlayerMovement.cs`
```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using TMPro;

public class PlayerMovement : MonoBehaviour
{
    [Header("Movement")]
    public float moveSpeed;

    public float groundDrag;

    public float jumpForce;
    public float jumpCooldown;
    public float airMultiplier;
    bool readyToJump;
```
- The script defines a class `PlayerMovement` inheriting from `MonoBehaviour`.
- Public variables are declared to control movement parameters such as `moveSpeed`, `groundDrag`, `jumpForce`, `jumpCooldown`, and `airMultiplier`.
- `readyToJump` is a boolean flag to determine if the player is ready to jump again.

```csharp
    [HideInInspector] public float walkSpeed;
    [HideInInspector] public float sprintSpeed;

    [Header("Keybinds")]
    public KeyCode jumpKey = KeyCode.Space;

    [Header("Ground Check")]
    public float playerHeight;
    public LayerMask whatIsGround;
    bool grounded;

    public Transform orientation;

    float horizontalInput;
    float verticalInput;

    Vector3 moveDirection;

    Rigidbody rb;
```
- Additional movement parameters are declared as hidden public variables (`walkSpeed`, `sprintSpeed`) for possible future use.
- `KeyCode` variable `jumpKey` is set to determine the key for jumping.
- Ground check parameters (`playerHeight`, `whatIsGround`) and the player's orientation are defined.
- `horizontalInput` and `verticalInput` store the input from horizontal and vertical axes.
- `moveDirection` represents the direction of player movement.
- Rigidbody component `rb` is declared to handle physics interactions.

```csharp
    private void Start()
    {
        rb = GetComponent<Rigidbody>();
        rb.freezeRotation = true;

        readyToJump = true;
    }
```
- The `Start()` method is called when the script starts.
- Rigidbody component `rb` is assigned to the Rigidbody component attached to the same GameObject.
- Rotation is frozen to prevent unintended rotation physics.
- `readyToJump` flag is set to `true` to allow jumping initially.

```csharp
    private void Update()
    {
        // ground check
        grounded = Physics.Raycast(transform.position, Vector3.down, playerHeight * 0.5f + 0.3f, whatIsGround);

        MyInput();
        SpeedControl();

        // handle drag
        if (grounded)
            rb.drag = groundDrag;
        else
            rb.drag = 0;
    }
```
- The `Update()` method is called every frame.
- Ground check is performed using `Physics.Raycast` to detect if the player is grounded.
- Input processing, speed control, and drag handling are performed.
- If the player is grounded, the ground drag value is applied to the Rigidbody; otherwise, drag is set to 0.

```csharp
    private void FixedUpdate()
    {
        MovePlayer();
    }
```
- The `FixedUpdate()` method is called at fixed time intervals, synced with the physics engine.
- It calls the `MovePlayer()` method to handle player movement.

```csharp
    private void MyInput()
    {
        horizontalInput = Input.GetAxisRaw("Horizontal");
        verticalInput = Input.GetAxisRaw("Vertical");

        // when to jump
        if (Input.GetKey(jumpKey) && readyToJump && grounded)
        {
            readyToJump = false;

            Jump();

            Invoke(nameof(ResetJump), jumpCooldown);
        }
    }
```
- The `MyInput()` method processes player input.
- Horizontal and vertical input values are obtained from the input system.
- If the jump key is pressed and the player is ready to jump and grounded, the player jumps.
- `Invoke()` is used to delay the resetting of the jump state by `jumpCooldown` seconds.

```csharp
    private void MovePlayer()
    {
        // calculate movement direction
        moveDirection = orientation.forward * verticalInput + orientation.right * horizontalInput;

        // on ground
        if (grounded)
            rb.AddForce(moveDirection.normalized * moveSpeed * 10f, ForceMode.Force);

        // in air
        else if (!grounded)
            rb.AddForce(moveDirection.normalized * moveSpeed * 10f * airMultiplier, ForceMode.Force);
    }
```
- The `MovePlayer()` method calculates and applies player movement.
- Movement direction is calculated based on the orientation of the player.
- If the player is grounded, force is applied in the calculated direction with a magnitude of `moveSpeed`.
- If the player is in the air, force is applied with a multiplier of `airMultiplier`.

```csharp
    private void SpeedControl()
    {
        Vector3 flatVel = new Vector3(rb.velocity.x, 0f, rb.velocity.z);

        // limit velocity if needed
        if (flatVel.magnitude > moveSpeed)
        {
            Vector3 limitedVel = flatVel.normalized * moveSpeed;
            rb.velocity = new Vector3(limitedVel.x, rb.velocity.y, limitedVel.z);
        }
    }
```
- The `SpeedControl()` method limits the player's velocity if it exceeds the maximum move speed.
- The horizontal component of the velocity is extracted to create a flat velocity vector.
- If the magnitude of the flat velocity exceeds the move speed, it is normalized and scaled to the move speed.
- The player's velocity is then updated with the limited velocity.

```csharp
    private void Jump()
    {
        // reset y velocity
        rb.velocity = new Vector3(rb.velocity.x, 0f, rb.velocity.z);

        rb.AddForce(transform.up * jumpForce, ForceMode.Impulse);
    }
```
- The `Jump()` method handles the player's jump action.
- The vertical component of the velocity is reset to 0 to prevent adding to the existing velocity.
- An upward force is applied to the player using `ForceMode.Impulse` to initiate the jump.

```csharp
    private void ResetJump()
    {
        readyToJump = true;
    }
}
```
- The `ResetJump()` method resets the `readyToJump` flag to `true` after the jump cooldown period has elapsed.
