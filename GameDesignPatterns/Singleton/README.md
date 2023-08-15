# 싱글톤 
런타임 동안 메모리에 오직 하나의 인스턴스만 존재한다. 

- 장점  
    - 전역 접근 가능  
    - 동시성 제어
    
- 단점
    - 유닛 테스트가 어려워질 수 있다  

```C#
void Awake()
{
    if(instance == nul)
    {
        instance = this;
        DontDestroyOnLoad(gameObject);
    }
    else
        Destroy(gameObject);
    
}
```
