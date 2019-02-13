下面介绍通过`jad`/`mc`/`redefine` 命令实现动态更新代码的功能。

目前，直接访问 https://[[HOST_SUBDOMAIN]]-80-[[KATACODA_HOST]].environments.katacoda.com/user/0 ，会返回500异常：

```
Something went wrong: 500 Internal Server Error
```

下面通过热更新代码，修改这个逻辑。

### jad

`jad --source-only com.example.demo.arthas.user.UserController > /tmp/UserController.java`{{execute T2}}

jad反编绎的结果保存在 `/tmp/UserController.java`文件里了。

再打开一个`Terminal 3`，然后用vim来编辑`/tmp/UserController.java`：

`vim /tmp/UserController.java`{{execute T2}}

比如当 user id 小于1时，也正常返回，不抛出异常：

```java
	@GetMapping("/user/{id}")
	public User findUserById(@PathVariable Integer id) {
		logger.info("id: {}" , id);

		if (id != null && id < 1) {
			return new User(id, "name" + id);
			// throw new IllegalArgumentException("id < 1");
		} else {
			return new User(id, "name" + id);
		}
	}
```

### mc

保存好`/tmp/UserController.java`之后，使用`mc`(Memory Compiler)命令来编绎：

`mc /tmp/UserController.java -d /tmp`{{execute T2}}

```
$ mc /tmp/UserController.java -d /tmp
Memory compiler output:
/tmp/com/example/demo/arthas/user/UserController.class
Affect(row-cnt:1) cost in 346 ms
```

### redefine

再使用`redefine`命令重新加载新编绎好的`UserController.class`：

`redefine /tmp/com/example/demo/arthas/user/UserController.class`{{execute T2}}

```
$ redefine /tmp/com/example/demo/arthas/user/UserController.class
redefine success, size: 1
```

### 热修改代码结果

`redefine`成功之后，再次访问 https://[[HOST_SUBDOMAIN]]-80-[[KATACODA_HOST]].environments.katacoda.com/user/0 ，结果是：

```
{
  "id": 0,
  "name": "name0"
}
```
