############
本土化
############

.. contents::
    :local:
    :depth: 2

********************
語言環境操作
********************

CodeIgniter 提供了一些工具來幫助你將你的應用程序本土化為不同的語言。
雖然應用程式的完全本土化是一個複雜的課題，但在你的應用程序中用不同的支持語言來轉換字串是很簡單的。

語言字串儲存在 **app/Language** 目錄中，每一個支援的語言都有一個子目錄::

    /app
        /Language
            /en
                app.php
            /fr
                app.php

.. important:: 語言環境偵測只適用於使用 IncomingRequest 類別的網路請求，命令列請求將不具有這些功能。

語言環境配置
======================

每個網站都有它們預設運行的語言/區域設定。這可以在 **Config/App.php** 中設定::

    public $defaultLocale = 'en';

該值可以是應用程式用於管理文字字串和其他格式的任何字串。建議使用 `BCP 47 <http://www.rfc-editor.org/rfc/bcp/bcp47.txt>`_ 語言碼。
這種語言碼，如 en-US 代表美式英語， fr-FR 代表法語/法國。
有關此內容的更詳細的介紹，請訪問 `W3C's site <https://www.w3.org/International/articles/language-tags/>`_ 。

該系統足夠智慧，如果找不到完全匹配，可以回退到更通用的語言碼。
如果語言環境被設置為 **en-US** ，而我們只設置了 **en** 的語言文件，那麼這些文件將被使用，
因為不存在更具體的 **en-US** 。然而，如果在 **app/Language/en-US** 存在一個語言目錄，那麼它將被首先使用。

語言環境檢測
================

支援兩種方法在請求期間檢測正確的語言環境。
第一種是「設置並忘記」方法，它將自動執行 :doc:`內容協商 </incoming/content_negotiation>` ，以便您確定要使用的正確語言環境。
第二種方法允許您在路由中指定將用於設定語言環境的區段。

如果你需要直接設定語言環境，你可以使用 ``IncomingRequest::setLocale(string $locale)`` 。

內容協商
-------------------

你可以通過在 設定/應用程式 中設定兩個額外的設定，使內容協商自動發生。
第一個值告訴請求類別，我們確實想協商一個語言環境的問題，所以只需將其設置為 true 。
::

    public $negotiateLocale = true;

啟用此功能后，系統將根據您在 ``$supportLocales`` 中定義的語言環境陣列來自動協商正確的語言。
如果在您支援的語言和請求的語言之間找不到匹配項，則將使用 $supportedLocales 中的第一項。
在下面的範例中，如果未找到匹配項，將使用 **en** 語言環境設定
::

    public $supportedLocales = ['en', 'es', 'fr-FR'];

路由
---------

第二種方法使用自定義的置換符號來檢測所需的語言環境，並在請求上設定它。
置換符號 ``{locale}`` 可以作為區段放在路由中。如果存在，則匹配區段的內容將是您的語言環境
::

    $routes->get('{locale}/books', 'App\Books::index');

在此範例中，如果使用者嘗試訪問 ``http://example.com/fr/books`` ，
則語言環境將設置為 ``fr`` ，假設它已設定為有效的語言環境。

.. note:: 如果該值不符合應用程式組態設定文件中定義的有效的語言環境，將使用默認的語言環境來代替。

檢索當前的語言環境
=============================

當前語言環境始終可以通過 ``getLocale()`` 方法從 IncomingRequest 物件中檢索。
如果您的控制器繼承 ``CodeIgniter\Controller``，則可以通過 ``$this->request`` 獲得此功能。
::

    <?php

    namespace App\Controllers;

    class UserController extends \CodeIgniter\Controller
    {
        public function index()
        {
            $locale = $this->request->getLocale();
        }
    }

或者，你可以使用 :doc:`服務類別 </concepts/services>` 來檢索當前的請求。
::

    $locale = service('request')->getLocale();

*********************
語言本土化
*********************

創建語言檔
=======================

