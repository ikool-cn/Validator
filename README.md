## PHP数据验证器 支持无限级对象数据验证，使用简单强大


#### 安装
引入Validator.php类文件即可
```php
require 'Validator.php';
```
#### 验证单个数据
1. Validator内置了很多静态方法可以直接调用
```php
var_dump(Validator::isMobile('12345678901'));
//bool(false)
```
2. 过滤数据
```php
var_dump(Validator::filterUtf8('Kiss 💋 me'));
//string(8) "Kiss  me"
```

#### 常规的数据验证
```php
$posts = [
    'username' => 'ikool',
    'passwd' => 'abc123',
    'age' => 25
];
$rule = [
    'username' => 'required|is_alpha_num|minlen:6',
    'passwd' => 'required|minlen:8',
    'age' => 'required|gte:18|lte:35',
];
$v = new Validator();
if ($v->setRules($rule)->validate($posts)) {
    print_r($v->getData());
} else {
    echo $v->getErrorString();
}
//username最小长度为6个字符
```

#### 自定义TAG和自定义错误提示
上面的demo中返回的错误是由系统自动生成的，可以自定义TAG或者自定义错误提示

1. 自定义TAG 使用单个反引号包裹
```php
$posts = [
    'username' => 'ikool',
    'passwd' => 'abc123',
    'age' => 25
];
$rule = [
    'username' => 'required|is_alpha_num|minlen:6 `用户名`',
    'passwd' => 'required|minlen:8',
    'age' => 'required|gte:18|lte:35',
];
$v = new Validator();
if ($v->setRules($rule)->validate($posts)) {
    print_r($v->getData());
} else {
    echo $v->getErrorString();
}
//用户名最小长度为6个字符
```

2. 自定义错误 使用双反引号包裹
```php
$posts = [
    'username' => 'ikool',
    'passwd' => 'abc123',
    'age' => 25
];
$rule = [
    'username' => 'required|is_alpha_num|minlen:6 ``用户名必须是字母和数字组成哦``',
    'passwd' => 'required|minlen:8',
    'age' => 'required|gte:18|lte:35',
];
$v = new Validator();
if ($v->setRules($rule)->validate($posts)) {
    print_r($v->getData());
} else {
    echo $v->getErrorString();
}
//用户名必须是字母和数字组成哦
```

#### 自动去空格
系统默认开启了字符串去除两端空格 可以使用 `$v->setFieldAutoTrim(false);` 来关闭
```php
$posts = [
    'username' => ' ikool ',
    'passwd' => 'abc123',
];
$rule = [
    'username' => 'required|minlen:5',
    'passwd' => 'required|minlen:6 `密码`',
];
$v = new Validator();
$v->setFieldAutoTrim(false);
if ($v->setRules($rule)->validate($posts)) {
    var_dump($v->getDataByField('username'));
} else {
    echo $v->getErrorString();
}
//string(7) " ikool "
```

#### 调用PHP内置函数
任何callable函数都可以被用在规则里面，如下trim和substr
```php
$posts = [
    'username' => ' ikool ',
    'passwd' => 'abc123',
];
$rule = [
    'username' => 'required|trim|substr:0,3',
    'passwd' => 'required|minlen:6 `密码`',
];
$v = new Validator();
$v->setFieldAutoTrim(false);
if ($v->setRules($rule)->validate($posts)) {
    var_dump($v->getDataByField('username'));
} else {
    echo $v->getErrorString();
}
//string(3) "iko"
```

#### 参数位置占位符@me
当前参数传递不是函数第一个参数时需要用@me做占位，如：str_replace:o,a,@me 把o替换成a
```php
$posts = [
    'username' => ' ikool ',
    'passwd' => 'abc123',
];
$rule = [
    'username' => 'required|trim|str_replace:o,a,@me',
    'passwd' => 'required|minlen:6 `密码`',
];
$v = new Validator();
if ($v->setRules($rule)->validate($posts)) {
    var_dump($v->getDataByField('username'));
} else {
    echo $v->getErrorString();
}
//string(5) "ikaal"
```

#### 自定义验证函数
1. 自定义函数返回布尔值表示验证
```php
$v->setTagMap('is_num', function ($var) {
    return is_numeric($var) ? true : false;
});
```

2. 自定义函数返回非布尔值表示过滤
```php
$v->setTagMap('filter_num', function ($var) {
    return preg_replace('/[^\d]/', '', $var);
});
```

完整的例子
```php
$posts = [
    'username' => ' ikool ',
    'qq' => 'abc123bcd456',
];
$rule = [
    'username' => 'required|minlen:5',
    'qq' => 'required|filter_num|minlen:6 `QQ`',
];
$v = new Validator();
$v->setTagMap('filter_num', function ($var) {
    return preg_replace('/[^\d]/', '', $var);
});
if ($v->setRules($rule)->validate($posts)) {
    var_dump($v->getDataByField('qq'));
} else {
    echo $v->getErrorString();
}
//string(6) "123456"
```

#### 两个字段值必须相同(same用法)
一般用在密码和重复密码比较的场景
```php
$posts = [
    'username' => 'ikool',
    'passwd' => 'abc123',
    'repasswd' => 'abc12',
];
$rule = [
    'username' => 'required|is_alpha_num|minlen:5',
    'passwd' => 'required|minlen:6 `密码`',
    'repasswd' => 'required|same:passwd ``密码必须和重复密码一致``',
];
$v = new Validator();
if ($v->setRules($rule)->validate($posts)) {
    print_r($v->getData());
} else {
    echo $v->getErrorString();
}
//密码必须和重复密码一致
```

