---
layout: post
title: 'RestTemplate实用'
date: 2023-03-23
description: 'RestTemplate实用'
tags: '技术'
--- 

最近在开发中遇到了与外部接口请求的业务，发现大佬的代码中用到了一个类`RestTemplate`，特此：找了一篇优秀的文章来学习、封装使用RestTemplate。
让自己再次遇到类似业务能够使用自己的请求工具快速上手熟悉。

RestTemplate 是由Spring提供的一个HTTP请求工具，它提供了常见的REST请求方案的模板，例如GET请求、POST请求、PUT请求、DELETE请求以及一些通过的请求执行方法 exchange 以及execute。RestTemplate 继承自 InterceptingHttpAccessor 并且实现类 RestOperations 接口，其中 RestOperations 接口定义了基本的 RESTful 操作，这些操作在 RestTemplate 中都得到了实现。

## POST请求
### postForObject

1、使用LinkedMultiValueMap作为参数（Form表单提交）
```java
    @Test
    //使用LinkedMultiValueMap作为参数（Form表单提交）
    public void postForObject1() {
        RestTemplate restTemplate = new RestTemplate();
        String url = "http://127.0.0.1:8998/test/post/RequestBody";
        // String url = "http://127.0.0.1:8998/test/post/RequestParam";
        // String url = "http://127.0.0.1:8998/test/post";

        LinkedMultiValueMap<String, Object> paramMap = new LinkedMultiValueMap<>();
        paramMap.add("id","123");
        paramMap.add("name","张三");
        String result = restTemplate.postForObject(url, paramMap, String.class);
        //如果后台想要接收id或name参数,需要后端接收的名称和传递的名称一样
        System.out.println("result:" + result);
    }
```

2、使用Object作为参数（JSON提交）
```java
    @Test
    //使用Object作为参数（JSON提交）
    public void postForObject2() {
        RestTemplate restTemplate = new RestTemplate();
        String url = "http://127.0.0.1:8998/test/post";

        User user = new User(123, "张三");
        String result = restTemplate.postForObject(url, user, String.class);
        System.out.println("result:" + result);
    }
```

3、使用JSONObject作为参数（JSON提交）
```java
    @Test
    //使用JSONObject作为参数（JSON提交）
    public void postForObject3() {
        RestTemplate restTemplate = new RestTemplate();
        String url = "http://127.0.0.1:8998/test/post";

        JSONObject object = new JSONObject();
        object.put("id","123");
        object.put("name","张三");
        String result = restTemplate.postForObject(url, object, String.class);
        System.out.println("result:" + result);
    }
```

### postForEntity
```java
    @Test
    //使用LinkedMultiValueMap作为参数（Form表单提交）
    public void postForEntity1() {
        RestTemplate restTemplate = new RestTemplate();
        String url = "http://127.0.0.1:8998/test/post/RequestBody";

        HttpHeaders headers = new HttpHeaders();
        headers.set("token","token");
        headers.setContentType(MediaType.APPLICATION_JSON);

        LinkedMultiValueMap<String, Object> paramMap = new LinkedMultiValueMap<>();
        paramMap.add("id","123");
        paramMap.add("name","张三");

        HttpEntity<MultiValueMap<String, Object>> httpEntity = new HttpEntity<>(paramMap, headers);
        ResponseEntity<String> response = restTemplate.postForEntity(url, httpEntity, String.class);
        System.out.println("============================");
        System.out.println("result:" + response.getBody());
    }
```

```java
   @Test
    //使用Object作为参数（JSON提交）
    public void postForEntity2() {
        RestTemplate restTemplate = new RestTemplate();
        String url = "http://127.0.0.1:8998/test/post/RequestBody";

        HttpHeaders headers = new HttpHeaders();
        headers.set("token","token");
        // headers.setContentType(MediaType.APPLICATION_JSON);

        User user = new User(123, "张三");
        HttpEntity<User> httpEntity = new HttpEntity<>(user, headers);
        //我很想知道这里的httpEneity是什么结构，这个结构同时包含User和HttpHeaders

        ResponseEntity<String> response = restTemplate.postForEntity(url, httpEntity, String.class);
        System.out.println("result:" + response.getBody());
    }
```

```java
    @Test
    //使用JSONObject作为参数（JSON提交）
    public void postForEntity3() {
        RestTemplate restTemplate = new RestTemplate();
        String url = "http://127.0.0.1:8998/test/post/RequestBody";

        JSONObject object = new JSONObject();
        object.put("id","123");
        object.put("name","张三");

        ResponseEntity<String> response = restTemplate.postForEntity(url, object, String.class);
        System.out.println("result:" + response.getBody());
    }
```

### exchange
```java
    @Test
    public void exchange1 () {
        RestTemplate restTemplate = new RestTemplate();
        String url = "http://127.0.0.1:8998/test/post/RequestBody";

        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON);
        LinkedMultiValueMap<String, Object> paramMap = new LinkedMultiValueMap<>();
        paramMap.add("id","123");

        HttpEntity<MultiValueMap<String, Object>> httpEntity = new HttpEntity<>(paramMap, headers);
        ResponseEntity<String> response = restTemplate.exchange(url, HttpMethod.POST, httpEntity, String.class);
        System.out.println("==================");
        System.out.println(response.getBody());
    }
```

