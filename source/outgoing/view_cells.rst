##########
視圖單元
##########

視圖單元允許您插入在控制器之外產生的 HTML。它只呼叫特定的類別和方法，且必須回傳合法的 HTML 字串。
這個方法可以是任何自動載入器可定位之類別的可呼叫方法。唯一的限制是該類別的建構不能有任何的建構參數。
此用意旨在幫助模組化您的程式碼。
::

    <?= view_cell('\App\Libraries\Blog::recentPosts') ?>

在這個例子中，類別 ``App\Libraries\Blog`` 被載入，且方法 ``recentPosts()`` 被執行。此方法必須回傳合法的 HTML 字串。
此方法可以是靜態方法或者是非靜態方法。

單元參數
---------------

您可以透過傳入一串參數作為此方法的第二個參數來改善呼叫方式。
傳入的值可以是鍵值陣列或者是以逗號分隔鍵值字串::

    // Passing Parameter Array
    <?= view_cell('\App\Libraries\Blog::recentPosts', ['category' => 'codeigniter', 'limit' => 5]) ?>

    // Passing Parameter String
    <?= view_cell('\App\Libraries\Blog::recentPosts', 'category=codeigniter, limit=5') ?>

    public function recentPosts(array $params = [])
    {
        $posts = $this->blogModel->where('category', $params['category'])
                                 ->orderBy('published_on', 'desc')
                                 ->limit($params['limit'])
                                 ->get();

        return view('recentPosts', ['posts' => $posts]);
    }

此外，您可以在此方法中使用與參數變數相對應的參數名稱來提高程式碼可讀性。
當您採用此方法時，在這個視圖單元呼叫的所有參數皆必須被指定::

    <?= view_cell('\App\Libraries\Blog::recentPosts', 'category=codeigniter, limit=5') ?>

    public function recentPosts(string $category, int $limit)
    {
        $posts = $this->blogModel->where('category', $category)
                                 ->orderBy('published_on', 'desc')
                                 ->limit($limit)
                                 ->get();

        return view('recentPosts', ['posts' => $posts]);
    }

單元快取
------------

您可以透過將要快取資料的秒數作為第三個參數傳入來快取視圖單元呼叫結果。
這將使用當前配置的快取引擎。
::

    // Cache the view for 5 minutes
    <?= view_cell('\App\Libraries\Blog::recentPosts', 'limit=5', 300) ?>

如果您願意，可以提供一個自訂名稱作為第四個參數來命名快取檔案名稱::

    // Cache the view for 5 minutes
    <?= view_cell('\App\Libraries\Blog::recentPosts', 'limit=5', 300, 'newcacheid') ?>