語言沒有任何特定的命名規則要求。文件的命名應該符合邏輯，以描述它所容納的內容類型。
例如，假設你想創建一個包含錯誤信息的文件。你可以簡單地命名它為 **Errors.php** 。

在檔案中，您將返回一個陣列，其中陣列中的每個元素都有一個語言鍵，並且可以具有要返回的字串。
::

    'language_key' => 'The actual message to be shown.'

它還支援巢狀定義
::

    'language_key' => [
        'nested' => [
            'key' => 'The actual message to be shown.',
        ],
    ],

.. note:: 最好對給定檔案中的所有訊息使用通用前綴，以避免與其他檔案中名稱相似的項目發生衝突。
    例如，如果要創建錯誤消息，則可以在它們前面加上 error\_ 。
::

    return [
        'errorEmailMissing'    => 'You must submit an email address',
        'errorURLMissing'      => 'You must submit a URL',
        'errorUsernameMissing' => 'You must submit a username',
        'nested'               => [
            'error' => [
                'message' => 'A specific error message',
            ],
        ],
    ];

基本用法
===========

你可以使用 ``lang()`` 輔助函數從任何一個語言檔案中檢索文本，方法是將檔案名和語言鍵作為第一個參數，
用句號（.）分開。例如，要從 ``Errors`` 語言檔案中加載 ``errorEmailMissing`` 字串，你需要執行以下操作：
::

    echo lang('Errors.errorEmailMissing');

對於巢狀定義，你需要執行以下操作：
::

    echo lang('Errors.nested.error.message');

如果要求的語言鍵不存在於當前語言環境的檔案中，該字串將被傳回，沒有改變。
在這個例子中，如果它不存在，它將返回 'Errors.errorEmailMissing' 或 'Errors.nested.error.message' 。

替換參數
--------------------

.. note:: 
    以下功能都需要在系統上載入 `intl <https://www.php.net/manual/en/book.intl.php>`_ 擴充才能正常工作。
    如果未載入擴充，則不會嘗試替換。
    一個很好的概述可以在 `Sitepoint <https://www.sitepoint.com/localization-demystified-understanding-php-intl/>`_ 上找到。

你可以傳遞一個陣列，以替換語言字串中的置換符號作為 ``lang()`` 函數的第二個參數。這可以實現非常簡單的數字翻譯和格式化。
::

    // The language file, Tests.php:
    return [
        "apples"      => "I have {0, number} apples.",
        "men"         => "The top {1, number} men out-performed the remaining {0, number}",
        "namedApples" => "I have {number_apples, number, integer} apples.",
    ];

    // Displays "I have 3 apples."
    echo lang('Tests.apples', [ 3 ]);

置換符號中的第一個項目對應於陣列中項目的索引（如果它是數字的話）。
::

    // Displays "The top 23 men out-performed the remaining 20"
    echo lang('Tests.men', [20, 23]);

如果您願意，您還可以使用命名鍵來更簡單的理解。
::

    // Displays "I have 3 apples."
    echo lang("Tests.namedApples", ['number_apples' => 3]);

顯然，您可以做的不僅僅是數字替換。
根據底層庫的 `官方 ICU 文件 <https://unicode-org.github.io/icu-docs/apidoc/released/icu4c/classMessageFormat.html#details>`_ ，
可以替換以下類型的數據：

* 數字 - 整數、貨幣、百分比
* 日期 - 短、中、長、滿
* 時間 - 短、中、長、滿
* 拼寫 - 寫出數字，例如：34 變為三十四
* 序數
* 期間

