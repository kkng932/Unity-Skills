# 전역 정적 변수  
모든 씬에서 사용 가능한 변수  
구현이 쉽고 json으로 변환 후 저장, 관리가 쉬워서 사용 중이다.  
아래 데이터는 ScriptableObject로 저장이 된 상태이다.  

```C#
public class GameData
{
    static BoolData tempData;
    public static BoolData TempData
    {
        get
        {
            if (tempData == null)
                tempData = Resources.Load<BoolData>("DLCContents/Data/tempData");
            return tempData;
        }
    }
}
```
아래는 bool형 데이터 저장용 ScriptableObject 클래스이다.  
```C#
using UnityEngine;
using System.Collections;

[CreateAssetMenu(menuName ="Data/BoolData")]
public class BoolData : ScriptableObject
{
    public bool m_Value;
    public float m_Dirty;
}
```
