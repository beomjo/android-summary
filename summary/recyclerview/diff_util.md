# DiffUtil  

## DiffUtil이란?
DiffUtil은 서로 다른 아이템인지를 체크하여 달라진 아이템만 갱신을 도와주는 Util로 RecyclerView의 성능 향상을 위해 사용한다. 
Eugene W. Myers의 차분 알고리즘을 사용하여 한 목록을 다른 목록으로 변환하기 위한 최소 업데이트 수를 계산한다고 한다.  


## `DiffUtil.ItemCallback`
`ListAdapter`의 `DiffUtil.ItemCallback`은 `areItemsTheSame`과 `areContentsTheSame` 두 메소드만 상속받아 구현하여 간단하게 사용할 수 있으므로 `DiffUtil.ItemCallback`만 알아보도록 한다.  

```java
 public abstract static class ItemCallback<T> {
      
        public abstract boolean areItemsTheSame(@NonNull T oldItem, @NonNull T newItem);

       
        public abstract boolean areContentsTheSame(@NonNull T oldItem, @NonNull T newItem);
    }
```

- `areItemsTheSame` : 현재 리스트에 노출하고 있는 아이템과 새로운 아이템이 서로 같은지 비교를 위해 고유한 ID 값등을 체크
- `areContentsTheSame` : `areItemsTheSame`이 false일 경우 호출되며, 현재 리스트에 노출하고 있는 아이템과 새로운 아이템의 equals를 비교

equals를 비교할 때 Kotlin data class를 꼭 활용하도록 하자.