==============
HTTP 響應
==============

Response 類別透過僅適用於當伺服器回應呼叫它的客戶端的方法來擴展 :doc:`HTTP 訊息類別 </incoming/message>` 

.. contents::
    :local:
    :depth: 2

使用 Response
=========================

一個 Response 類別已經為你實例化並且傳入你的控制器。它可以透過 
``$this->response`` 來訪問。由於 CodeIgniter 會替你傳送頁眉與主體，所以大多數的時候你並不需要直接接觸到類別。如果頁面成功的創建出它被要求創建的內容會是一件很棒的事。
當出現了問題，或者你需要送回非常特定的狀態碼，又或者利用強大的 HTTP 快取，它都會隨時就緒。

設定輸出
------------------

當你需要直接設定腳本的輸出，並且不依靠 CodeIgniter 來自動取得它，你必須手動的透過 
``setBody`` 方法來執行它。這經常與設定響應的狀態碼結合使用。::

    $this->response->setStatusCode(404)->setBody($body);

原因短語 ('OK', 'Created', 'Moved Permanently') 會被自動加入，但你也可以添加自定義的原因作為 ``setStatusCode()`` 方法的第二個參數::

    $this->response->setStatusCode(404, 'Nope. Not here.');

你可以將陣列設定格式為 JSON 或 XML 並以 ``setJSON`` and ``setXML`` 方法設定內容型別頁眉成合適的 mime。通常，你會傳入一個陣列的待轉換資料::

    $data = [
        'success' => true,
        'id' => 123,
    ];

    return $this->response->setJSON($data);
    // or
    return $this->response->setXML($data);

設定頁眉
---------------

你經常會需要設置要為響應設置的頁眉。Response 類別透過 ``setHeader()`` 方法使這件事變得容易操作。
第一個參數是頁眉的名字。第二個參數是值，它可以是當傳給客戶端時會被正確結合的字串或是一個陣列的值。
使用這些函數而不是使用原生的 PHP 函數可以使你確保沒有頁眉會被過早的送出，產生錯誤，進而使測試成為可能。
::

    $response->setHeader('Location', 'http://example.com')
             ->setHeader('WWW-Authenticate', 'Negotiate');

要是頁眉存在並且可以有多於一個值，你可以分別使用 ``appendHeader()`` 與 ``prependHeader()`` 
方法在值列表的結尾或開始處增加值。第一個參數是頁眉的名字，第二個參數則是要附加或前置的值。
::

    $response->setHeader('Cache-Control', 'no-cache')
             ->appendHeader('Cache-Control', 'must-revalidate');

頁眉可以透過 ``removeHeader()`` 方法從響應中刪除，它只需頁眉名稱作為唯一的參數。這不區分大小寫。
::

    $response->removeHeader('Location');

強制檔案下載
===================

Response 類別提供了一個將檔案送到客戶端的簡單方法，提示瀏覽器將檔案下載到你的電腦。這會設置設置合適的頁眉使其發生。

第一個參數是 **你想要已下載的檔案被命名的名字**，第二個參數是檔案資料。

如果你將第二個參數設為 null 但 ``$filename`` 是一個存在且可讀的檔案路徑，那它的內容還是會被讀取。

如果你將第三個參數設為布林值 true，那檔案實際的 MIME 型別 
(基於檔名擴充) 會被送出，所以如果你的瀏覽器有對應那個型別的處理程序-它便可以使用它。

範例::

    $data = 'Here is some text!';
    $name = 'mytext.txt';
    return $response->download($name, $data);

如果你想要從你的伺服器下載一個存在的檔案，你會需要在第二個參數明確的傳入 ``null`` ::

    // Contents of photo.jpg will be automatically read
    return $response->download('/path/to/photo.jpg', null);

使用可選的 ``setFileName()`` 方法來更改已經被送到客戶端的瀏覽器檔案的檔名::

    return $response->download('awkwardEncryptedFileName.fakeExt', null)->setFileName('expenses.csv');

.. note:: 響應物件必須必須回傳才能將下載發送給客戶端。這允許響應在傳給客戶端之前傳過所有 **after** 過濾器。

HTTP 快取
============

內置於 HTTP 規範中的工具可以幫助客戶端(通常是網頁瀏覽器)快取結果。正確使用的話，可以使你的應用程式效能大大提升，因為它會通知客戶端他們不須聯繫
 getServer 因為沒有東西被更改。而且你不能比這個再更快了。

