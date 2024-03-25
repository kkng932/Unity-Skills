# 터치 감지  
UI.Image를 사용하고, 터치 했을 때 해당 이미지를 터치했는지 감지할 때 많이 사용했던 코드이다.  
FSM과 함께 사용해 터치하는 순간 상태변화시켜 한번만 실행하도록 사용하였다.  
계층뷰와 상관없이 UI 감지 가능하다. 


```C#
 public void Update()
        {
            if (Input.GetMouseButtonDown(0))
            {
                PointerEventData pointerData = new PointerEventData(eventSystem);
                List<RaycastResult> rayCastResult = new List<RaycastResult>();
                pointerData.position = Input.mousePosition;
                graphicRaycaster.Raycast(pointerData, rayCastResult);

                // 상속 여부로 찾기 
                ParentClass tempClick = rayCastResult.Find(
                    x => x.gameObject.GetComponentInParent<ParentClass>() != null).gameObject?.GetComponentInParent<Parent>();

                if (tempClick != null)
                {
                   // 터치 처리 
                }
            }
        }

```
