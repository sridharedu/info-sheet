SPRING BOOT + REDIS CACHE 💥🧠

---------------------------------------------------
QUICK START 🏁 (Cache backed by Redis)
---------------------------------------------------
pom.xml → need both:
  spring-boot-starter-data-redis
  spring-boot-starter-cache

→ add on main class:
  @EnableCaching

→ app.yml:
spring:
  cache:
    type: redis
  redis:
    host: localhost
    port: 6379
    # timeout: 5000
    # password: xyz (if set)

🧠 Spring auto-configs RedisCacheManager
🧠 Redis stores cache entries as hashes (key=cacheName::key)

---------------------------------------------------
USE IN CODE 🧪
---------------------------------------------------
→ Cache method result:
  @Cacheable(value = "users", key = "#id")
  public User getUserById(String id) {
     ...db call...
  }

→ Update cache:
  @CachePut(value = "users", key = "#id")
  public User updateUser(User u) { ... }

→ Evict from cache:
  @CacheEvict(value = "users", key = "#id")

→ Clear all keys in one cache:
  @CacheEvict(value = "users", allEntries = true)

🧠 works for any return type (POJO, String, List)

---------------------------------------------------
CACHING FLOW →
---------------------------------------------------
Client → API → checks Redis  
    ↳ hit → returns cached  
    ↳ miss → calls method → saves return → puts in Redis  

🧠 next call with same key → skips method call  
🧠 cache hit doesn’t even log in controller if logging @ method entry

---------------------------------------------------
CACHE TTL (VERY IMPORTANT) ⏳
---------------------------------------------------
by default → no expiry 😱 → memory bloats  
→ set TTL manually via config:

@Bean
public RedisCacheManager cacheManager(RedisConnectionFactory cf) {
  RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
     .entryTtl(Duration.ofMinutes(30))
     .disableCachingNullValues();

  return RedisCacheManager.builder(cf)
     .cacheDefaults(config)
     .build();
}

🧠 can define TTL per cache via map config
🧠 disableCachingNullValues() = avoids storing nulls (save memory)

---------------------------------------------------
CACHE KEY LOGIC 🧠
---------------------------------------------------
Default key:
  cacheName::SimpleKey (uses param toString)

→ Customize key:
  @Cacheable(value = "users", key = "'usr::' + #id")
  (adds prefix → helps scan manually)

🧠 key collision = common bug when using wrong param

---------------------------------------------------
REDIS FORMAT (real Redis keys)
---------------------------------------------------
- myCache::1
- users::1234
- products::SimpleKey [if method has no arg]

→ inspect:
  redis-cli
  keys *  (slow, don’t use in prod)

→ get value:
  hgetall "users::1234"

---------------------------------------------------
TROUBLESHOOT 🔧
---------------------------------------------------
☐ Cache not working?
  - forgot @EnableCaching
  - wrong key (check key generation)
  - Redis down?
  - method returns null? → cache may skip (based on config)

☐ Cache stores nulls?
  - default = yes
  - disableCachingNullValues() in config

☐ Multiple caches?
  - define TTL per cache via cacheManager config

☐ App not hitting cache?
  - method is private? (proxy won't work)
  - @Cacheable inside same class calling another method? won’t work
      → solution: move to separate service OR self-inject + call

---------------------------------------------------
DEV TRICKS 💡
---------------------------------------------------
✓ Add log on DB method to check if it’s called or not → verify cache
✓ Use redis-cli to inspect keys
✓ Set TTL always (no TTL = memory leak in long run)
✓ Use version in cache name: "users_v2" → safe invalidation after data model changes
✓ Clear cache on update/delete ops (ALWAYS pair CacheEvict with data-changing methods)
✓ NEVER use Cacheable on write methods (will cause stale writes!)

---------------------------------------------------
USE CASES 🎯
---------------------------------------------------
✔ Read-heavy endpoints (profile, dashboard, config)
✔ External API result caching
✔ Auth/user settings
✔ Static master data
✔ Config flags, pricing tiers
✔ Frequent lookups (mobile UI → hits same user/profile)

---------------------------------------------------
NOT FOR ❌
---------------------------------------------------
✘ rapidly changing data (orders, payments)
✘ write-heavy endpoints
✘ security sensitive stuff (roles, tokens) — cache inconsistency risk
✘ anything that can’t tolerate stale data

