# 9. 高级Express

## （未）9.1 身份验证

本章使用Redis做数据库。

### 9.1.1 保存和加载用户

## （未）9.2 高级路由技术

## （未）9.3 REST API

认证API使用Connect的`basicAuth()`中间件。

### 9.3.1 设计API

令请求以`/api`开头。

	app.get('/api/user/:id', api.user);
	app.get('/api/entries/:page?', api.entries);
	app.post('/api/entry', api.add);

### （未）9.3.2 Adding Basic authentication

### 9.3.3 实现路由

先实现`GET /api/user/:id`。找不到响应404。 

`routes/api.js`：

	exports.user = function(req, res, next){
		User.get(req.params.id, function(err, user){
			if (err) return next(err);
			if (!user.id) return res.send(404);
			res.json(user);
		});
	};

Next, add the following route path to app.js:

	app.get('/api/user/:id', api.user);

测试：
	
	$ curl http://tobi:ferret@127.0.0.1:3000/api/user/1 -v

为防止敏感数据被返回，创建一个`toJSON`方法：

	User.prototype.toJSON = function(){
		return {
			id: this.id,
			name: this.name
		}
	};

