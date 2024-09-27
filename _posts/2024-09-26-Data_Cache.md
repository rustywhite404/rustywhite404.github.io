---
title: 캐싱으로 조회 성능 개선하기 1  
date: 2024-09-26 17:15:08
categories: etc    
published: true 
tags:
- tip    
---

캐싱(Caching)은 자주 사용되는 데이터에 더 빠르게 접근할 수 있도록 데이터를 어딘가에 임시 저장해두는 방법이다. 
예를 들어 웹사이트를 방문할 때, 매번 서버에서 같은 이미지를 받아오는 대신 한 번 받은 이미지를 로컬 저장소에 저장해 두고 다음에 그 이미지를 다시 사용할 때는 저장된 데이터를 가져오는 방식이다. 이렇게 하면 서버에 다시 요청하는 시간을 절약할 수 있다. 

**어떤 경우에 캐싱을 사용하면 좋을까?**

1. 자주 사용되는 데이터: 
자주 요청되지만 자주 바뀌지 않는 데이터는 캐시에 저장해두면 매번 서버로부터 다시 받아올 필요가 없다.  
ex) 웹사이트의 이미지, 정적 파일 (CSS, JS 파일)
2. 서버 부하를 줄이고 싶을 때: 
서버에 동일한 요청이 계속 들어오는 상황에서는 캐시를 사용하면 서버의 부담을 줄일 수 있다.  
ex) 인기 있는 뉴스 기사 목록, 자주 조회되는 API 응답
3. 성능을 향상시키고 싶을 때
속도가 중요한 경우, 예를 들어 실시간 애플리케이션이나 빠른 응답이 필요한 경우 캐시가 큰 도움이 된다.   
ex) 쇼핑몰에서 상품 목록을 빠르게 보여줘야 할 때

단, **너무 자주 변하는 데이터나 항상 최신 상태가 필요한 데이터**에는 캐시를 적용하면 문제가 생길 수 있으니 적절히 사용해야 한다. 

> 성능 향상을 위한 캐싱을 공부하려고 참고한 강의와 블로그에서는 대형 쇼핑몰의 카테고리 목록, 실시간 제공이 필요하지 않은 대시보드 데이터, 지하철 노선도, 조직도 등의 데이터에 캐싱을 적용했다.  

나는 주로 1번 케이스에서 캐싱 처리를 많이 해 보았는데, 2번이나 3번 케이스에서의 캐싱 처리와 전후 비교법을 배우고 싶었다. 방법을 찾아보며 공부한 내용을 정리해 둔다. 


### 1. Ehcache 사용을 위한 설정     

