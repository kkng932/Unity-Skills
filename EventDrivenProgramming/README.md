# 이벤트 주도적 프로그래밍(Event Driven Programming)  
이벤트가 발생했을 때 성능, 코드 가독성을 높히고 싶어서 사용하였다.  

```C#
// 참고: [유니티 C# 스크립팅 마스터하기] 
// 이벤트 관리를 위한 코드입니다.
// 이벤트 주도적 프로그래밍


// EVENT_TYPE은 테이블 순서와 같아야 합니다. 

using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public enum EVENT_TYPE
{
    // 사용할 이벤트 나열 
    // 손님냥이 해금 조건
    /// <summary>
    /// 가게 운영 {0}일  
    /// </summary>
    eDay = 0,
    /// <summary>
    /// {0} 고양이 호감도 {1} 달성 
    /// </summary>
    eCatExp,
    /// <summary>
    /// {0} 요리 재료 해금 
    /// </summary>
    eIngUnlock,
    /// <summary>
    /// 아이템 {0} 또는 {1} 구매 
    /// </summary>
    eBuyItem,
    /// <summary>
    /// 아이템 {0}, {1}, {2} 모두 구매 
    /// </summary>
    eBuyItemAll,
    /// <summary>
    /// 아이템 {0} 세트 구매 
    /// </summary>
    eBuyItemSet,
    /// <summary>
    /// 소지금 {0} 달성 
    /// </summary>
    eMoney,
    /// <summary>
    /// 누적 플레이타임 {0}분 달성 
    /// </summary>
    ePlayTime,
    //==================================================
    // 재료 해금 조건
    /// <summary>
    /// 특정 랭크 달성 , 영업일 n 
    /// </summary>
    eRatingAndDay,
    /// <summary>
    /// 특정 랭크 달, 특정 재료 해금 
    /// </summary>
    eRatingAndIng,
    /// <summary>
    /// 특정 랭크 달성 , 특정 재료 해금 
    /// </summary>
    eRatingAndCatLv,
    /// <summary>
    /// 특정 랭크 달성 , 상점냥이 아이템 n번 구매 
    /// </summary>
    eRatingAndBuy,
    //==================================================
    // 알바 해금 조건 
    eStaffEmploy,

    //==================================================
    // 필요해서 만듦
    /// <summary>
    /// 레벨 달성 
    /// </summary>
    eLevel 
};
public interface IListener
{
    void OnEvent(EVENT_TYPE EventType, Component Sender, object Param = null);
}

public class MyCustomeListener : MonoBehaviour, IListener
{
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

        // 이벤트 형식 키가 존재 검사
        if (Listeners.TryGetValue(eventType, out ListenList))
        {
            ListenList.Add(Listener);
            return;
        }

        // 없으면 새로운 리스트 생성
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

> 참고: [유니티 C# 스크립팅 마스터하기]