### getForObject
```java
    @Test
    public void getForObject() {
        RestTemplate restTemplate = new RestTemplate();
        String url = "http://127.0.0.1:8998/test/get/RequestParam?id={id}";

        HashMap<String, Object> paramMap = new HashMap<>();
        paramMap.put("id","123");

        String result = restTemplate.getForObject(url, String.class, paramMap);
        System.out.println(result);
    }
```

```java
    @Test
    public void getForEntity() {
        RestTemplate restTemplate = new RestTemplate();
        String url = "http://127.0.0.1:8998/test/get";

        HashMap<String, Object> paramMap = new HashMap<>();
        paramMap.put("id","123");

        ResponseEntity<String> response = restTemplate.getForEntity(url, String.class, paramMap);
        System.out.println(response.getBody());
    }
```



## Get请求

## 接受Controller中@RequestBody和@RequestParam
这里我写了五个个
- 四个请求：get请求和post按接收的注解不同分为@RequestBody和@RequestParam
- 一个返回类

```java
    @Data
    @AllArgsConstructor
    @NoArgsConstructor
    class OptResult implements Serializable {
        private static final long serialVersionUID = -6899059737730837772L;
        private String result = "1";
        private String msg;
    }

    @RequestMapping(value = "/post/RequestBody", method = RequestMethod.POST)
    public OptResult postForObject2(@RequestBody Map<String, Object> paramMap) {
        OptResult optResult = new OptResult();
        optResult.setMsg(paramMap.toString());
        optResult.setResult("1 == POST");
        return optResult;
    }

    @RequestMapping(value = "/post/RequestParam", method = RequestMethod.POST)
    public OptResult postForObject1(@RequestParam Map<String, Object> paramMap) {
        OptResult optResult = new OptResult();
        optResult.setMsg(paramMap.toString());
        optResult.setResult("1 == POST");
        return optResult;
    }

    @RequestMapping(value = "/get/RequestBody", method = RequestMethod.GET)
    public OptResult getForObject2(@RequestBody Map<String,Object> paramMap) {
        OptResult optResult = new OptResult();
        optResult.setMsg(paramMap.toString());
        optResult.setResult("1 == GET");
        return optResult;
    }

    @RequestMapping(value = "/get/RequestParam", method = RequestMethod.GET)
    public OptResult getForObject1(@RequestParam Map<String,Object> paramMap) {
        OptResult optResult = new OptResult();
        optResult.setMsg(paramMap.toString());
        optResult.setResult("1 == GET");
        return optResult;
    }
```

### @RequestBody

> @RequestBody主要用来接收前端传递给后端的json字符串中的数据的(请求体中的数据的)；GET方式无请求体，所以使用
> @RequestBody接收数据时，前端不能使用GET方式提交数据，而是用POST方式进行提交。在后端的同一个接收方法里，
> @RequestBody与@RequestParam()可以同时使用，@RequestBody最多只能有一个，而@RequestParam()可以有多个、

> @RequestBody接收的参数是来自爱requestBody中，即**请求体**。一般用于处理非`Content-Type:application/x-www-form-urlencoded`编码格式的数据，比如:`application/json`、`application/xml`等类型的数据。

GET请求中，通过HttpEntity传递的参数，必须要在请求头中声明数据的类型Content-Type，SpringMVC通过使用HandlerAdapter配置来解析HttpEntity中的数据，然后绑定到响应的bean上。

如果前端向后端传递的是非实体类对象，后端也可以使用 @RequestBody 注解，那就用`List<Map<String,String>>`来接收

### @RequestParam

用来处理`Content-Type`：为`application/x-www-form-urlencoded`编码的内容。（Http协议中，如果不指定`Content-Type`，则默认传递的参数就是`application/x-www-form-urlencoded`类型）

POST类型和GET类型都可以使用@RequestParam注解来接收参数

@RequestParam注解有三个参数
- required：表示是否必须，默认为true，必须。
- defaultValue：可设置请求参数的默认值，如果设置了该值，required=true将失效，自动为false，如果没有传该参数，就使用默认值。
- value：为接收url的参数名（相当于key值）

```java
//以下两种写法效果一模一样
@RequestParam("") 或 @RequestParam(value="")

//当配置多个属性的时候
@RequestParam(value="",required=true,defaultValue="")

//如果使用下面这种方式接值，那么前端传过来的参数名称就要和inputStr一样，这里才能接收到
@RequestParam String inputStr

//但是如果加上value属性的话，那么前端传过来的参数名称就要和value属性中的一致才能接收到
@RequestParam(value="aa") String inputStr

//如果加上required属性，当required=true就是必须要传参值过来，当required=false表示不传的话，会给参数赋值为null
@RequestParam(value="aa", required=true)

//有一种特殊情况是参数为int类型时，设置required=false后参数不传值得话会给int类型参数赋值为null，而int是基本数据类型不能赋值
@RequestParam(value="inptInt",required=false) int inputInt
```
