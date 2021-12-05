################
HTML 表格類別
################

表格類別提供了讓你能夠由陣列或是資料庫結果集合自動生成 HTML 表格的方法。

.. contents::
  :local:

*********************
使用表格類別
*********************

初始化類別
======================

表格類別沒有作為一個服務被提供，需要 "正常地" 被初始化，舉例來說::

    $table = new \CodeIgniter\View\Table();

範例
========

這裡展示了一個如何從一個多維陣列建立一個表格的範例。要注意，陣列的第一個索引值會成為表頭 ( 或是你可以透過 ``setHeading()`` 方法設定你自己的表頭，這在下方的方法參考有描述 )。

::

    $table = new \CodeIgniter\View\Table();

    $data = [
        ['Name', 'Color', 'Size'],
        ['Fred', 'Blue',  'Small'],
        ['Mary', 'Red',   'Large'],
        ['John', 'Green', 'Medium'],
    ];

    echo $table->generate($data);

這裡的是從資料庫查詢結果建立表格的範例。表格類別將會自動根據表格的名稱產生表頭 ( 或是你可以透過 ``setHeading()`` 方法設定你自己的表頭，這在下方的類別參考有描述 )。

::

    $table = new \CodeIgniter\View\Table();

    $query = $db->query('SELECT * FROM my_table');

    echo $table->generate($query);

這裡展示的是你如何使用分散的參數建立表格的範例::

    $table = new \CodeIgniter\View\Table();

    $table->setHeading('Name', 'Color', 'Size');

    $table->addRow('Fred', 'Blue', 'Small');
    $table->addRow('Mary', 'Red', 'Large');
    $table->addRow('John', 'Green', 'Medium');

    echo $table->generate();

這是相同的範例，不過不是用個別的參數，而是用陣列::

    $table = new \CodeIgniter\View\Table();

    $table->setHeading(array('Name', 'Color', 'Size'));

    $table->addRow(['Fred', 'Blue', 'Small']);
    $table->addRow(['Mary', 'Red', 'Large']);
    $table->addRow(['John', 'Green', 'Medium']);

    echo $table->generate();

改變表格的外觀
===============================

表格類別允許你設定一個表格的樣板，可以用它來指定你的佈局。這裡的是樣板的雛形::

    $template = [
        'table_open'         => '<table border="0" cellpadding="4" cellspacing="0">',

        'thead_open'         => '<thead>',
        'thead_close'        => '</thead>',

        'heading_row_start'  => '<tr>',
        'heading_row_end'    => '</tr>',
        'heading_cell_start' => '<th>',
        'heading_cell_end'   => '</th>',

        'tfoot_open'         => '<tfoot>',
        'tfoot_close'        => '</tfoot>',

        'footing_row_start'  => '<tr>',
        'footing_row_end'    => '</tr>',
        'footing_cell_start' => '<td>',
        'footing_cell_end'   => '</td>',

        'tbody_open'         => '<tbody>',
        'tbody_close'        => '</tbody>',

        'row_start'          => '<tr>',
        'row_end'            => '</tr>',
        'cell_start'         => '<td>',
        'cell_end'           => '</td>',

        'row_alt_start'      => '<tr>',
        'row_alt_end'        => '</tr>',
        'cell_alt_start'     => '<td>',
        'cell_alt_end'       => '</td>',

        'table_close'        => '</table>'
    ];

    $table->setTemplate($template);

.. note:: 你會注意到樣板裡有兩組 「row」 的區塊。這允許你建立交替行的顏色或是設計會在每次列資料疊代時變換的元素。

你不需要提交一個完整的樣板。如果你只需要改變佈局的一部分，你可以只提交那些元素。在這個範例中，只有改變了表格的起始標籤::

    $template = [
        'table_open' => '<table border="1" cellpadding="2" cellspacing="1" class="mytable">'
    ];

    $table->setTemplate($template);

你可以傳遞樣板設定的陣列給表格的建構子以設定預設::

    $customSettings = [
        'table_open' => '<table border="1" cellpadding="2" cellspacing="1" class="mytable">'
    ];

    $table = new \CodeIgniter\View\Table($customSettings);


***************
類別參考
***************

