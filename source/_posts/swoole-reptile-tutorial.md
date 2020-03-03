---
title: Swoole 简单爬虫
date: 2019-07-08 15:49:33
tags: ["swoole"]
category:
  - php
  - swoole
---

### 前言
闲着学习下，利用swoole下载github对应项目的全部包

### 目录
```
.
├── composer.json
├── packages
├── src
│   ├── Dom.php
│   └── Scraper.php
└── start.php
```

### 流程
1. 注入http服务类 Saber `public function __construct(Saber $saber, string $savePath)`
2. 配置要下载的包r `public function scrape(array $options)`
3. 开始爬取任务 `public function run()`
4. 获取项目分页信息（一直循环到没有包为止） `protected function fetchPagination(string $package, string $lastVersion = '')`
5. 下载包 `protected function download(string $version, string $url, string $ext)`

###  使用包
- [symfony/dom-crawler](https://packagist.org/packages/symfony/dom-crawler)
- [symfony/css-selector](https://packagist.org/packages/symfony/css-selector)
- [swlib/saber](https://packagist.org/packages/swlib/saber)

### 创建项目

```
# composer.json
{
    "name": "yourname/github-package-downloader",
    "authors": [
        {
            "name": "yourname",
            "email": "yourname@live.com"
        }
    ],
    "require": {
        "symfony/dom-crawler": "^4.2",
        "symfony/css-selector": "^4.3",
        "swlib/saber": "^1.0"
    },
    "autoload": {
        "psr-4": {
            "App\\": "./src"
        }
    }
}

```

```bash
composer install
```

### 编写
- Scraper.php
- Dom.php
- start.php

```php
# src/Scraper.php

namespace App;

use App\Dom;
use Swlib\Saber;

final class Scraper
{
    private $saber;

    private $package;

    private $savePath;

    const HOST = 'https://github.com';

    const DOWNLOAD = 'https://codeload.github.com';

    public function __construct(Saber $saber, string $savePath)
    {
        $this->saber = $saber;
        $this->savePath = $savePath;
    }

    /**
     * 配置
     *
     * @param array $packages
     * @return void
     */
    public function scrape(array $packages)
    {
        $this->packages = $packages;

        return $this;
    }

    /**
     * 开始运行
     *
     * @return void
     */
    public function run()
    {
        foreach ($this->packages as $package) {
            go(function () use ($package) {
                $this->fetchPagination($package);
            });
        }
    }

    /**
     * 拉取信息
     *
     * @param string $package
     * @param string $lastVersion
     * @return void
     */
    protected function fetchPagination(string $package, string $lastVersion = '')
    {
        $url = $package . '/tags?after=' . $lastVersion;
        try {
            echo "=============================================\n";
            echo "开始拉取:{$url}\n";
            echo "=============================================\n";
            $html = $this->saber->get(Scraper::HOST . '/' . $url, ['max_co' => 5, 'timeout' => 500]);
            

            $dom = new Dom((string)$html);

            if (true === $dom->hasPackage()) {
                $lastVersion = $dom->getLastVersion();
                $data = $dom->getVersionUrls();
                
                foreach($data as $item) {
                    foreach($item['urls'] as $ext => $url) {
                        $this->download($item['version'], $url, $ext);
                    }
                }
                
                $this->fetchPagination($package, $lastVersion);
                
            } else {
                echo "拉取完成\n";
            }
        } catch (\Exception $e) {
            echo $e->getMessage();
        }
    }

    /**
     * 下载、保存包
     *
     * @return void
     */
    protected function download(string $version, string $url, string $ext)
    {
        go(function () use ($version, $url, $ext) {
            list($tmp, $user, $package, $archive, $file) = explode('/', $url);
            $dir = $this->savePath . "/{$user}/{$package}";

            if (!file_exists($dir)) {
                mkdir($dir, 0777, true);
            }

            $savePath = $dir . '/' . $file;
            try {
                $response = $this->saber->download(
                    Scraper::DOWNLOAD . '/' . $user.'/'. $package .'/'. $ext .'/' . $version,
                    $savePath
                );

                if ($response->success) {
                    echo "下载完成\n";
                }
            } catch (\Exception $e) {
                $this->download($version, $url, $ext);
            }
        });
    }
}

```


```php

# src/Dom.php

namespace App;

use Symfony\Component\DomCrawler\Crawler;

class Dom
{
    protected $html;

    public function __construct(string $html)
    {
        $this->crawler = new Crawler((string)$html);
    }

    /**
     * 是否存在包
     *
     * @return boolean
     */
    public function hasPackage()
    {
        return count($this->crawler->filter('.blankslate')) ? false : true; 
    }

    /**
     * 是否分页
     *
     * @return boolean
     */
    public function hasPagination()
    {
        try {
            $this->crawler->filter('.pagination a')->text();
            return true;
        } catch(\Exception $e) {
            return false;
        }
    }

    /**
     * 获取版本号与下载链接
     *
     * @return void
     */
    public function getVersionUrls()
    {
        $data = [];
        $this->crawler->filter('.Box-row')->each(function (Crawler $node, $i) use(&$data) {
            $data[] = [
                'version' => $this->getVersion($node),
                'urls' => $this->getDownloadLink($node),
            ];
        });

        return $data;
    }

    /**
     * 获取版本号
     *
     * @param Crawler $node
     * @return void
     */
    protected function getVersion(Crawler $node)
    {
        return trim($node->filter('.commit .d-flex h4 a')->text());
    }

    /**
     * 获取下载链接
     *
     * @param Crawler $node
     * @return void
     */
    protected function getDownloadLink(Crawler $node)
    {
        $urls = [
            'zip' => $node->filter('ul.list-style-none li:nth-child(3) a')->attr('href'),
            'tar.gz' => $node->filter('ul.list-style-none li:nth-child(4) a')->attr('href')
        ];

        return $urls;
    }

    /**
     * 获取最后一个版本
     *
     * @return void
     */
    public function getLastVersion()
    {
        $version = $this->crawler->filter('.Box-row:last-child .d-flex a')->text();

        return trim($version);
    }

}
```

```php

# start.php

use App\Scraper;
use Swlib\Saber;

require_once './vendor/autoload.php';

$saber = Saber::create([
    'User-Agent' => 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/77.0.3824.6 Safari/537.36',
    'use_pool' => true
]);

// 注入http库，配置保存路径
$scraper = new Scraper($saber, __DIR__ . '/packages');

// 配置需要爬取的项目并运行
$scraper->scrape([
    'swlib/saber',
])->run();
```

### 运行
```
php start.php
```