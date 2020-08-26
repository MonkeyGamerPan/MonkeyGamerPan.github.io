---
layout: post
title: "Unity Web Address"
featured-img: shane-rounce-205187
categories: [unity, address]
---
## Unity官方案例控制角色移动

人物的旋转控制在Update方法中，人物的移动跳跃则是在FixedUpdate中。

```c#
void Update()
    {
        //Debug.Log(Cursor.lockState);
        float rotationAmount;
        //通过鼠标或者键盘控制人物的转向
        //只要鼠标被按下,鼠标状态就会变为CursorLockMode.Locked状态
        if (Input.GetMouseButton(1) && (!requireLock || controlLock || Cursor.lockState ==
            CursorLockMode.Locked))
        {
            if (controlLock)
                //表示在Game窗口锁定并隐藏鼠标
                Cursor.lockState = CursorLockMode.Locked;//或者Cursor.visible = false;
            rotationAmount = Input.GetAxis("Mouse X") * mouseTurnSpeed * Time.deltaTime;
        }
        else
        {
            //键盘控制 单纯转向 
            if (controlLock)
                Cursor.lockState = CursorLockMode.None;//或者Cursor.visible = true;
            rotationAmount = Input.GetAxis("Horizontal") * turnSpeed * Time.deltaTime;
        }
        rigi.transform.RotateAround(rigi.transform.up, rotationAmount);
 
    }
```

<br/>

```c#
 float SidestepAxisInput
    {
        //如果按住鼠标右键，水平轴也会变成侧移处理
        get
        {
            if (Input.GetMouseButton(1))
            {
                float sideStep = Input.GetAxis("Sidestep");
                float hor = Input.GetAxis("Horizontal");
                return Mathf.Abs(sideStep) > Mathf.Abs(hor) ? sideStep : hor;
            }
            else
            {
                return Input.GetAxis("Sidestep");
            }
        }
    }

```

<br/>

```c#
void FixedUpdate()
    {
        //起点在player对象下方groundedCheckOffset处,方向为向下,射线长度为groundedDistance,
        //检测层为groundLayers指定的层
        grounded = Physics.Raycast(rigi.transform.position + rigi.transform.up * -groundedCheckOffset,
            transform.up * -1, groundedDistance, groundLayers);
        if (grounded)
        {
            rigi.drag = groundDrag;
 
            //在地上,可进行跳跃动作
            if (Input.GetButton("Jump"))
            {
                //为刚体施加一个向上的力:该力是由向上的跳跃速度和在刚体原速度方向上的一个速度合成的
                //VelocityChange是一个瞬时速度
                rigi.AddForce(transform.up * jumpSpeed + rigi.velocity.normalized * directionalJumpFactor
                    , ForceMode.VelocityChange);
 
                if (onMyJump != null)
                    onMyJump();
            }
            else
            {
                //在地上且未跳跃,计算行走的方向
                //合成水平和竖直方向的矢量
                Vector3 movement = Input.GetAxis("Vertical") * rigi.transform.forward +
                    SidestepAxisInput * rigi.transform.right;
 
                //速度的限制
                float appliedSpeed = walking ? speed / walkSpeedDownscale : speed;
 
                if (Input.GetAxis("Vertical") < 0.0f)
                {
                    appliedSpeed /= walkSpeedDownscale;
                }
 
                if (movement.magnitude > 0.01f)
                {
                    //在move的方向上添加一个瞬时速度
                    rigi.AddForce(movement.normalized * appliedSpeed, ForceMode.VelocityChange);
                }
                else
                {
                    //停止运动,且仅在y轴能运动
                    rigi.velocity = new Vector3(0, rigi.velocity.y, 0);
                    return;
                }
            }
        }
        else
        {
            //在空中,阻力为0
            rigi.drag = 0;
        }
 
    }
```















