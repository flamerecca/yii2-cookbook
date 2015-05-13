使用與客製化 captcha
=============================

根據[維基百科](http://en.wikipedia.org/wiki/Captcha) CAPTCHA 代表「完全自動公開圖零測試，分辨對方是電腦還是人類」（Completely Automated Public Turing test to tell Computers and Humans Apart）。換句話說，CAPTCHA 提供一個對人類很簡單但是電腦很難解的問題。
CAPTCHA 的目的是避免網站被人用程式濫用，像是在留言欄裡面寫惡意網站的網址，或者選舉時大量灌票。


對目前的電腦來說，圖像辨認還是一個蠻困難的問題，所以一個 CAPTCHA 常見的做法是顯示一張圖片，要求使用者輸入裡面的內容。

如何在表單裡面加入 CAPTCHA
-------------------------

Yii provides a set of ready to use classes to add CAPTCHA to any form. Let's review how it could be done.


First of all, we need an action that will display an image containing text. Typical place for it is `SiteController`.
Since there's ready to use action, it could be added via `actions()` method:

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

In the above we're reusing `yii\captcha\CaptchaAction` as `site/captcha` route. `fixedVerifyCode` is set for
test environment in order for the test to know which answer is correct.

Now in the form model (it could be either ActiveRecord or Model) we need to add a property that will contain
user input for verification code and validation rule for it:

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

Now we can actually display image and verification input box in a view containing a form:

```php
<?php $form = ActiveForm::begin(['id' => 'contact-form']); ?>
    // ...
    <?= $form->field($model, 'verifyCode')->widget(Captcha::className()) ?>
    // ...
<?php ActiveForm::end(); ?>
```

That's it. Now robots won't pass. At least dumb ones.

Simple math captcha
-------------------

Nowadays CAPTCHA robots are relatively good at parsing image so while by using typical CAPTCHA
you're significanly lowering number of spammy actions, some robots will still be able to parse the image
and enter the verification code correctly.

In order to prevent it we have to increase the challenge. We could add extra ripple and special effects
to the letters on the image but while it could make it harder for computer, it certainly will make it
significantly harder for humans which isn't really what we want.

A good solution for it is to mix a custom task into the challenge. Example of such task could be
a simple math question such as "2 + 1 = ?". Of course, the more unique this question is, the more
secure is the CAPTCHA.

! example of implementing captcha that uses simple math as a question.