1. (Maven 프로젝트 기준) pom.xml에 아래 의존성을 추가한다.
![이미지](https://i.imgur.com/0zOPWur.png)  

```  
<!-- cache -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-cache</artifactId>
        </dependency>
        <dependency>
            <groupId>net.sf.ehcache</groupId>
            <artifactId>ehcache</artifactId>
        </dependency>

```   
  
2. ehcache.xml을 추가해 캐시 환경설정을 해준다. 
![이미지](https://i.imgur.com/0zOPWur.png)  
```
<?xml version="1.0" encoding="UTF-8"?>
<ehcache>

    <defaultCache
            maxElementsInMemory="1000"
            maxElementsOnDisk="0"
            eternal="false"
            statistics="false"
            timeToIdleSeconds="10"
            timeToLiveSeconds="10"
            overflowToDisk="false"
            diskPersistent="false"
            memoryStoreEvictionPolicy="LRU"/>

    <cache
            name="NoticeReadMapper.findAll"
            maxElementsInMemory="10000"
            maxElementsOnDisk="0"
            eternal="false"
            statistics="false"
            timeToIdleSeconds="10"
            timeToLiveSeconds="10"
            overflowToDisk="false"
            diskPersistent="false"
            memoryStoreEvictionPolicy="LRU"/>

    <cache
            name="NoticeReadMapper.findByPage"
            maxElementsInMemory="10000"
            maxElementsOnDisk="0"
            eternal="false"
            statistics="false"
            timeToIdleSeconds="10"
            timeToLiveSeconds="10"
            overflowToDisk="false"
            diskPersistent="false"
            memoryStoreEvictionPolicy="LRU"/>

</ehcache>
```  
- name : 캐시의 이름
- maxElementsInMemory : **메모리에 저장**할 수 있는 최대 요소 수
- maxElementsOnDisk : **디스크에 저장**할 수 있는 최대 요소 수 
- eternal : **캐시 항목이 영원히 유지되는지 여부** 
=> 여기서는 false로 설정되어 있으므로 캐시 항목은 유효 기간(timeToLiveSeconds) 또는 유휴 기간(timeToIdleSeconds)이 지나면 제거된다.
- statistics : JMX 통계정보 갱신 옵션
- timeToIdleSeconds : 캐시된 데이터가 사용 되지 않은 채로 유지되는 최대 시간
- timeToLiveSeconds : 설정된 시간 동안 유지 후 갱신(=캐시 된 데이터의 수명)
- overflowToDisk : 메모리에 캐시된 데이터가 메모리 한계를 초과하는 경우 디스크로 넘길지 여부를 지정
- diskPersistent : 디스크에 저장된 데이터가 시스템 재시작 후에도 유지되어야 하는지 여부 
- memoryStoreEvictionPolicy : 메모리가 꽉 찼을 때 데이터 제거 알고리즘 옵션  
ex) **`LRU` , `LFU` , `FIFO`**

3. ehcache Configuration 추가하기  
EhCache를 사용할 수 있도록 **EhCacheManagerFactoryBean**과 **EhCacheCacheManger**를 Bean으로 등록해준다. 
![이미지](https://i.imgur.com/LoVJyVd.png)  

```java 
package com.example.performancecache.config;

import org.springframework.cache.CacheManager;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.cache.ehcache.EhCacheCacheManager;
import org.springframework.cache.ehcache.EhCacheManagerFactoryBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.core.io.ClassPathResource;

@Configuration
@EnableCaching
public class EhcacheConfiguration {

    @Bean
    @Primary
    public CacheManager cacheManager(EhCacheManagerFactoryBean ehCacheManagerFactoryBean) {
        return new EhCacheCacheManager(ehCacheManagerFactoryBean.getObject());
    }

    @Bean
    public EhCacheManagerFactoryBean ehCacheManagerFactoryBean() {
        EhCacheManagerFactoryBean ehCacheManagerFactoryBean = new EhCacheManagerFactoryBean();
        ehCacheManagerFactoryBean.setConfigLocation(new ClassPathResource("ehcache.xml"));
        ehCacheManagerFactoryBean.setShared(true);
        return ehCacheManagerFactoryBean;
    }
}

```  
- **@EnableCaching : 스프링 캐싱을 활성화**
- ehCacheManagerFactoryBean.**setShared** : 여러 스프링 빈이 동일한 **`EhCacheManager`** 인스턴스를 공유할 수 있도록 설정

4. 캐싱 사용해보기  
- @Cacheable 사용하기.
    - **`value`** 속성은 캐시의 이름. 
    - ehcache.xml 구성 파일에서 **`<cache>`** 엘리먼트의 **`name`** 속성과 일치 시켜주기  
    
    ![이미지](https://i.imgur.com/nND02wA.png) 

```java 
@Override
@Cacheable(value = "NoticeReadMapper.findAll")
@Transactional
public List<Notice> getAllNotices() {
    return noticeReadMapper.findAll();
}
``` 

여기까지 하면 사전 설정은 끝났다. 정상적으로 작동하는지 확인하기 위해 미리 작성된 api 예제를 실행시켜 보았다. 


### 2. 정상적으로 작동하는지 테스트 해보기   
![이미지](https://i.imgur.com/vZKf7VJ.png) 

```java 
import net.sf.ehcache.Ehcache;
import net.sf.ehcache.Element;
import org.springframework.cache.CacheManager;
import org.springframework.cache.ehcache.EhCacheCache;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.*;
import java.util.stream.Collectors;

@RestController
@RequestMapping("/api")
public class EhcacheController {
    private CacheManager cacheManager;

    public EhcacheController(CacheManager cacheManager)
    {
        this.cacheManager = cacheManager;
    }

    @GetMapping("/ehcache")
    public Object findAll(){
        List<Map<String, List<String>>> result = cacheManager.getCacheNames().stream()
                .map(cacheName -> {
                    EhCacheCache cache = (EhCacheCache) cacheManager.getCache(cacheName);
                    Ehcache ehcache = cache.getNativeCache();
                    Map<String, List<String>> entry = new HashMap<>();

                    ehcache.getKeys().forEach(key -> {
                        Element element = ehcache.get(key);
                        if (element != null) {
                            entry.computeIfAbsent(cacheName, k -> new ArrayList<>()).add(element.toString());
                        }
                    });

                    return entry;
                })
                .collect(Collectors.toList());

        return result;
    }
}
``` 

위 예제를 간단히 설명하자면 `http://localhost:8080/api/notices`를 호출하면 전체 더미데이터를 보여주고, `http://localhost:8080/api/ehcache`를 호출하면 캐시에 저장된 데이터를 보여주도록 작성되어 있다. 현재 `ehcache.xml`에서 설정된 `timeToLiveSeconds`가 10초이므로, notices를 호출하며 생성된 캐시는 ehcache에서 10초간 확인 할 수 있다. 10초가 지난 후 새로고침을 해보면 캐시에는 아무것도 남아있지 않다.  

![이미지](https://i.imgur.com/8qXKeVo.png)  
![이미지](https://i.imgur.com/LLjMvIV.png)  
- 10초 후 새로고침  
 ![이미지](https://i.imgur.com/FDYTWiN.png)  
 

### 3. 프로세스를 이해해보자  
정상적으로 코드가 작동하는 건 확인했지만, 어떤 흐름으로 작동하는 건지 좀 더 이해를 하고 넘어가고 싶어 코드를 살펴보았다.  
1. Service 레이어의 `@Cacheable(value = "NoticeReadMapper.findAll")` 어노테이션 덕분에, `getAllNotices()` 메소드가 호출되며 데이터베이스에서 가져온 데이터 리스트가 캐시에 캐시에 저장된다. 

2. `/api/ehcache` 엔드포인트는 `EhCacheCache` 객체를 통해 캐시에 저장된 키와 데이터를 조회한다.  

더 찾아보니 지금은 캐시에 저장된 모든 데이터를 조회해서 반환하고 있지만, 각 캐시에 부여되는 `Key`를 통해 특정 캐시만 조회 할 수도 있다고 한다. 아래는 Key를 가지고 원하는 부분만 조회해오는 예제 코드이다.    
```JAVA 
@GetMapping("/ehcache/{cacheName}/{key}")
public Object getSpecificCacheData(@PathVariable String cacheName, @PathVariable String key) {
    EhCacheCache cache = (EhCacheCache) cacheManager.getCache(cacheName);
    
    if (cache != null) {
        Ehcache ehcache = cache.getNativeCache();
        Element element = ehcache.get(key);
        
        if (element != null) {
            return element.getObjectValue(); // 캐시된 실제 데이터를 반환
        } else {
            return "No data found for key: " + key;
        }
    }
    
    return "No cache found with name: " + cacheName;
}

```  
1. `/api/ehcache/{cacheName}/{key}`를 호출할 때, `{cacheName}`은 조회하고자 하는 캐시의 이름, `{key}`는 조회할 데이터의 키로 사용된다.
2. `cacheManager`를 통해 캐시를 가져오고, 그 캐시 안에서 해당 키에 해당하는 데이터를 찾아 반환한다. 만약 해당 키나 캐시 이름이 없으면 적절한 에러 메시지를 반환하도록 처리되어 있다.  
ex) `/api/ehcache/NoticeReadMapper.findAll/123`처럼 호출하면, `NoticeReadMapper.findAll` 캐시에서 키가 `123`인 데이터를 조회할 수 있다.

> ***캐시의 키 이름은 어떻게 지정되는 걸까***  
아직 낯선 내용이다 보니 파생되는 궁금증이 많아 알아보았다. 캐시의 키는 `@Cacheable` 어노테이션을 사용할 때 자동으로 생성되거나, 원하는대로 커스터마이징도 가능하다고 한다. 다음 포스팅에서 key와 condition 옵션을 활용하는 법도 정리해 봐야겠다. 
