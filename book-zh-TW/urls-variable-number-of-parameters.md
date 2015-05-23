含參數編號的網址
=======================================

在很多狀況下，你會需要從網址收到一系列的參數編號。

舉例來說，你可能希望像是 `http://example.com/products/cars/sport` 這樣的網址，會觸發 `ProductController::actionCategory`，並且該 action 收到一個陣列，包含 `cars` 和 `sport`兩個元素。

準備
---------

首先，我們要開啟 pretty URLs。我們修改 config 如下： 

```php
$config = [
    // ...
    'components' => [
        // ...
        'urlManager' => [
            'showScriptName' => false,
            'enablePrettyUrl' => true,
            'rules' => require 'urls.php',
        ],
    ]
```

注意這邊我們使用不同的檔案來處理規則，而不是直接寫在 config 裡面。這在程式逐漸成長時會很有幫助。

現在，在 `config/urls.php` 加入以下內容：

```php
<?php
return [
    'products/<categories:.*>' => 'product/category',
];
```

再來，建立 `ProductController`：

```php
namespace app\controllers;

use yii\web\Controller;

class ProductController extends Controller
{
    public function actionCategory($categories)
    {
        $params = explode('/', $categories);
        print_r($categories);
    }
}
```

完成了，現在你可以嘗試 `http://example.com/products/cars/sport`，你會看到

```
Array ( [0] => cars [1] => sport)
```