這是透過 ``Cache-Control`` 與 ``ETag`` 頁眉處理。這個手冊不是一個完整解析所有快取頁眉能力的地方，但你可以透過
`Google Developers <https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching>`_
了解更多。

在預設上，所有通過 CodeIgniter 發送的響應物件都會將 HTTP 快取關閉。它的選項與確切的情況太多變了，對我們來說與其創建一個好的默認設置不如將它關閉。
透過 ``setCache()`` 方法可以輕易地將快取值設為你需要的值::

    $options = [
        'max-age'  => 300,
        's-maxage' => 900,
        'etag'     => 'abcde'
    ];
    $this->response->setCache($options);

``$options`` 陣列只接受一個鍵/值對陣列，除了少數例外，這些鍵/值會被分配到 ``Cache-Control`` 頁眉。你可以自由地將所有的選項依照你特定的情況設置成你需要的樣子。
當大多數的選項都被應用到 ``Cache-Control`` 頁眉，它會聰明的處理
 ``etag`` 與 ``last-modified`` 選項到他們合適的頁眉。

內容安全策略
=======================

其中一個對抗 XSS 攻擊最好的保護措施便是在網站上實施內容安全策略(CSP)。
這會強制你將每一個從你網站的 HTML 拉入的內容來源放入白名單，包含圖片、樣式表、javascript 檔案等等。
瀏覽器會拒絕來自非白名單的來源發出的內容。此白名單是由響應的 ``Content-Security-Policy`` 頁眉中創建的，並且它可以通過多種不同的方式配置。

