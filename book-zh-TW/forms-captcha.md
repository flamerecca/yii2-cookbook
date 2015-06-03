使用與客製化 captcha
=============================

根據[維基百科](http://en.wikipedia.org/wiki/Captcha) CAPTCHA 代表「完全自動公開圖零測試，分辨對方是電腦還是人類」（Completely Automated Public Turing test to tell Computers and Humans Apart）。換句話說，CAPTCHA 提供一個對人類很簡單但是電腦很難解的問題。
CAPTCHA 的目的是避免網站被人用程式濫用，像是在留言欄裡面寫惡意網站的網址，或者選舉時大量灌票。


對目前的電腦來說，圖像辨認還是一個蠻困難的問題，所以一個 CAPTCHA 常見的做法是顯示一張圖片，要求使用者輸入裡面的內容。

如何在表單裡面加入 CAPTCHA
-------------------------

Yii 提供各種表單一些可以馬上使用的 CAPTCHA ，我們來看看該怎麼做。


首先，我們需要一個顯示含文字圖片的動作。常見的地方是放在 `SiteController`裡面。
既然是一個馬上要使用的動作，我們可以透過 `actions()` 這個 method 加入：

```php
class SiteController extends Controller
{
    // ...
    public function actions()
    {
        return [
            // ...
            'captcha' => [
                'class' => 'yii\captcha\CaptchaAction',
                'fixedVerifyCode' => YII_ENV_TEST ? 'testme' : null,
            ],
        ];
    }
    
    // ...
}
```

上面的 code ， 我們使用 `yii\captcha\CaptchaAction` 作為 `site/captcha` 的 route， `fixedVerifyCode` 的設定是為了測試環境用，以確定答案判斷是否正確。

然後在表單的模型裡面（可能是 ActiveRecord 或者 Model），我們在 rule() 裡面加入一個新的參數，作為使用者輸入的驗證碼規則：

```php
class ContactForm extends Model
{
    // ...
    public $verifyCode;
    
    // ...
    public function rules()
    {
        return [
            // ...
            ['verifyCode', 'captcha'],
        ];
    }
    
    // ...
}
```

現在我們可以在表單裡面，顯示驗證碼的圖片以及輸入框：

```php
<?php $form = ActiveForm::begin(['id' => 'contact-form']); ?>
    // ...
    <?= $form->field($model, 'verifyCode')->widget(Captcha::className()) ?>
    // ...
<?php ActiveForm::end(); ?>
```

好了！現在機器人被擋住了（起碼笨的機器人被擋住了）。

用簡單數學作 CAPTCHA
-------------------

在最近， 程式越來越擅長處理圖形辨識。
所以傳統的 CAPTCHA 作法效果越來越差了，因為有部分的機器人可以正確判斷你的 CAPTCHA 圖片並且輸入對應的文字。

要避免這些攻擊，我們需要提升難度。我們可以在圖片上面加上一些漣漪和特效，不過這作法在對電腦辨識提升難度的同時，也同時對人類的辨識提升了難度，這當然不是我們希望看見的。

一個好的解決方法，是加入一些需要人腦判斷的問題。像一些簡單的數學問題（比如2 + 1 = ?）。當然，這些問題越是特別，CAPTCHA 的安全性就越高。

! example of implementing captcha that uses simple math as a question.
