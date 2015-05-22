紀錄：問題與解決方法
===============================

在Yii裡面，紀錄(Logging) 是非常有彈性的。基本上很簡單，但是要取得所有想要的功能，有時很花時間。下面是一些解決方案。
希望你可以找到需要的功能

紀錄404頁面，並且不發送email
---------------------------------------------

404 not found頁面太常出現了，似乎不大值得寄email提醒。不過，紀錄404頁面有時還是很有幫助的，我們來實做看看！


```php
'components' => [
    'log' => [
        'targets' => [
            'file' => [
                'class' => 'yii\log\FileTarget',
                'categories' => ['yii\web\HttpException:404'],
                'levels' => ['error', 'warning'],
            ],
            'email' => [
                'class' => 'yii\log\EmailTarget',
                'except' => ['yii\web\HttpException:404'],
                'levels' => ['error', 'warning'],
                'message' => ['from' => 'robot@example.com', 'to' => 'admin@example.com'],
            ],
        ],
    ],
],
```

When there's unhandled exception in the application Yii logs it additionally to displaying it
to end user or showing customized error page. Exception message is what actually to be written and
the fully qualified exception class name is the category we can use to filter messages when
configuring targets. 404 can be triggered by throwing `yii\web\NotFoundHttpException` or automatically.
In both cases exception class is the same and is inherited from `\yii\web\HttpException` which is a bit
special in regards to logging. The speciality is the fact that HTTP status code prepended by `:` is
appended to the end of the log message category. In the above we're using `categories` to include
and `except` to exclude 404 log messages.

即時紀錄
-----------------

By default Yii accumulates logs till the script is finished or till the number of logs accumulated is
enough which is 1000 messages by default for both logger itself and log target. It could be that you
want to log messages immediately. For example, when running an import job and checking logs to see
the progress. In this case you need to change settings via application config file:

```php
'components' => [
    'log' => [
        'flushInterval' => 1, // <-- here
        'targets' => [
            'file' => [
                'class' => 'yii\log\FileTarget',
                'levels' => ['error', 'warning'],
                'exportInterval' => 1, // <-- and here
            ],
        ],
    ]
]
```