這聽起來很複雜，而且在某些網站，這確實是個挑戰。但是，對於很多簡單的網站，所有內容都由同一個網域提供服務
(http://example.com)，要整合是非常輕鬆的。

由於這是一個複雜的主題，本用戶指南將不會詳細介紹所有細節。如果想知道更多資訊，請造訪下列網站:

* `Content Security Policy main site <https://content-security-policy.com/>`_
* `W3C Specification <https://www.w3.org/TR/CSP>`_
* `Introduction at HTML5Rocks <https://www.html5rocks.com/en/tutorials/security/content-security-policy/>`_
* `Article at SitePoint <https://www.sitepoint.com/improving-web-security-with-the-content-security-policy/>`_

開啟 CSP
--------------

在預設上，此項支援是關閉的。如果要在你的應用程式啟用此支援，請於
**app/Config/App.php** 內編輯 ``CSPEnabled`` 值::

    public $CSPEnabled = true;

啟用之後，響應物件會包含一個 ``CodeIgniter\HTTP\ContentSecurityPolicy`` 實例。在
 **app/Config/ContentSecurityPolicy.php** 內設定的值會被應用到那個實例中，並且如果在運行時沒有需要更改的地方，那正確格式化的頁眉會被送出，你就成功了。

當 CSP 已被啟用，兩個頁眉行會被加入 HTTP 響應:一個 Content-Security-Policy 頁眉，有著識別明確的被允許用於不同語境的內容型別或起源的策略，還有一個 Content-Security-Policy-Report-Only 頁眉，會識別被允許的內容型別或起源，但它同時也會被回報到你所選擇的目的地。


我們的實作會提供一個默認處理，可以透過 ``reportOnly()`` 方法更改。
當一個額外的條目被添加到一個 CSP 指示，如下所示，它會被添加到適合阻擋或防止的 CSP 頁眉。它可以在每次呼叫的基礎上覆寫，透過提供一個自選的第二參數到添加方法的呼叫。

運行組態設定
---------------------

如果你的應用程式需要在運行時做更改，你可以存取在 ``$response->CSP`` 的實體。這類別包含許多方法可以清楚的導引到你需要設置的合適的頁眉值。
下方提供了一些範例，儘管有不同的參數組合，但不論是指示名稱或他們自己的陣列都是全部被接受的。
::

    // specify the default directive treatment
    $response->CSP->reportOnly(false);

    // specify the origin to use if none provided for a directive
    $response->CSP->setDefaultSrc('cdn.example.com');

    // specify the URL that "report-only" reports get sent to
    $response->CSP->setReportURI('http://example.com/csp/reports');

    // specify that HTTP requests be upgraded to HTTPS
    $response->CSP->upgradeInsecureRequests(true);

    // add types or origins to CSP directives
    // assuming that the default treatment is to block rather than just report
    $response->CSP->addBaseURI('example.com', true); // report only
    $response->CSP->addChildSrc('https://youtube.com'); // blocked
    $response->CSP->addConnectSrc('https://*.facebook.com', false); // blocked
    $response->CSP->addFontSrc('fonts.example.com');
    $response->CSP->addFormAction('self');
    $response->CSP->addFrameAncestor('none', true); // report this one
    $response->CSP->addImageSrc('cdn.example.com');
    $response->CSP->addMediaSrc('cdn.example.com');
    $response->CSP->addManifestSrc('cdn.example.com');
    $response->CSP->addObjectSrc('cdn.example.com', false); // reject from here
    $response->CSP->addPluginType('application/pdf', false); // reject this media type
    $response->CSP->addScriptSrc('scripts.example.com', true); // allow but report requests from here
    $response->CSP->addStyleSrc('css.example.com');
    $response->CSP->addSandbox(['allow-forms', 'allow-scripts']);

對於每一個 "add" 方法，第一個參數都是一個適合的字串值或一個他們自己的陣列。

``reportOnly`` 方法允許你指定對於後續來源的預設回報處理，除非它被覆寫。例如，你可以指定 youtube.com 是被允許的，
然後提供數個被允許但被回報的來源::

    $response->addChildSrc('https://youtube.com'); // allowed
    $response->reportOnly(true);
    $response->addChildSrc('https://metube.com'); // allowed but reported
    $response->addChildSrc('https://ourtube.com',false); // allowed

行內內容
--------------

將一個網頁設定成連自己頁面的行內腳本與樣式都不保護是可行的，因為這有可能是用戶生成內容的結果。為了防止這樣，CSP 允許你在
``<style>`` 與 ``<script>`` 標籤中指定一個隨機數，並將這些值加到響應的頁眉。在現實生活中這很難處理，並且在動態生成時是最安全的。為了簡化這個，你可以在標籤中包含一個 ``{csp-style-nonce}`` 或
``{csp-script-nonce}`` 置換符號，然後它將會自動幫你處理好
::

    // Original
    <script {csp-script-nonce}>
        console.log("Script won't run as it doesn't contain a nonce attribute");
    </script>

    // Becomes
    <script nonce="Eskdikejidojdk978Ad8jf">
        console.log("Script won't run as it doesn't contain a nonce attribute");
    </script>

    // OR
    <style {csp-style-nonce}>
        . . .
    </style>

類別參考
===============

.. note:: 除了被列出的方法之外，此類別也繼承 :doc:`Message Class </incoming/message>` 的方法。

父類別提供可用方法有：

* :meth:`CodeIgniter\\HTTP\\Message::body`
* :meth:`CodeIgniter\\HTTP\\Message::setBody`
* :meth:`CodeIgniter\\HTTP\\Message::populateHeaders`
* :meth:`CodeIgniter\\HTTP\\Message::headers`
* :meth:`CodeIgniter\\HTTP\\Message::header`
* :meth:`CodeIgniter\\HTTP\\Message::headerLine`
* :meth:`CodeIgniter\\HTTP\\Message::setHeader`
* :meth:`CodeIgniter\\HTTP\\Message::removeHeader`
* :meth:`CodeIgniter\\HTTP\\Message::appendHeader`
* :meth:`CodeIgniter\\HTTP\\Message::protocolVersion`
* :meth:`CodeIgniter\\HTTP\\Message::setProtocolVersion`
* :meth:`CodeIgniter\\HTTP\\Message::negotiateMedia`
* :meth:`CodeIgniter\\HTTP\\Message::negotiateCharset`
* :meth:`CodeIgniter\\HTTP\\Message::negotiateEncoding`
* :meth:`CodeIgniter\\HTTP\\Message::negotiateLanguage`
* :meth:`CodeIgniter\\HTTP\\Message::negotiateLanguage`

.. php:class:: CodeIgniter\\HTTP\\Response

    .. php:method:: getStatusCode()

        :returns: 此響應現在的 HTTP 狀態碼
        :rtype: int

        回傳此響應目前的狀態碼。如果狀態碼沒有被設定，則會拋出 BadMethodCallException::

            echo $response->getStatusCode();

    .. php:method:: setStatusCode($code[, $reason=''])

        :param int $code: HTTP 狀態碼
        :param string $reason: 發生的原因，此為可選擇性參數。
        :returns: 當前響應實體
        :rtype: ``CodeIgniter\HTTP\Response``

        設定該被此響應送出的 HTTP 狀態碼::

            $response->setStatusCode(404);
        
        原因片語將根據官方列表自動產生。如果您需要設定自定義的狀態碼，可以將原因片語作為第二個參數傳入::

            $response->setStatusCode(230, "Tardis initiated");

    .. php:method:: getReasonPhrase()

        :returns: 發生當前狀態碼的原因．
        :rtype: string
        
        回傳此響應當前狀態碼。如果狀態沒有被設定，則回傳一個空字串::

            echo $response->getReasonPhrase();

    .. php:method:: setDate($date)

        :param DateTime $date: 一個設定此響應的時間之 DateTime 實體。
        :returns: 當前響應實體。
        :rtype: ``CodeIgniter\HTTP\Response``

        設定此響應的時間。 ``$date`` 參數必須是 ``DateTime`` 的實體::

            $date = DateTime::createFromFormat('j-M-Y', '15-Feb-2016');
            $response->setDate($date);

    .. php:method:: setContentType($mime[, $charset='UTF-8'])

        :param string $mime: 響應的內容型別。
        :param string $charset: 此響應採用的字元編碼。
        :returns: 當前響應實體。
        :rtype: ``CodeIgniter\HTTP\Response``

        設定此響應的內容型別::

            $response->setContentType('text/plain');
            $response->setContentType('text/html');
            $response->setContentType('application/json');
        
        在預設情況下，此方法採用 ``UTF-8`` 作為字元編碼。如果您需要修改，則可以將字元編碼作為第二個參數傳入。::

            $response->setContentType('text/plain', 'x-pig-latin');

    .. php:method:: noCache()

        :returns: 當前響應實體。
        :rtype: ``CodeIgniter\HTTP\Response``
        
        設定 ``Cache-Control`` 標頭來關閉所有 HTTP 快取。這是所有響應訊息的預設設定::

            $response->noCache();

            // Sets the following header:
            Cache-Control: no-store, max-age=0, no-cache

    .. php:method:: setCache($options)

        :param array $options: 一組控制快取設定的鍵值陣列
        :returns: 當前響應實體。
        :rtype: ``CodeIgniter\HTTP\Response``
        
        設定 ``Cache-Control`` 標頭，包含 ``ETags`` 和 ``Last-Modified``。典型的鍵有：

        * etag
        * last-modified
        * max-age
        * s-maxage
        * private
        * public
        * must-revalidate
        * proxy-revalidate
        * no-transform
        
        當傳入 last-modified 選項時，傳入值可以是一個日期字串或者是 DateTime 物件。

    .. php:method:: setLastModified($date)

        :param string|DateTime $date: 設置 Last-Modified 標頭的時間
        :returns: 當前響應實體。
        :rtype: ``CodeIgniter\HTTP\Response``

        設定 ``Last-Modified`` 標頭。 ``$date`` 物件可以是一個字串或者是 ``DateTime`` 實體::

            $response->setLastModified(date('D, d M Y H:i:s'));
            $response->setLastModified(DateTime::createFromFormat('u', $time));

    .. php:method:: send(): Response

        :returns: 當前響應實體。
        :rtype: ``CodeIgniter\HTTP\Response``
        
        通知響應將所有東西回送到客戶端。首先會送出標頭，再來才是響應的本體內容。
        對於主要的應用程式響應，您不需要呼叫這個方法，因為會由 CodeIgniter 自動處理。
        
    .. php:method:: setCookie($name = ''[, $value = ''[, $expire = ''[, $domain = ''[, $path = '/'[, $prefix = ''[, $secure = false[, $httponly = false[, $samesite = null]]]]]]]])

        :param mixed $name: Cookie 名稱或者一組參數陣列
        :param string $value: Cookie 值
        :param int $expire: Cookie 過期時間，以秒為單位。
        :param string $domain: Cookie 作用域
        :param string $path: Cookie 路徑
        :param string $prefix: Cookie 前綴名稱
        :param bool $secure: 是否只通過 HTTPS 傳輸 cookie
        :param bool $httponly: 是否只允許 HTTP 請求存取 cookie，而 JavaScript 不行存取。
        :param string $samesite: 參數的值。如果該值被設為 ``''``，則不會在 cookie 上設定 SameSite 屬性。如果該值被設定為 `null`，則來自 `config/App.php` 的預設值會被使用。
        :rtype: void  SameSite cookie
        
        設定包含您指定的值之 cookie。有兩種方式可以將訊息傳送給該方法來設定 cookie： 陣列或者獨立參數：

        **陣列方法**
    
        使用此方法，需要將一個鍵值陣列作為第一個參數傳入::

            $cookie = [
                'name'   => 'The Cookie Name',
                'value'  => 'The Value',
                'expire' => '86500',
                'domain' => '.some-domain.com',
                'path'   => '/',
                'prefix' => 'myprefix_',
                'secure' => true,
                'httponly' => false,
                'samesite' => 'Lax'
            ];

            $response->setCookie($cookie);

        **注意事項**

        只需要名稱和值。如果需要刪除 cookie，則將設置為過期即可。
        過期時間採用 **秒數** 計時，將從當前時間開始計算。
        不要設定一個具體的時間，而是你希望從*現在*開始 cookie 的有效時間。
        如果過期時間被設為 0，則該cookie 只會在瀏覽器打開時有效，關閉時會被清除。

        對於全站的 cookie，無論您的網站是如何被請求的，請將您的網址加到 **domain** 中，並以句點開始，例如：
        
        通常不需要該路徑，因為已經預設設定根路徑。
        
        當您需要避免與伺服器其他相同名稱 cookie 發生衝突時，才需要前綴。

        當您需要加密 cookie 時，才需要將 secure 欄位設定為 ``true``。

        SameSite 值用來控制 cookie 如何在網域和子網域中被共用。
        合法的值有 'None', 'Lax', 'Strict', 或一個空白字串 ``''``。
        如果該值被設定為空白字串，則會使用預設的 SameSite 屬性。

        **獨立參數**

        如果您願意，您也可以將資料作為獨立參數傳入來設定 cookie::

            $response->setCookie($name, $value, $expire, $domain, $path, $prefix, $secure, $httponly, $samesite);

    .. php:method:: deleteCookie($name = ''[, $domain = ''[, $path = '/'[, $prefix = '']]])

        :param mixed $name: Cookie 名稱或一組參數陣列
        :param string $domain: Cookie 網域
        :param string $path: Cookie 路徑
        :param string $prefix: Cookie 前綴名稱
        :rtype: void

        透過將現存的 cookie 的過期時間設為 ``0``，即可將其刪除。

        **注意事項**

        只需要名稱。

        當您需要避免與伺服器其他相同名稱 cookie 發生衝突時，才需要前綴。

        如果只該刪除該子集的 cookie，請提供前綴。
        如果只該刪除該網域的 cookie，請提供網域名稱。
        如果只該刪除該路徑的 cookie，請提供路徑名稱。

        如果任何可選參數為空，則所有符合的同名 cookie 會被刪除。

        Example::

            $response->deleteCookie($name);

    .. php:method:: hasCookie($name = ''[, $value = null[, $prefix = '']])

        :param mixed $name: Cookie 名稱或一組參數陣列
        :param string $value: cookie 值
        :param string $prefix: Cookie 前綴名稱
        :rtype: bool

        檢查該響應是否含有特定的 cookie。

        **注意事項**

        只有名稱是必要的。如果有指定前綴，則它會被放在 cookie 名稱前面。
        
        如果 value 沒有被給定，則將只會檢查是否存在相對應名稱的 cookie。
        如果 value 被給定，則不只檢查是否存在相對應名稱的 cookie，還會檢查是否含有給定的 value。

        Example::

            if ($response->hasCookie($name)) ...

    .. php:method:: getCookie($name = ''[, $prefix = ''])

        :param string $name: Cookie 名稱
        :param string $prefix: Cookie 前綴名稱
        :rtype: ``Cookie|Cookie[]|null``
        
        如果有找到，則回傳相對應名稱的 cookie，否則回傳 ``null``。
        如果沒有給定名稱，則回傳一個含有 ``Cookie`` 物件的陣列。

        Example::

            $cookie = $response->getCookie($name);

    .. php:method:: getCookies()

        :rtype: ``Cookie[]``

        回傳該響應實體中的所有 cookie。
        這些只包含您專門指定在此請求中所設定的 cookie。