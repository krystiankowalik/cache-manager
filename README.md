# Cache Implementation

Sample implementation of cache for java applications to improve performance for handling multiple deterministic requests.

## Specification

Application freezes due to recurring requests which yield identical results for the same sets of parameters. The cache to improve the performance in this case should implement the following interfaces:

CacheItem - the base container for individual objects which are retrievable from CacheItem through getValue() method.   

```
public interface CacheItem {
    /**
     * @return the key of the cache item - a unique identifier of the item 
     * within the underlying cache's collection.
     */
    String getKey();

    /**
     * @return the value of the cached item - the underlying Object
     */
    Object getValue();
}

```
CacheView - allows to view the contents of the cache by providing read-only methods:
```

public interface CacheView {

    /**
     * @return the number of non-null elements within cache
     **/
    int size();

    /**
     * @param index integer, sequential number of the element of the cache
     * based on the order of addition
     * @return the cacheItem by index
     **/
    CacheItem getItem(int index);

    /**
     * @param key String unique identifier of the element
     * @return the cacheItem by its key.
     */
    CacheItem getItem(String key);
}

```
Cache - allows to modify the cache by adding new objects or deleting all items from cache. It can also return view of the cache.
```

public interface Cache {

    /**
     * Saves an Object into the Cache under the specified key.
     *
     * @param item any object to be saved into the Cache
     * @param key  a String that the object will be associated with
     * @return returns the saved cacheItem
     */
    CacheItem cacheItem(Object item, String key);

    /**
     * Clears the cache
     */
    void invalidateCache();

    /**
     * @return CacheView
     */
    CacheView getView();
}

```

### Considerations

#### Computational complexity

First of all, the idea behind the cache is to improve the performance of the application. For this reason, it is paramount to ensure that it does not slow it down by itself. The purpose of the cache requires that objects are retrieved/saved with each request. Since the interfaces to be implemented in the project provide that a cache item can be retrieved by both key and index, the natural concern that arises is that of the proper data structure to hold the cache's contents. With a view to ensuring that both of those operations will be performed with an optimal computational complexity, I have selected ListOrderedMap from org.apache.commons.collections4.map to be the container. 

#### Underlying data structure

ListOrderedMap has been implemented as a Map decorated with a List. The underlying Map holds the key-value associations and the List ensures the order of elements' addition is retained. Thus, the elements are searchable by both key and index with O(1) complexity.


For the sake of comparison, I added 3 more sample implementations as branches to this project:
* [CircularFifoQueue](https://github.com/krystiankowalik/cache/tree/CircularFifoQueue_Impl/) - The implementation uses CircularFifoQueue from org.apache.commons.collections4.map. The container seemed handy for the purposes of this project, as it has been implemented as a queue with a predetermined, fixed number of elements which removes the eldest member when adding a new one if the maximum capacity is reached. Unfortunately, the tests have proved that get(String key) method was not optimal. The data structure does  not provide key/value mappings so they were retrieved by iterating through all values, which is likely to result is O(log(n)) computational complexity.
* [LinkedHashMap](https://github.com/krystiankowalik/cache/tree/LinkedHashMap_impl) - This implementation used LinkedHashMap as the underlying cache container. It can also be set to remove the eldest entry on adding new elements when the specified maximum capacity has been reached. Nevertheless, while when dealing with retrieval by key, the LinkedHashMap has proved to be very efficient and it does preserve the order in which the elements are added, it has not been designed to access elements by index, which can observed by running the stress tests. Therefore, it was also rejected.
* [HashMap+ArrayList](https://github.com/krystiankowalik/cache/tree/HashMap_Impl) - This implementation combines the use of HashMap as the primary container of the cache items and the tool for key/value associations with ArrayList as the store of keys in order of addition. Similarly to ListOrderedMap, this data structure combination allows to efficiently retrieve CacheItems by both index and key. However, it has not been chosen as the best structure, as it required a more 'boiler-plate' implementation within the cache source code, reducing the readability of the code. Furthermore, looking at the timings of the unit tests, ListOrderedMap seems to be slightly better optimized. 

#### Concurrency issues

The application which handles multiple requests to/from database or through HTTP is likely to heavily rely on multithreading to keep its flow. For this reason, it is important to provide as thread-safe cache implementation as possible. Ideally, this would be done by using the data structure designed for such operations, e.g. ConcurrentHashMap. Nevertheless, this was not possible in this case, as the map does not store the order of elements.  

#### Clean code adherence

### Unit tests

 
 