### 无限级数据验证
```php
$post = [
    'username' => 'abc123',
    'mobile' => '13345678901',
    'attr' => [
        'sex' => 0,
        'age' => 28,
        'shoe' => [
            'brand' => 'NB',
            'size' => 41,
        ]
    ],
];
$rule = [
    'username' => 'required|minlen:6',
    'mobile' => 'required|is_mobile',
    'attr' => [
        'sex' => 'required|in:0,1,2',
        'age' => 'required|gte:18|lte:35',
        'shoe' => [
            'brand' => 'required',
            'size' => 'required|gte:18|lte:45',
        ]
    ]
];
$v = new Validator();
if ($v->setRules($rule)->validate($post)) {
    print_r($v->getData());
} else {
    echo $v->getErrorString();
}
//Array
//(
//    [username] => abc123
//    [mobile] => 13345678901
//    [attr] => Array
//        (
//            [sex] => 0
//            [age] => 28
//            [shoe] => Array
//                (
//                    [brand] => NB
//                    [size] => 41
//                )
//
//        )
//
//)
```
* 注意：无限级数据规则设置支持对象无限级 但不支持数组，如下数据结构
```php
$post = [
    'username' => 'abc123',
    'mobile' => '13345678901',
    'address' => [
       ['provice' => '北京', 'city' => '北京'],
       ['provice' => '江苏', 'city' => '浙江'],
    ],
];
```
解决办法：使用setTagMap来设置定义函数迭代验证


#### 获取错误
1. 获取字符格式的错误
```php
...
echo $v->getErrorString();
//密码必须和重复密码一致
```

2. 获取数组形式的错误
```php
...
print_r($v->getError());
//Array
//(
//    [repasswd] => 密码必须和重复密码一致
//)
```

#### 获取结果数据
1. 获取所有数据 含未验证字段 `$v->getAllData()`
```php
$posts = [
    'username' => 'ikool',
    'age' => '123s',
    'hobby' => 'sport',
];
$rule = [
    'username' => 'required|minlen:5',
    'age' => 'required|intval',
];
$v = new Validator();
if ($v->setRules($rule)->validate($posts)) {
    print_r($v->getAllData());
} else {
    print_r($v->getError());
}
//Array
//(
//    [username] => ikool
//    [age] => 123
//    [hobby] => sport
//)
```

2. 获取已设置验证规则的数据 不含未验证字段 `$v->getData()`
* 注意：当字段未设置required属性且未传递该字段时 结果不含该字段
```php
$posts = [
    'username' => 'ikool',
    'age' => '123s',
    'hobby' => 'sport',
];
$rule = [
    'username' => 'required|minlen:5',
    'age' => 'required|intval',
];
$v = new Validator();
if ($v->setRules($rule)->validate($posts)) {
    print_r($v->getData());
} else {
    print_r($v->getError());
}
//Array
//(
//    [username] => ikool
//    [age] => 123
//)
```

3. 获取某个字段的值 支持链式结构 foo.bar
```php
$post = [
    'username' => 'abc123',
    'mobile' => '13345678901',
    'attr' => [
        'sex' => 0,
        'age' => 28,
        'shoe' => [
            'brand' => 'NB',
            'size' => 41,
        ]
    ],
];
$rule = [
    'username' => 'required|minlen:6',
    'mobile' => 'required|is_mobile',
    'attr' => [
        'sex' => 'required|in:0,1,2',
        'age' => 'required|gte:18|lte:35',
        'shoe' => [
            'brand' => 'required',
            'size' => 'required|gte:18|lte:45',
        ]
    ]
];
$v = new Validator();
if ($v->setRules($rule)->validate($post)) {
    var_dump($v->getDataByField('mobile'));
    var_dump($v->getDataByField('attr.shoe.brand'));
} else {
    echo $v->getErrorString();
}
//string(11) "13345678901"
//string(2) "NB"
```

#### 内置规则TAG列表
```
required
len
minlen
maxlen
width
minwidth
maxwidth
gt 
lt 
gte
lte
eq
neq
in
nin
same
is_mobile
is_email
is_idcard
is_ip
is_url
is_array
is_float 
is_int
is_numeric
is_string
is_natural
is_natural_no_zero
is_hanzi 
is_mongoid 
is_alpha
is_alpha_num
is_alpha_num_dash
```

#### 内置函数列表
```php
public function required($var): bool
public function same($var, $compare_var): bool
public static function len($var, $len): bool
public static function minlen($var, $len): bool
public static function maxlen($var, $len): bool
public static function width($var, $len): bool
public static function minwidth($var, $len): bool
public static function maxwidth($var, $len): bool
public static function isMobile($var): bool
public static function isEmail($var): bool
public static function isIp($var, $type = 'ipv4'): bool
public static function isUrl($var): bool
public static function isIdcard($var): bool //是否为身份证号码
public static function isNatural($var): bool
public static function isNaturalNoZero($var): bool
public static function isHanzi($var): bool //是否汉字
public static function isMongoid($var): bool
public static function isAlpha($var): bool
public static function isAlphaNum($var): bool
public static function isAlphaNumDash($var): bool
public static function gt($var, $min): bool
public static function lt($var, $max): bool
public static function gte($var, $min): bool
public static function lte($var, $max): bool
public static function eq($var, $obj): bool
public static function neq($var, $obj): bool
public static function in($var, ...$set): bool
public static function nin($var, ...$set): bool
public static function filterUtf8($var): string //过滤3字节以上的字符
```