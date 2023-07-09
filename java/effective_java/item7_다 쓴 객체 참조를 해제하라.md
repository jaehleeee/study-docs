# 7. 다 쓴 객체 참조를 해제하라

## 핵심 정리
#### 레퍼런스가 없어야 가비지 컬렉션의 대상이 된다.
#### 자기 메모리를 직접 관리하는 클래스라면 메모리 누수를 조심해야 한다. (스택, 캐시, 리스너 또는 콜백)
 * 캐시를 만들때 WeakHashMap을 사용하면 좀 더 메모리를 잘 관리할 수 있다.
    * WeakHashMap 에 weak cache만 남아있다면? gc에서 weak cache를 정리한다.
 * 캐시 개수 제한을 두는 방법도 좋다. (LRU 정책 등 활용을 통해)
#### 참조 객체를 null 처리하는 일은 예외적인 경우인데, 유효범위가 끝나면 null 처리해주는 것이 좋다.
 * 유효범위의 예시 : 특정 메서드내에서만 메서드 과정에서 사용하는 객체인 경우, 메서드 끝날때 null 처리해주는게 좋다.

## 완벽 공략
#### NullPointerException
 * NPE 발생 이유
    * 메서드에서 null 리턴하기 때문
    * null 체크를 하지 않기 때문
 * Optional 활용을 통해 NPE를 방지하자
   * Optional은 파라미터로 쓰라고 만든게 아니라, 리턴 타입의 용도이다.
   * Collection을 Optional로 감싸지 마라.
   * null을 Optional 객체에 넣지 마라. Optional.empty() 를 넣어야 한다.

#### WeakHashMap
 * Key가 더 이상 Strong Reference 되는 곳이 없다면, 해당 엔트리를 제거한다.
    * CacheKey라는 클래스를 Map의 key로 둔다면(Map<CacheKey, Value>) CacheKey c1 이라는 키와 Value v2 라는 객체들이 Map 엔트리로 들어가 있다고 가정하자.
    * 이때 c1 = null을 넣으면, WeakHashMap에서는 Map에 저장되어 있던 엔트리도 제거한다.
 * WeakReference의 List는 제대로 동작하지 않으니 그냥 사용하면 안된다 (ex: `List<WeakReference<User>>`) 필요하다면, Custom List 만들어야 한다
 * 레퍼런스 종류
    * Strong : 보통 알고 있는 참조. `new 객체()`를 통해 객체를 매핑하는 경우가 Strong이다.
    * Soft : `new SoftReferecne<>(객체)` 형태로 사용. Strong reference가 없다면 GC 대상이 된다. 다만, 메모리가 부족하는 등 꼭 필요한 경우가 생길때만 GC 처리된다.(즉 웬만하면 안없어진다)
    * Weak : `new WeakReferecne<>(객체)` 형태로 사용. Strong reference가 없다면 GC 대상이 된다. GC 발생시 바로 없어진다.
    * Phantom : `new PhantomReferecne<>(객체, queue)` 형태로 사용(queue 데이터구조 필수). 사라질때 queue에 들어간다.
       * 자원 정리할떄 주로 사용되는 용도이다.
 * 결론적으로, 굳이 Strong 이외의 참조는 사용을 지양하자...
         
#### 백그라운드 쓰레드
 * ScheduledThreadPoolExecutor
