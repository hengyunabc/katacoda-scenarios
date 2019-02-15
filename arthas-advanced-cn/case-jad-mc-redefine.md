下面介绍通过`jad`/`mc`/`redefine` 命令实现动态更新代码的功能。

目前，直接访问 https://[[HOST_SUBDOMAIN]]-80-[[KATACODA_HOST]].environments.katacoda.com/user/0 ，会返回500异常：

```
Something went wrong: 500 Internal Server Error
```

下面通过热更新代码，修改这个逻辑。

### jad反编译UserController

`jad --source-only com.example.demo.arthas.user.UserController > /tmp/UserController.java`{{execute T2}}

jad反编译的结果保存在 `/tmp/UserController.java`文件里了。

再打开一个`Terminal 3`，然后用vim来编辑`/tmp/UserController.java`：

`vim /tmp/UserController.java`{{execute T3}}

比如当 user id 小于1时，也正常返回，不抛出异常：

```java
    @GetMapping(value={"/user/{id}"})
    public User findUserById(@PathVariable Integer id) {
        logger.info("id: {}", (Object)id);
        if (id != null && id < 1) {
			return new User(id, "name" + id);
            // throw new IllegalArgumentException("id < 1");
        }
        return new User(id.intValue(), "name" + id);
    }
```

### sc查找加载UserController的ClassLoader

```bash
$ sc -d *UserController
 class-info        com.example.demo.arthas.user.UserController
 code-source       file:/Users/hengyunabc/code/java/spring-boot-inside/demo-arthas-spring-boot/target/demo-arthas-spring-boot-0.0.1-SNAPSHOT.jar!/BOOT-INF/classes!/
 name              com.example.demo.arthas.user.UserController
 isInterface       false
 isAnnotation      false
 isEnum            false
 isAnonymousClass  false
 isArray           false
 isLocalClass      false
 isMemberClass     false
 isPrimitive       false
 isSynthetic       false
 simple-name       UserController
 modifier          public
 annotation        org.springframework.web.bind.annotation.RestController
 interfaces
 super-class       +-java.lang.Object
 class-loader      +-org.springframework.boot.loader.LaunchedURLClassLoader@6b884d57
                     +-sun.misc.Launcher$AppClassLoader@3d4eac69
                       +-sun.misc.Launcher$ExtClassLoader@7494e528
 classLoaderHash   6b884d57
```

可以发现是 spring boot `LaunchedURLClassLoader@6b884d57` 加载的。

### mc

保存好`/tmp/UserController.java`之后，使用`mc`(Memory Compiler)命令来编译，并且通过`-c`参数指定ClassLoader：

`mc -c 6b884d57 /tmp/UserController.java -d /tmp`{{execute T2}}

```bash
$ mc /tmp/UserController.java -d /tmp
Memory compiler output:
/tmp/com/example/demo/arthas/user/UserController.class
Affect(row-cnt:1) cost in 346 ms
```

### redefine

再使用`redefine`命令重新加载新编译好的`UserController.class`：

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
