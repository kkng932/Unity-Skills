# 전역 정적 변수  
모든 씬에서 사용 가능한 변수  
구현이 쉽고 json으로 변환 후 저장, 관리가 쉬워서 사용 중이다. 

```C#
public class GameData
{
    static BoolData m_DLCUnlock;
    public static BoolData DLCUnlock
    {
        get
        {
            if (m_DLCUnlock == null)
                m_DLCUnlock = Resources.Load<BoolData>("DLCContents/Data/DLCUnlock");
            return m_DLCUnlock;
        }
    }
}
```
