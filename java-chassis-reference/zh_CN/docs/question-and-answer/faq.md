# 常见问题

> 如果在使用Java Chassis的过程中碰到问题或者疑问，可以在 [Java Chassis Github Issues](https://github.com/apache/servicecomb-java-chassis/issues) 提交问题和搜索答案。

* [Q: 契约生成会报错 Caused by: java.lang.Error: OperationId must be unique，不支持函数重载？](#Q1)  
* [Q: Map类型的key必须使用 String 类型？](#Q2)  
* [Q: 参数返回值不能使用接口？](#Q3)
* [Q: 参数返回值不能使用泛型？](#Q4)
* [Q: 实现类中 public 方法全部被发布为接口，如何排除？](#Q5)
* [Q: 如何自定义某个Java方法对应的REST接口里的HTTP Status Code？](#Q6)
* [Q: 使用HTTP Header参数出现乱码如何处理？](#Q7)

<h2 id="Q1">Q: 契约生成会报错 Caused by: java.lang.Error: OperationId must be unique，不支持函数重载？</h2>

* A: 支持函数重载，但是需要注意每个接口必须有唯一的operation id。可以加上`@Operation`标签给重载的接口指定唯一
  的 operation id。示例代码如下：

```java
  @Path("/sayHello")
  @GET
  @Produces("text/plain;charset=UTF-8")
  @Operation(operationId = "sayHello", summary = "say hello without parameter")
  public String sayHello() {
    return "Hello";
  }

  @Path("/sayHelloName")
  @GET
  @Produces("application/json;charset=UTF-8")
  @Operation(operationId = "sayHelloName", summary = "say hello with name parameter")
  public String sayHello(@ApiParam("name") String name) {
    return "Hello " + name;
  }
```

<h2 id="Q2">Q: Map类型的 key 必须使用 String 类型？</h2>

* A: 是的。java-chassis 遵循 [Open API 规范](https://swagger.io/docs/specification/data-models/dictionaries/), MAP
  类型的 key 必须使用 String 类型。 业务可以结合实际情况，使用符合规范的类型。 如果必须使用其他类型，
  可以考虑接口定义使用 Object 规避，客户端可以对返回值结果自行进行 json 转换。


<h2 id="Q3">Q: 参数返回值不能使用接口？</h2>

* A: 是的。java-chassis 不允许参数、返回值的类型为 `Interface` 或者 `Abstract Class`。因为这些类型  无法生成正确的 swagger 描述。如果必须使用这些类型，可以考虑接口定义使用 Object 规避，客户端可以对返回值结果自行进行 json 转换。


<h2 id="Q4">Q: 参数返回值不能使用泛型？</h2>

* A: 可以使用泛型。但是必须明确泛型类型。比如：

```java
  @PostMapping(path = "holderUser")
  public Holder<User> holderUser(@RequestBody Holder<User> input) {
    Assert.isInstanceOf(Holder.class, input);
    Assert.isInstanceOf(User.class, input.value);
    return input;
  }

  @GetMapping(path = "/genericParams")
  @ApiOperation(value = "genericParams", nickname = "genericParams")
  public List<List<String>> genericParams(@RequestParam("code") int code, @RequestBody List<List<String>> names) {
    return names;
  }
```

未指定泛型类型是不允许的。比如：

```java
  @GetMapping(path = "/genericParams")
  @ApiOperation(value = "genericParams", nickname = "genericParams")
  public List genericParams(@RequestParam("code") int code, @RequestBody List names) {
    return names;
  }
```

如果业务必须使用泛型，并且不能确定类型，可以考虑接口定义使用 Object 规避，客户端可以对返回结果自行进行 json 转换。


<h2 id="Q5">Q: 实现类中 public 方法全部被发布为接口，如何排除？</h2>

* A: java chassis 会将所有 public 方法发布为接口。 如果有些接口不需要发布为接口，可以使用 @Operation
  标签声明不发布为接口。例子如下：

```java
  @Operation(summary = "", hidden = true)
  public void hidden() {

  }
```

更加复杂的情况，比如：

```java
@RpcSchema(schemaId = "MyService")
public MyService extends AbstractMyService implements MyInterface {
  
}
```

那么 `AbstractMyService` 的公共方法也会发布为 RPC 接口。 这种情况可以声明为只发布接口里面的方法, 比如：

```java
@RpcSchema(schemaId = "MyService", schemaInterface = MyInterface.class)
public MyService extends AbstractMyService implements MyInterface {

}
```

<h2 id="Q6">Q: 如何自定义某个Java方法对应的REST接口里的HTTP Status Code？</h2>

* A:  参考[多个返回值和错误码](../build-provider/multi-code.md)

<h2 id="Q7">Q: 使用HTTP Header参数出现乱码如何处理？</h2>

* A:  根据HTTP协议，HTTP Header的值只允许ASCII字符。虽然HTTP协议历史上有些改进，但目前的标准场景，仍然建议只使用ASCII字符，否则会出现乱码。 因此，在使用HTTP Header参数的情况，如果使用了非ASCII字符集，需要在请求前对内容进行编码，在收到响应后，需要对内容进行解码。 

```java
// HTTP header只允许 ASCII 字符。
// 在Header参数中使用特殊字符，需要在发送请求前进行编码，在收到响应后进行解码，具体编码方式可以业务自行选择。
// 如果不进行编码，则可能会出现乱码（HTTP协议历史上只标准化了ASCII字符，其他字符集可能工作，可能不工作）。
String encodedResult = controller.sayHei(HttpUtils.encodeURLParam("中文HTTP Header"));
TestMgr.check("hei 中文HTTP Header", HttpUtils.decodeURLParam(encodedResult));
```

> 注意：InvocationContext也会使用HTTP Header进行传输，如果包含特殊字符，也需要在设置前编码，使用前解码。


> 最佳实践：尽可能不在Header参数中使用特殊字符，特殊内容使用Body传输。
