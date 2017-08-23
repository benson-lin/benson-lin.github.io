---
title: "laravel框架常见sql语句转换"
layout: post
date: 2017-08-23
tags:
tags:
- PHP
categories: PHP
blog: true
---

## groupBy中有count函数


```php
//下面的sql记录了某人的在某个时间段内某种操作一共有分别有多少个
// select operator, operation, count(operation) from mou_table where operate_time>'2017-08-23 00:00:00' and operate_time<'2017-08-23 23:59:59' group by operator, operation;

$records = MouModel::select('operator', 'operation', DB::raw('count(operation) as count'))
                ->where('operate_time', '>', $start)
                ->where('operate_time', '<', $end)
                ->groupBy('operator', 'operation')->get()->toArray();

```


## 使用源生的sql语句

```php
use Illuminate\Support\Facades\DB;

$sqlQuery = 'select ...';
DB::connection('connection')->select($sqlQuery);

MouModel::whereRaw($whereRaw);
```

## 批量删除操作

```php
	MouModel::whereIn('id', $ids)->delete();
```

## 有则插入，没有则更新

```php
//如果有id则更新remark和时间
   MouModel::updateOrCreate([
		'id' => $id,
	],[
		'time' => timetostr(time()),
		'remark' => $remark,
	]);
```


## 拿第一条

```php
$record = MouModel::where('time', '<', $time)->orderBy('time', 'desc')->first();
//拿到字段后可以直接通过->拿值，要提前用isEmpty判断是否有值
echo $record->name;
```

## 所有查询条件都可选技巧，多model服用同样查询条件

```php
$builder = MouModel1::whereRaw('1 = 1');
switch ($type) {
	case 0: $builder = MouModel1::whereRaw('1 = 1');break;
	case 1: $builder = MouModel2::whereRaw('1 = 1');break;
	case 2: $builder = MouModel3::whereRaw('1 = 1');break;

}

if (!empty($id))  $builder->where('id', $id);
```

## 分页

```php
$result = MouModel1::whereIn('id', $ids)->paginate($length)->toArray();
//获取data和total
$data = $result['data'];
$total = $result['total'];
        
//Paginator::setCurrentPage(2);设置某一页
//返回的数据如下
{
   "total": 50,
   "per_page": 15,
   "current_page": 1,
   "last_page": 4,
   "next_page_url": "http://laravel.app?page=2",
   "prev_page_url": null,
   "from": 1,
   "to": 15,
   "data":[
        {
            // Result Object
        },
        {
            // Result Object
        } 
   ]
}

//或者
$limit = $input['limit'];
$skip = ($input['page'] - 1) * $limit;
$model = $model->where('...', '');
$data = $model->skip($skip)->take($limit)->get()->toArray();
$count = $model->count();
```

