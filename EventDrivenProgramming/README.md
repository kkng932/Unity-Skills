# 이벤트 주도적 프로그래밍(Event Driven Programming)  
> 참고: [유니티 C# 스크립팅 마스터하기]  

이벤트가 발생했을 때 성능, 코드 가독성을 높히고 싶어서 사용하였다.  
어떤 조건을 달성했을 때 업적이 깨질 때 적용해서 사용했는데, 조건 테이블을 그대로 사용하기 편했고 수정이 용이했으며 코드 가독성이 높았다.  


```C#

using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public enum EVENT_TYPE
{
    // 사용할 이벤트 나열 

    /// <summary>
    /// 가게 운영 {0}일  
    /// </summary>
    eDay = 0,
    /// <summary>
    /// 아이템 구매
    /// </summary>
    eBuyItem,
    /// <summary>
    /// 소지금 {0} 달성 
    /// </summary>
    eMoney,

    // 등등...

    /// <summary>
    /// 레벨 달성 
    /// </summary>
    eLevel 
};

public interface IListener
{
    // 이벤트가 발생할 때, 리스너에서 호출할 함수  
    void OnEvent(EVENT_TYPE EventType, Component Sender, object Param = null);
}

// IListener 인터페이스를 상속하여 만든 리스너  
public class MyCustomeListener : MonoBehaviour, IListener
{
    // 이벤트 수신을 위해 함수 구현  
    public void OnEvent(EVENT_TYPE EventType, Component Sender, object Param = null)
    {

    }
}
public class EventManager : MonoBehaviour
{
    public static EventManager Instance { get { return _instance; } }
    private static EventManager _instance = null;
    // 이벤트를 수신받길 원하는 리스너들을 저장
    // 이벤트를 수신받고싶은 리스너를 리스트로 관리 
    private Dictionary<EVENT_TYPE, List<IListener>> Listeners = new Dictionary<EVENT_TYPE, List<IListener>>();

    private void Awake()
    {
        if (_instance == null)
        {
            _instance = this;
            DontDestroyOnLoad(gameObject);
            return;
        }
        DestroyImmediate(gameObject);
    }

    // 이벤트 발생 알림을 받기 위한 등록
    public void AddListener(EVENT_TYPE eventType, IListener Listener)
    {
        List<IListener> ListenList = null;

        if (Listeners.TryGetValue(eventType, out ListenList))
        {
            ListenList.Add(Listener);
            return;
        }

        ListenList = new List<IListener>();
        ListenList.Add(Listener);
        Listeners.Add(eventType, ListenList);
    }

    // 이벤트 매니저에게 소식을 알려줌 
    public void PostNotification(EVENT_TYPE eventType, Component Sender, object param = null)
    {
        List<IListener> ListenList = null;
        if (!Listeners.TryGetValue(eventType, out ListenList))
            return;

        for (int i = 0; i < ListenList.Count; i++)
        {
            ListenList?[i].OnEvent(eventType, Sender, param);
        }
    }

    // 더 이상 사용하지 않는 이벤트 지우기 
    public void RemoveEvent(EVENT_TYPE eventType) => Listeners.Remove(eventType);

    // 씬이 바뀌어서 이미 파괴된 오브젝트를 참조하지 못하게 수정 
    public void RemoveRedundancies()
    {
        Dictionary<EVENT_TYPE, List<IListener>> newListeners = new Dictionary<EVENT_TYPE, List<IListener>>();
        foreach (KeyValuePair<EVENT_TYPE, List<IListener>> Item in Listeners)
        {
            for (int i = Item.Value.Count - 1; i >= 0; i--)
            {
                if (Item.Value[i].Equals(null))
                    Item.Value.RemoveAt(i);
            }
            if (Item.Value.Count > 0)
                newListeners.Add(Item.Key, Item.Value);
        }
        Listeners = newListeners;
    }

    // 씬이 바뀌었을 때 호출되는 함수 
    private void OnLevelWasLoaded()
    {
        RemoveRedundancies();
    }
}
```

이벤트 발생 위치에서 이벤트 매니저에게 알리면 된다.  
```C#
EventManager.Instance.PostNotification(EVENT_TYPE.eLevel, this, level);
```
이벤트 발생시에 바뀔 내용 작성   
```C#
public class TempClass : MonoBehaviour, IListener
{
    void Start()
    {
        // 미리 등록  
        EventManager.Instance.AddListener(EVENT_TYPE.eLevel, this);
    }

    // 이벤트 발생 시에 호출할 로직
    public void OnEvent(EVENT_TYPE EventType, Component Sender, object Param = null)
    {
        switch (EventType)
        {
            case EVENT_TYPE.eLevel:
                // 해당 이벤트 발생 시에 변경될 내용 작성 
                break;
        }
    }
}
```