這裡有一些例子：
::

    // The language file, Tests.php
    return [
        'shortTime'  => 'The time is now {0, time, short}.',
        'mediumTime' => 'The time is now {0, time, medium}.',
        'longTime'   => 'The time is now {0, time, long}.',
        'fullTime'   => 'The time is now {0, time, full}.',
        'shortDate'  => 'The date is now {0, date, short}.',
        'mediumDate' => 'The date is now {0, date, medium}.',
        'longDate'   => 'The date is now {0, date, long}.',
        'fullDate'   => 'The date is now {0, date, full}.',
        'spelledOut' => '34 is {0, spellout}',
        'ordinal'    => 'The ordinal is {0, ordinal}',
        'duration'   => 'It has been {0, duration}',
    ];

    // Displays "The time is now 11:18 PM"
    echo lang('Tests.shortTime', [time()]);
    // Displays "The time is now 11:18:50 PM"
    echo lang('Tests.mediumTime', [time()]);
    // Displays "The time is now 11:19:09 PM CDT"
    echo lang('Tests.longTime', [time()]);
    // Displays "The time is now 11:19:26 PM Central Daylight Time"
    echo lang('Tests.fullTime', [time()]);

    // Displays "The date is now 8/14/16"
    echo lang('Tests.shortDate', [time()]);
    // Displays "The date is now Aug 14, 2016"
    echo lang('Tests.mediumDate', [time()]);
    // Displays "The date is now August 14, 2016"
    echo lang('Tests.longDate', [time()]);
    // Displays "The date is now Sunday, August 14, 2016"
    echo lang('Tests.fullDate', [time()]);

    // Displays "34 is thirty-four"
    echo lang('Tests.spelledOut', [34]);

    // Displays "It has been 408,676:24:35"
    echo lang('Tests.ordinal', [time()]);

您應該確保閱讀 MessageFormatter 類別和基礎 ICU 格式，以便更好地瞭解它具有哪些功能，例如執行條件替換、複數等。
前面提供的兩個鏈接都能讓你對可用的選項有一個很好的了解。

指定語言環境
-----------------

要指定在替換參數時使用不同的語言環境，你可以把語言環境作為第三個參數傳給 ``lang()`` 方法。
::

    // Displays "The time is now 23:21:28 GMT-5"
    echo lang('Test.longTime', [time()], 'ru-RU');

    // Displays "£7.41"
    echo lang('{price, number, currency}', ['price' => 7.41], 'en-GB');
    // Displays "$7.41"
    echo lang('{price, number, currency}', ['price' => 7.41], 'en-US');

巢狀陣列
-------------

語言檔案還允許巢狀陣列更容易使用列表等...。
::

    // Language/en/Fruit.php

    return [
        'list' => [
            'Apples',
            'Bananas',
            'Grapes',
            'Lemons',
            'Oranges',
            'Strawberries',
        ],
    ];

    // Displays "Apples, Bananas, Grapes, Lemons, Oranges, Strawberries"
    echo implode(', ', lang('Fruit.list'));

語言回退
=================

如果你有一組給定的語言環境訊息，例如 ``Language/en/app.php`` ，
你可以為該語言環境添加語言變體，每個變體都在自己的文件夾中，例如 ``Language/en-US/app.php`` 。

你只需要為那些語言環境變體進行不同的本地化的訊息提供值。任何缺失的訊息定義將自動從主要的語言環境中提取。

更棒的是 ── 本地化可以一直回退到英語，以防你還沒有機會為你的語言環境翻譯它們就讓新的訊息被添加到框架中。

因此，如果你使用的是 ``fr-CA`` 語言環境，那麼本地化的訊息將首先在 ``Language/fr/CA`` 中尋找，
然後是 ``Language/fr`` ，最後是 ``Language/en`` 。

訊息翻譯
====================

我們有一套 "官方" 的翻譯，在他們的 `儲存庫 <https://github.com/codeigniter4/translations>`_ 中。

您可以下載該儲存庫，並將其 ``Language`` 資料夾複製到您的 ``app`` 中。
由於 ``app`` 命名空間已映射到你的 ``app`` 資料夾，合併的翻譯將被自動接收。

另外，更好的做法是在你的項目中使用 ``composer require codeigniter4/translations``，
由於翻譯文件夾得到了適當的映射，翻譯後的信息會被自動接收。
