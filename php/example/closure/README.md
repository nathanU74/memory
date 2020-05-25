### php closure example

````php
# function
public function getViaCache($key, callable $callback = null, $useCache = true) {
  $cacheData = (! $useCache) ? null : $this->memcached()->get($key);

  if (is_null($callback) || ! is_callable($callback)) {
    $newCacheData = $cacheData;
  } else {
    if (isset($cacheData)) {
      $newCacheData = $cacheData;
    } else {
      $newCacheData = call_user_func($callback);
      if ($useCache) {
        $this->memcached()->set($key, $newCacheData, $ttl);
      }
    }
  }

  return $newCacheData;
}




# Callable to wrap.
public function getUser($cid) {
	$key = KeyVariable::get('USER_INFO_INDEX', $cid);
  $info = self::get($key);
  if (! isset($info)) {
    $info = self::loader()->getViaCache($key, function () use ($cid) {
      return self::dbManager()->userdb()->getUser($cid);
    });
    self::set($key, $info);
  }
  return $info;
}




# Model
public function getUser($cid) {
  $sql = 'SELECT characterID, characterName FROM tb_user WHERE characterID = ?';
  return self::loader()->db()->execute($sql, $cid)->fetch_assoc();
}
````

