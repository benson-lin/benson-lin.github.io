---
title: "jquery实现联动省市县获取"
layout: post
date: 2017-11-22
tags:
- JavaScript
categories: JAVASCRIPT
blog: true
excerpt: 
---

## jquery实现省市县获取



### 1 获取省份数据

```js
var allProvince = [];
getProvince();
function getProvince(){
	var data;
	 $.ajax({
        url:'/beian/ip/getAllProvinces',
        dataType: 'json',
        method: 'GET',
    }).done(function(result){
        if (result.code) {
            return AlertDialog.error(result.message);
        }
        allProvince = result.data;
    }).fail(function(){
        return AlertDialog.error('无法获取省份地址信息！');
    });
    return data;
}
```

### 2.1 使用模板的形式

函数根据省份填入方式不同参数个数不同：
- 使用前端模板加入省份：好处是不用在每次调用函数时都传入省份参数
- 使用js加入省份数据

```js
<!-- html代码如下 -->
<div class="form-group">
	<label class="control-label col-sm-3"><span class="text-danger">*</span>网关物理地址：</label>
	<div class="col-sm-8">
		<select id="new-gateway-province">
		 <option value=""></option>
		 @\{{ each province as value}}
				<option value="@\{{value.Id}}">
					@\{{value.Mc}}
				</option>
		 @\{{/each}}
		</select>
		<select id="new-gateway-city">
		</select>
		<select id="new-gateway-town">
		</select>
	</div>
</div>
```

```js
/**
 * @param {} provinceDom 省节点
 * @param {} cityDom 市节点
 * @param {} townDom 镇节点
 * @param {} provinceCode 省代码，在编辑时加入该参数默认添加值，省市县代码如果有一个则其他两个需同时存在
 * @param {} cityCode 市代码，在编辑时加入该参数默认添加值
 * @param {} townCode 区代码，在编辑时加入该参数默认添加值
 */				
provinceCityTownChange = function(provinceDom, cityDom, townDom, provinceCode, cityCode, townCode){
 	
	provinceDom.change(function(){
		townDom.empty();
		cityDom.empty();
		if (!$(this).val()) {
			return false;
		}
		$.ajax({
	        url:'/beian/ip/getAllCities',
            data: {code: $(this).val()},
	        dataType: 'json',
	        method: 'GET',
	    }).done(function(result){
	        if (result.code) {
	            return AlertDialog.error(result.message);
	        }
	        var city = result.data;
	        var cityOption = "";
			$.each(city, function(index, value){
				cityOption += '<option value="'+value.Id+'">'+value.Mc+'</option>'
			});
			cityDom.append(cityOption);
			if (cityCode != undefined) {
				cityDom.val(cityCode).trigger('change');
			} else {
				cityDom.val(city[0].Id).trigger('change');
			}
			
	    }).fail(function(){
	        return AlertDialog.error('无法获取城市地址信息！');
	    });
	});
	
	cityDom.change(function(){
		townDom.empty();
		$.ajax({
		        url:'/beian/ip/getAllTowns',
	            data: {code: $(this).val()},
		        dataType: 'json',
		        method: 'GET',
		    }).done(function(result){
		        if (result.code) {
		            return AlertDialog.error(result.message);
		        }
		        var town = result.data;
		        var townOption = "";
				$.each(town, function(index, value){
					townOption += '<option value="'+value.Id+'">'+value.Mc+'</option>'
				});
				townDom.empty().append(townOption);
				if (townCode != undefined) {
					townDom.val(townCode).trigger('change');
				} 
		    }).fail(function(){
		        return AlertDialog.error('无法获取区地址信息！');
		    });
	});
	
	if (provinceCode != undefined) {
		provinceDom.val(provinceCode).trigger('change');
	}
 }
```


```js
//获取节点并绑定
//对添加的情况
var provinceDom = html.find("#new-gateway-province");
var cityDom = html.find("#new-gateway-city");
var townDom = html.find("#new-gateway-town");
provinceCityTownChange(provinceDom, cityDom, townDom);


//对编辑的情况
var gatewayProvince = 440000;
var gatewayCity = 440600;
var gatewayTown = 440607;//广东省佛山市三水区
var provinceDom = body.find("#new-gateway-province");
var cityDom = body.find("#new-gateway-city");
var townDom = body.find("#new-gateway-town");
provinceCityTownChange(provinceDom, cityDom, townDom, gatewayProvince, gatewayCity, gatewayTown);				
				
```

### 2.2 使用js的形式


```js
//在函数最开始append省份数据，调用函数时需传入省份参数
provinceCityTownChange = function(provinceDom, cityDom, townDom, allProvince, provinceCode, cityCode, townCode){
 	
	var provinceOption = '<option value=""></option>';
	$.each(allProvince, function(index, province){
		provinceOption += '<option value="'+province.Id+'">'+province.Mc+'</option>';
	});
	provinceDom.append(provinceOption);
	
	provinceDom.change(function(){
		townDom.empty();
		cityDom.empty();
		if (!$(this).val()) {
			return false;
		}
		$.ajax({
	        url:'/beian/ip/getAllCities',
            data: {code: $(this).val(),accountType: $('select[name=accountType]').val()},
	        dataType: 'json',
	        method: 'GET',
	    }).done(function(result){
	        if (result.code) {
	            return AlertDialog.error(result.message);
	        }
	        var city = result.data;
	        var cityOption = "";
			$.each(city, function(index, value){
				cityOption += '<option value="'+value.Id+'">'+value.Mc+'</option>'
			});
			cityDom.append(cityOption);
			if (cityCode != undefined) {
				cityDom.val(cityCode).trigger('change');
			} else {
				cityDom.val(city[0].Id).trigger('change');
			}
			
	    }).fail(function(){
	        return AlertDialog.error('无法获取城市地址信息！');
	    });
	});
	
	cityDom.change(function(){
		townDom.empty();
		$.ajax({
		        url:'/beian/ip/getAllTowns',
	            data: {code: $(this).val(), accountType: $('select[name=accountType]').val()},
		        dataType: 'json',
		        method: 'GET',
		    }).done(function(result){
		        if (result.code) {
		            return AlertDialog.error(result.message);
		        }
		        var town = result.data;
		        var townOption = "";
				$.each(town, function(index, value){
					townOption += '<option value="'+value.Id+'">'+value.Mc+'</option>'
				});
				townDom.empty().append(townOption);
				if (townCode != undefined) {
					townDom.val(townCode).trigger('change');
				} 
		    }).fail(function(){
		        return AlertDialog.error('无法获取区地址信息！');
		    });
	});
	
	if (provinceCode != undefined) {
		provinceDom.val(provinceCode).trigger('change');
	}
 }
 
```

```js
//新增时
var provinceDom = html.find("#new-gateway-province");
var cityDom = html.find("#new-gateway-city");
var townDom = html.find("#new-gateway-town");
provinceCityTownChange(provinceDom, cityDom, townDom, allProvince);

//编辑时
var provinceDom = body.find("#new-gateway-province");
var cityDom = body.find("#new-gateway-city");
var townDom = body.find("#new-gateway-town");
provinceCityTownChange(provinceDom, cityDom, townDom, allProvince, gatewayProvince, gatewayCity, gatewayTown);
				
```




