# 터치 감지  
UI.Image를 사용하고, 터치 했을 때 해당 이미지를 터치했는지 감지할 때 많이 사용했던 코드이다.  

```C#
 public void Update()
        {
            if (Input.GetMouseButtonDown(0))
            {
                PointerEventData pointerData = new PointerEventData(GameManager.Instance.eventSystem);
                List<RaycastResult> rayCastResult = new List<RaycastResult>();
                pointerData.position = Input.mousePosition;
                GameManager.Instance.mainGameRaycaster.Raycast(pointerData, rayCastResult);
 
                GameObject tempClick = rayCastResult.Find(
                    x => x.gameObject.name.Equals("찾고자하는 이름"))?.gameObject;
                if (tempClick != null)
                {
                   // 터치 처리 
                }
            }
        }

```