.. php:class:: Table

    .. attribute:: $function = null

        允許你指定要被應用在所有單元資料的原生 PHP 方法或是合法的方法陣列物件
        ::

            $table = new \CodeIgniter\View\Table();

            $table->setHeading('Name', 'Color', 'Size');
            $table->addRow('Fred', '<strong>Blue</strong>', 'Small');

            $table->function = 'htmlspecialchars';
            echo $table->generate();

        在上方的範例中，所有的單元資料將會通過 PHP 的 :php:func:`htmlspecialchars()` 方法，最後產生::

            <td>Fred</td><td>&lt;strong&gt;Blue&lt;/strong&gt;</td><td>Small</td>

    .. php:method:: generate([$tableData = null])

        :param    混合型    $tableData: 用於填充表格列的資料
        :returns:    HTML 表格
        :rtype:    字串

        回傳一個包含了被產生的表格的字串。接受一個可選擇一個陣列或是資料庫結果物件的參數。

    .. php:method:: setCaption($caption)

        :param    字串    $caption: 表格說明
        :returns:    表格實體 ( 方法鏈 )
        :rtype:    表格

        允許你增加表格的說明::

            $table->setCaption('Colors');

    .. php:method:: setHeading([$args = [] [, ...]])

        :param    混合型    $args: 一個陣列或是包含了表格欄位標題的多個字串
        :returns:    表格實體 ( 方法鏈 )
        :rtype:    表格

        允許你設定表格的表頭。你可以提交一個陣列或是分散的參數::

            $table->setHeading('Name', 'Color', 'Size'); // 或是

            $table->setHeading(['Name', 'Color', 'Size']);

    .. php:method:: setFooting([$args = [] [, ...]])

        :param    混合型    $args: 一個陣列或是包含了表尾值的多個字串
        :returns:    表格實體 ( 方法鏈 )
        :rtype:    表格

        允許你設定表尾。你可以提交一個陣列或是分散的參數::

            $table->setFooting('Subtotal', $subtotal, $notes); // 或是

            $table->setFooting(['Subtotal', $subtotal, $notes]);

    .. php:method:: addRow([$args = [] [, ...]])

        :param    混合型    $args: 一個陣列或是包含了列值的多個字串
        :returns:    表格實體 ( 方法鏈 )
        :rtype:    表格

        允許你增加一列到你的表格中。你可以提交一個陣列或是分散的參數::

            $table->addRow('Blue', 'Red', 'Green'); // 或是

            $table->addRow(['Blue', 'Red', 'Green']);


        如果你想要設定一個獨立單元的標籤屬性，你可以為那個單元使用一個關聯陣列。關聯鍵 **data** 定義了單元的資料。任何其他的鍵 => 值對被以鍵 = \\'值\\' 屬性加到標籤裡::

            $cell = ['data' => 'Blue', 'class' => 'highlight', 'colspan' => 2];
            $table->addRow($cell, 'Red', 'Green');

            // 產生
            // <td class='highlight' colspan='2'>Blue</td><td>Red</td><td>Green</td>

    .. php:method:: makeColumns([$array = [] [, $columnLimit = 0]])

        :param    陣列    $array: 一個包含了多列資料的陣列
        :param    整數    $columnLimit: 表格裡的欄位數
        :returns:    一個 HTML 表格欄位的陣列
        :rtype:    陣列

        這個方法要求輸入一個一維陣列與期望的欄位數並建立有著跟其相同深度的多維陣列。這允許一個有著許多元素的陣列被顯示在一個固定行數的表格中。請看這個例子::

            $list = ['one', 'two', 'three', 'four', 'five', 'six', 'seven', 'eight', 'nine', 'ten', 'eleven', 'twelve'];

            $newList = $table->makeColumns($list, 3);

            $table->generate($newList);

            // 以這個做為雛形產生表格

            <table border="0" cellpadding="4" cellspacing="0">
            <tr>
            <td>one</td><td>two</td><td>three</td>
            </tr><tr>
            <td>four</td><td>five</td><td>six</td>
            </tr><tr>
            <td>seven</td><td>eight</td><td>nine</td>
            </tr><tr>
            <td>ten</td><td>eleven</td><td>twelve</td></tr>
            </table>


    .. php:method:: setTemplate($template)

        :param    陣列    $template: 一個包含樣板值的關聯陣列
        :returns:    若成功回傳 true ， 失敗則回傳 false
        :rtype:    布林值

        允許你設定你的樣板。你可以提交一個完整或部分的樣板。
        ::

            $template = [
                'table_open' => '<table border="1" cellpadding="2" cellspacing="1" class="mytable">'
            ];

            $table->setTemplate($template);

    .. php:method:: setEmpty($value)

        :param    混合型    $value: 要放進空白單元的值
        :returns:    表格實體 ( 方法鏈 )
        :rtype:    表格

        讓你為任何空白的表格單元設定預設。舉例來說，你可能會設定一個不換行空格::

            $table->setEmpty("&nbsp;");

    .. php:method:: clear()

        :returns:    表格實體 ( 方法鏈 )
        :rtype:    表格

        讓你清除表頭、列的資料與說明。如果你需要展示多個不同資料的表格，你應該要在每個表格被產生後呼叫這個方法以清除先前表格的訊息。

        範例 ::

            $table = new \CodeIgniter\View\Table();

            $table->setCaption('Preferences')
                ->setHeading('Name', 'Color', 'Size')
                ->addRow('Fred', 'Blue', 'Small')
                ->addRow('Mary', 'Red', 'Large')
                ->addRow('John', 'Green', 'Medium');

            echo $table->generate();

            $table->clear();

            $table->setCaption('Shipping')
                ->setHeading('Name', 'Day', 'Delivery')
                ->addRow('Fred', 'Wednesday', 'Express')
                ->addRow('Mary', 'Monday', 'Air')
                ->addRow('John', 'Saturday', 'Overnight');

            echo $table->generate();
