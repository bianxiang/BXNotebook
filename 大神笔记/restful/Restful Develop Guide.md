# Part III. 开发规范

## -1. 思考

1. 是否要控制分页查询时，一页条目的最大值。如一次最多查100个。
1. 实体要增加创建者字段，安全框架要限制只能删除或更新自己创建的对象。

## 0. 概述

1. 每个接口都要辨识好数据库格式、版本。  
	接受数据的接口要加注`@Consumes`，返回数据的接口要加注`@Produces`。  
	若接口既接受数据也响应数据，`@Consumes`、`@Produces`都要加。
2. 接口的不兼容性修改要升版本号。  
	例如，以下情况属于不兼容性修改:
	* 响应数据中删除了字段
	* 期望接收的数据增加了必填字段  
	
	以下情况一般是兼容的：
	* 响应数据增加了字段
	* 期望接收的数据去掉了某个字段

------------------------------------
## 1. 列表查询

标准写法
```
    @GET
    @Produces(RestfulConsts.JSON_CONTENT_WITH_VERSION + "1")
    public Response list(@QueryParam(RestfulConsts.QUERY_PARAM_PAGE) @DefaultValue("1") int page,
            @QueryParam(RestfulConsts.QUERY_PARAM_PAGE_SIZE) @DefaultValue("10") int pageSize) {
        return toResponse(selectAll(page, pageSize));
    }
```

1. 分页参数要使用标准的名称，已定义在常量类`RestfulConsts`。
1. 拿到`Page`对象后，调用`AbstractResource.toResponse(Page)`方法产生响应，不要自己构造响应。

------------------------------------

## 2. 获取特定资源GET

标准写法


    @GET
    @Path("/{id}")
    @Produces(RestfulConsts.JSON_CONTENT_WITH_VERSION + "1")
    public M get(@PathParam("id") Integer id) {
        M record = selectById(id);
        return ResourceUtils.returnExistedResource(record);
    }

1. 资源的主键通过路径变量注入
1. 如果资源找不到响应404。调用`ResourceUtils.returnExistedResource`，如果结果为`null`它会自动抛出`javax.ws.rs.NotFoundException.NotFoundException`异常，导致响应`404`。

注意事项

1. 不需要校验`id`是否为`null`。如果执行进入了该方法，`id`必然不为null。
1. 如果用户输入的主键不是整数（如/gifts/a），框架会响应`404`。注意，对于路径参数，结果不是`400 Bad Request`。

------------------------------------
# 3. 删除

    @DELETE
    @Path("/{id}")
    public Response delete(@PathParam("id") Integer id, @QueryParam("strict") @DefaultValue("false") boolean strict, 
    	@Context SecurityContext securityContext) {
        if (strict && !isResourceExisted(id)) {
            throw new NotFoundException();
        }

        int affectedRow = deleteById(id);
        // 若影响行数小于等于0，可能由并发删除在此操作前删除资源，也可能是删除失败
        if (affectedRow > 0) {
            return Response.noContent().build();
        }

        if (isResourceExisted(id)) {
            throw new InternalServerErrorException(UPDATE_AFFECT_0_ERROR);
        }

        // 若资源已移除
        if (strict) {
            throw new NotFoundException();
        } else {
            return Response.noContent().build();
        }
    }


1. 两种模式。参见Part II接口规范。
1. 成功响应`204 No Content`。
1. 要小心并发更新问题：在处理当前请求过程中，有另一个请求删除了资源。
1. 通过`@Context`注入安全上下文，可以拿到当前登录的用户。

------------------------------------
# 4. 插入

    @POST
    @Consumes(RestfulConsts.JSON_CONTENT_WITH_VERSION + "1")
    public Response insert(@NotNull(message = "The new object cannot be null") M record) {
        // 确保id为null
        record.setId(null);
        add(record);

        URI createdResourceUri = UriBuilder.fromResource(GiftResource.class).path(String.valueOf(record.getId()))
                .build();
        return Response.created(createdResourceUri).build();
    }

1. 要校验对象是否为null。如有必要做其他校验。
1. 一定要确保`id`为null。防止覆盖其他数据。
1. 要通知客户端产生的主键。通过响应`201`并提供资源`Location`实现。

------------------------------------

# 5. 更新

    @PUT
    @Path("/{id}")
    @Consumes(RestfulConsts.JSON_CONTENT_WITH_VERSION + "1")
    public Response update(@PathParam("id") Integer id, @NotNull(message = "Updated object cannot be null") M record) {
        if (!isResourceExisted(id)) {
            throw new NotFoundException();
        }

        // 确保id正确
        record.setId(id);

        int affectedRows = updateById(record);
        if (affectedRows <= 0) {
            if (!isResourceExisted(id)) {
                throw new NotFoundException();
            } else {
                throw new InternalServerErrorException(UPDATE_AFFECT_0_ERROR);
            }
        } else {
            return Response.noContent().build();
        }
    }

1. 要校验实体对象。
1. 要先检查对象是否存在，不存在抛`404`。
1. 要小心并发更新问题：在处理当前请求过程中，有另一个请求删除了资源。

------------------------------------

## 6. 异常规范

请熟悉JAX-RS的异常体系。Restful前端异常的根类是`javax.ws.rs.WebApplicationException`。
它有几个子类，最重要的是`javax.ws.rs.ClientErrorException`和`javax.ws.rs.ServerErrorException`分别标识了客户端和服务器的错误。
在向客户端响应时，首先应想清楚是客户端还是服务器端的错误。

客户端错误根据HTTP 响应码细分成更具体的错误，如`BadRequestException`、`NotFoundException`。在资源类中应根据情况抛出这些异常类。
框架会自动捕获这些异常并产生响应的响应。

------------------------------------

## 7. 参数校验

### 不需要（不应该）校验的情况

1. 参数类型校验由Restful框架负责

------------------------------------

## 8. 安全

### 8.1 在资源方法中获取当前登录的用户

在资源方法中可以通过`@Context SecurityContext securityContext`注入安全上下文。
通过	`SecurityContext.getUserPrincipal()`方法可以拿到一个	`java.security.Principal`对象。
这个对象可以进一步被强制类型转换为`UserSecurityInfo`。`UserSecurityInfo`是我们应用中表示用户安全信息的对象。
这个对象的内容大致是：

	public class UserSecurityInfo implements Principal {
		...
	    public String getUsername();
	    public boolean isAnonymous();
	    public boolean hasPermission(String securityResourceKey, String uriPattern, String httpMethod);
	    @Override
	    public String getName();
	    ...
	}

若调用接口不需要用户登录，将注入一个匿名用户。此时`UserSecurityInfo.isAnonymous()`为`true`。